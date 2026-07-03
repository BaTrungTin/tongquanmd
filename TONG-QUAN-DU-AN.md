# 📚 TỔNG QUAN DỰ ÁN — Website Tour Du Lịch

> **Stack:** PHP (thuần, tự viết framework) · Pug (template engine) · MySQL · Node.js (render Pug) · Vanilla JS · CSS thuần

---

## 1. Mục đích dự án

Website quản lý và bán tour du lịch gồm hai phần chính:
- **Client (phía người dùng):** Duyệt tour, xem chi tiết, thêm vào giỏ hàng, đặt tour, thanh toán VNPay.
- **Admin (phía quản trị):** Đăng nhập, quản lý tour, danh mục, đơn hàng, tài khoản admin, cấu hình website, phân quyền.

---

## 2. Cấu trúc thư mục

```
tour/
├── public/               ← Web root (XAMPP trỏ vào đây)
│   ├── index.php         ← Entry point duy nhất (front controller)
│   ├── router.php        ← (dự phòng, không dùng chính)
│   ├── assets/
│   │   ├── css/style.css ← Toàn bộ CSS của website
│   │   └── js/script.js  ← Toàn bộ JS phía client
│   └── uploads/          ← Ảnh upload lên server
│
├── bootstrap.php         ← Khởi động ứng dụng (load env, config, DB, session)
│
├── routes/
│   └── web.php           ← Đăng ký toàn bộ route (URL → Controller)
│
├── app/
│   ├── Core/             ← Các lớp lõi tự viết
│   │   ├── Router.php    ← Điều hướng HTTP request
│   │   ├── Request.php   ← Đóng gói dữ liệu request
│   │   ├── Response.php  ← Helper redirect / JSON response
│   │   ├── Database.php  ← Khởi tạo PDO singleton
│   │   └── View.php      ← Render file .pug (qua Node.js) hoặc .php
│   │
│   ├── Controllers/
│   │   ├── Admin/        ← Controller dành cho trang admin
│   │   └── Client/       ← Controller dành cho trang client
│   │
│   ├── Models/           ← Các Model kế thừa BaseModel
│   │
│   ├── Middleware/       ← Middleware xác thực và khởi tạo dữ liệu dùng chung
│   │
│   └── Helpers/          ← Hàm hỗ trợ (Auth, Upload, Mail, String, …)
│
├── views/
│   ├── admin/            ← Template Pug cho trang admin
│   └── client/           ← Template Pug cho trang client
│
├── config/               ← Cấu hình app, database, biến toàn cục
├── database/             ← Schema SQL và seed data
├── scripts/              ← Script Node.js để render Pug
└── .env                  ← Biến môi trường (DB, mail, payment gateway)
```

---

## 3. Luồng hoạt động (Request Lifecycle)

```
Trình duyệt
    │
    ▼
public/index.php          (1) Front Controller duy nhất
    │  ├─ require bootstrap.php → load env, config, khởi tạo PDO, session
    │  └─ require routes/web.php → trả về $router
    │
    ▼
Router::dispatch()        (2) Khớp URL với danh sách route đã đăng ký
    │
    ▼
Middleware[]              (3) Chạy lần lượt
    │  ├─ ClientMiddleware::bootstrap()   → share categories, cities, settings vào View
    │  └─ AdminAuthMiddleware::verify()   → kiểm tra session, load role & permissions
    │
    ▼
Controller::action()      (4) Xử lý logic nghiệp vụ
    │  ├─ Gọi Model để query database
    │  └─ Gọi View::render('admin/pages/...', $data)
    │
    ▼
View::render()            (5) Tìm file .php hoặc .pug
    │  └─ Nếu là .pug: gọi Node.js → scripts/render-pug.mjs → trả HTML
    │
    ▼
HTML trả về trình duyệt
```

> **Lưu ý PATCH/DELETE:** Form HTML chỉ hỗ trợ GET/POST. Để dùng PATCH hoặc DELETE, form gửi POST kèm field `_method=PATCH` và `index.php` sẽ override method trước khi dispatch.

---

## 4. Chi tiết các lớp Core

### 4.1 `Router` — `app/Core/Router.php`
- Đăng ký route với `get()`, `post()`, `patch()`, `delete()`.
- URL hỗ trợ tham số động: `/tour/detail/:slug` → capture group `(?P<slug>[^/]+)`.
- `dispatch()` khớp method + regex path, chạy middleware, sau đó gọi `(new Controller)->action($request)`.

### 4.2 `Database` — `app/Core/Database.php`
- **Singleton** PDO, chỉ khởi tạo 1 lần duy nhất qua `Database::init($config)`.
- Cấu hình: `mysql:host=...; charset=utf8mb4`, error mode `EXCEPTION`, fetch mode `FETCH_ASSOC`.

### 4.3 `BaseModel` — `app/Models/BaseModel.php`
Lớp cha trừu tượng cho tất cả Model. Cung cấp các phương thức CRUD dùng PDO prepared statements:

| Phương thức | Mô tả |
|---|---|
| `findOne(array $filters)` | Lấy 1 bản ghi theo điều kiện |
| `find(array $filters, $options)` | Lấy nhiều bản ghi, hỗ trợ sort/limit/skip |
| `count(array $filters)` | Đếm số bản ghi |
| `insert(array $data)` | Thêm mới, trả về ID |
| `updateOne(filters, data)` | Cập nhật 1 bản ghi |
| `updateMany(filters, data)` | Cập nhật nhiều bản ghi |
| `deleteOne(array $filters)` | Xóa 1 bản ghi |
| `deleteMany(array $filters)` | Xóa nhiều bản ghi |
| `updateByIds(ids[], data)` | Cập nhật theo danh sách ID |
| `deleteByIds(ids[])` | Xóa theo danh sách ID |

Hỗ trợ bộ lọc nâng cao kiểu MongoDB-like:
```php
Tour::find(['price_adult' => ['$gte' => 1000000, '$lte' => 5000000]]);
Tour::find(['category' => ['$in' => [1, 2, 3]]]);
```
JSON fields (images, locations, schedules, …) được tự động encode/decode.

### 4.4 `View` — `app/Core/View.php`
- Ưu tiên render file `.php` trước, nếu không có thì render `.pug` (qua Node.js).
- `View::share($key, $value)` → inject biến toàn cục vào mọi template (dùng trong Middleware).
- `View::e($value)` → escape HTML an toàn.

---

## 5. Models (Bảng dữ liệu)

| Model | Bảng DB | Mô tả |
|---|---|---|
| `Tour` | `tours` | Tour du lịch (tên, giá, ảnh, lịch trình, …) |
| `Category` | `categories` | Danh mục tour (hỗ trợ cây danh mục cha-con) |
| `Order` | `orders` | Đơn đặt tour của khách hàng |
| `AccountAdmin` | `accounts_admin` | Tài khoản quản trị viên |
| `Role` | `roles` | Vai trò & phân quyền admin |
| `City` | `cities` | Danh sách tỉnh/thành phố |
| `Contact` | `contacts` | Email liên hệ của khách |
| `SettingWebsiteInfo` | `setting_website_info` | Cấu hình thông tin website |
| `ForgotPassword` | `forgot_password` | Token OTP đặt lại mật khẩu |

### Cấu trúc bảng `tours` (quan trọng nhất)
```sql
id, name, category, position, status, avatar,
images (JSON),         -- mảng URL ảnh
price_adult/children/baby,
price_new_adult/children/baby,   -- giá khuyến mãi
stock_adult/children/baby,       -- số lượng chỗ còn
locations (JSON),      -- các điểm đến
destination (JSON),    -- thông tin điểm đến chi tiết
time, vehicle, departure_date,
information (LONGTEXT),-- mô tả HTML
schedules (JSON),      -- lịch trình theo ngày
slug, deleted, deleted_by, deleted_at, created_at, updated_at
```

---

## 6. Controllers

### Client Controllers (`app/Controllers/Client/`)

| File | Chức năng |
|---|---|
| `HomeController.php` | Trang chủ — lấy tour, danh mục, thông tin website |
| `TourController.php` | Trang chi tiết tour theo slug |
| `CategoryController.php` | Danh sách tour theo danh mục (kèm bộ lọc giá, thành phố) |
| `CartController.php` | Trang giỏ hàng (đọc cart từ body JSON) |
| `OrderController.php` | Tạo đơn hàng, thanh toán VNPay, trang thành công |
| `SearchController.php` | Tìm kiếm tour theo từ khóa |
| `ContactController.php` | Gửi email liên hệ |

### Admin Controllers (`app/Controllers/Admin/`)

| File | Chức năng |
|---|---|
| `DashboardController.php` | Thống kê doanh thu, biểu đồ |
| `TourAdminController.php` | CRUD tour, thùng rác, thay đổi hàng loạt |
| `CategoryAdminController.php` | CRUD danh mục |
| `OrderAdminController.php` | Quản lý đơn hàng, cập nhật trạng thái |
| `AccountController.php` | Đăng nhập, đăng ký, quên mật khẩu (OTP email) |
| `SettingController.php` | Cấu hình website, quản lý admin, phân quyền role |
| `ProfileController.php` | Chỉnh sửa hồ sơ, đổi mật khẩu |
| `UserController.php` | Quản lý user (danh sách) |
| `ContactController.php` | Xem danh sách email liên hệ |
| `UploadController.php` | Upload ảnh (trả về JSON URL) |

---

## 7. Middleware

### `AdminAuthMiddleware::verify()`
1. Đọc session → lấy account đã đăng nhập.
2. Nếu chưa đăng nhập → redirect `/admin/account/login`.
3. Nếu có role → load permissions từ bảng `roles` → share vào View và gán vào `$request->permissions`.

### `ClientMiddleware::bootstrap()`
1. Load `SettingWebsiteInfo` (logo, thông tin website) → share vào View.
2. Load toàn bộ danh mục active → build cây danh mục → share vào View.
3. Load danh sách thành phố → share vào View.
4. Xác định nav active slug dựa trên URL hiện tại.

---

## 8. Helpers

| File | Mô tả |
|---|---|
| `AuthHelper.php` | Đọc/ghi session account, xác thực |
| `CategoryHelper.php` | Build cây danh mục (cha-con), lấy type (domestic/international) |
| `UploadHelper.php` | Xử lý upload file ảnh lên `public/uploads/` |
| `MailHelper.php` | Gửi email (PHPMailer, dùng Gmail SMTP) |
| `StrHelper.php` | Tạo slug, format tiền tệ, random string |
| `TourHelper.php` | Hàm tiện ích liên quan tour |
| `GenerateHelper.php` | Tạo mã ngẫu nhiên |

---

## 9. Views (Template Pug)

Dùng template engine **Pug** (Node.js). PHP gọi `node scripts/render-pug.mjs` để render ra HTML.

### Client (`views/client/`)
| File | Trang |
|---|---|
| `pages/home.pug` | Trang chủ |
| `pages/tour-detail.pug` | Chi tiết 1 tour |
| `pages/tour-list.pug` | Danh sách tour |
| `pages/cart.pug` | Giỏ hàng |
| `pages/order-success.pug` | Đặt tour thành công |
| `pages/search.pug` | Kết quả tìm kiếm |
| `layouts/` | Layout chung (header, footer) |
| `partials/` | Các phần tử nhỏ (card tour, …) |
| `mixins/` | Pug mixins tái sử dụng |

### Admin (`views/admin/`)
| File | Trang |
|---|---|
| `pages/dashboard.pug` | Tổng quan, biểu đồ doanh thu |
| `pages/tour-list.pug` | Danh sách tour + bộ lọc |
| `pages/tour-create.pug` | Tạo tour mới |
| `pages/tour-edit.pug` | Chỉnh sửa tour |
| `pages/tour-trash.pug` | Thùng rác tour |
| `pages/category-*.pug` | CRUD danh mục |
| `pages/order-*.pug` | Quản lý đơn hàng |
| `pages/setting-*.pug` | Cấu hình, phân quyền |
| `pages/login.pug` | Đăng nhập admin |
| `pages/register.pug` | Đăng ký admin |
| `pages/forgot-password.pug` | Quên mật khẩu |

---

## 10. Frontend (JS & CSS)

### `public/assets/js/script.js` (32KB)
File JS duy nhất xử lý toàn bộ tương tác phía client:
- **Giỏ hàng:** Lưu/đọc cart trong `localStorage`, cộng tổng tiền.
- **Tìm kiếm & lọc:** Gửi AJAX/form để lọc tour theo giá, thành phố, loại.
- **Admin dashboard:** Vẽ biểu đồ doanh thu, xử lý bảng CRUD.
- **Upload ảnh:** Giao tiếp với `/admin/upload/image` (AJAX multipart).
- **Quản lý hàng loạt:** Checkbox + select action (xóa, đổi trạng thái nhiều item).
- **Thanh toán:** Chuyển sang VNPay.

### `public/assets/css/style.css` (63KB)
File CSS duy nhất cho toàn bộ website (cả client lẫn admin).

---

## 11. Cơ sở dữ liệu

| File | Mô tả |
|---|---|
| `database/schema.sql` | Tạo DB và tất cả bảng |
| `database/seed.sql` | Dữ liệu mẫu ban đầu |
| `database/migrate_destination_json.sql` | Migration đặc biệt cho trường destination |

**Cấu hình DB** đặt trong `.env`:
```
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=tour_db
DB_USERNAME=root
DB_PASSWORD=
```

---

## 12. Cấu hình & Biến môi trường (`.env`)

| Biến | Mô tả |
|---|---|
| `DB_HOST / DB_PORT / DB_DATABASE / DB_USERNAME / DB_PASSWORD` | Kết nối MySQL |
| `JWT_SECRET` | Chữ ký JWT (nếu dùng) |
| `WEBSITE_DOMAIN` | Domain của website |
| `GMAIL_USER / GMAIL_PASS` | Tài khoản Gmail để gửi mail OTP |
| `VNPAY_TMNCODE / VNPAY_SECRET / VNPAY_URL` | Cổng thanh toán VNPay |
| `ZALOPAY_*` | Cổng thanh toán ZaloPay (dự phòng) |

---

## 13. Hướng dẫn cài đặt & chạy dự án

### Yêu cầu hệ thống
- **XAMPP** (PHP ≥ 8.1, MySQL)
- **Node.js** ≥ 16 (để render template Pug)
- **Composer** (quản lý dependency PHP)

### Các bước cài đặt

```bash
# 1. Clone / copy project vào thư mục XAMPP
# Đường dẫn: D:\XAMPP1\htdocs\tour\

# 2. Cài dependency PHP
cd D:\XAMPP1\htdocs\tour
composer install

# 3. Cài dependency Node.js (dùng cho Pug renderer)
npm install
# hoặc: yarn install

# 4. Tạo file .env từ file mẫu
copy .env.example .env
# Chỉnh sửa .env: điền thông tin DB, email, payment

# 5. Tạo database
# Mở phpMyAdmin → Import file: database/schema.sql
# (Tuỳ chọn) Import: database/seed.sql  ← dữ liệu mẫu

# 6. Cấu hình XAMPP Virtual Host
# Trỏ DocumentRoot vào: D:/XAMPP1/htdocs/tour/public
# Hoặc truy cập: http://localhost/tour/public/
```

### Cấu hình Apache (`.htaccess` / Virtual Host)
Vì tất cả request phải đi qua `public/index.php`, cần cấu hình rewrite:
```apache
<VirtualHost *:80>
    DocumentRoot "D:/XAMPP1/htdocs/tour/public"
    ServerName tour.local
    <Directory "D:/XAMPP1/htdocs/tour/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Đường dẫn truy cập
| URL | Trang |
|---|---|
| `http://localhost/` | Trang chủ client |
| `http://localhost/admin` | Trang admin (redirect login) |
| `http://localhost/admin/account/login` | Đăng nhập admin |
| `http://localhost/category/:slug` | Danh mục tour |
| `http://localhost/tour/detail/:slug` | Chi tiết tour |
| `http://localhost/cart` | Giỏ hàng |
| `http://localhost/admin/dashboard` | Dashboard quản trị |

---

## 14. Luồng tính năng chính

### Khách đặt tour
```
Trang chủ → Chọn danh mục / tìm kiếm
    → Trang chi tiết tour
    → Bấm "Đặt tour" → Thêm vào localStorage (giỏ hàng)
    → Trang /cart → Điền thông tin khách hàng
    → POST /order/create → Tạo đơn trong DB
    → Chọn phương thức thanh toán:
        ├── Tiền mặt → /order/success
        └── VNPay → Redirect VNPay → Callback → /order/payment-vnpay-result
```

### Admin quản lý tour
```
Đăng nhập /admin/account/login
    → Dashboard (thống kê, biểu đồ)
    → Tour List: lọc theo trạng thái, danh mục, loại (domestic/international)
    → Tạo / sửa tour: upload ảnh, nhập giá, lịch trình JSON
    → Xóa (soft delete) → Thùng rác → Khôi phục / Xóa vĩnh viễn
```

---

## 15. Lưu ý kỹ thuật quan trọng

> **Soft Delete:** Tất cả bảng dùng cột `deleted = 0/1`. Không xóa vật lý, chỉ đặt `deleted = 1`.

> **JSON Fields:** Các trường `images`, `locations`, `destination`, `schedules`, `permissions` được lưu JSON trong MySQL và tự động encode/decode qua `BaseModel`.

> **Method Override:** Form HTML không hỗ trợ PATCH/DELETE. Dùng `<input type="hidden" name="_method" value="PATCH">` và index.php sẽ override.

> **Render Pug:** PHP gọi `node scripts/render-pug.mjs <pugFile> <tempFile>`. Data được ghi ra file tạm (tránh giới hạn 8192 bytes của Windows argument).

> **Phân quyền:** Admin có role → role có mảng `permissions` (JSON). Middleware đọc và share vào View để Pug template ẩn/hiện tính năng theo quyền.

---

*File này được tạo tự động — cập nhật lần cuối: 2026-07-03*
