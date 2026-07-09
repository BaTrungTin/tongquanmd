# 📋 PHÂN CÔNG BACKEND - DỰ ÁN WEBSITE ĐẶT TOUR DU LỊCH

> **Dự án:** Website Đặt Tour Du Lịch (PHP MVC)
> **Kiến trúc:** Custom PHP MVC (Router → Controller → Model → View)
> **Ngày phân công:** Tháng 7/2026

---

## 🏗️ TỔNG QUAN KIẾN TRÚC

```
public/index.php          ← Entry point duy nhất
    ↓
bootstrap.php             ← Khởi tạo app, load env, autoload
    ↓
routes/web.php            ← Đăng ký tất cả route
    ↓
app/Controllers/          ← Xử lý logic nghiệp vụ
    ├── Admin/            ← Trang quản trị
    └── Client/           ← Trang người dùng
    ↓
app/Models/               ← Truy vấn database (MySQL)
    ↓
views/                    ← Giao diện (Pug/PHP)
```

---

---

# 👤 THÀNH VIÊN 1 — Phụ trách: QUẢN TRỊ HỆ THỐNG (Admin Panel)

## 🎯 Phạm vi công việc
Chịu trách nhiệm toàn bộ **trang quản trị (Admin)**: quản lý tour, danh mục, đơn hàng, tài khoản, cài đặt website.

---

## 📁 Files cần nắm vững

### Controllers (Admin)
| File | Chức năng |
|------|-----------|
| `app/Controllers/Admin/TourAdminController.php` | CRUD tour: thêm, sửa, xóa, xem danh sách |
| `app/Controllers/Admin/CategoryAdminController.php` | CRUD danh mục tour (cha/con) |
| `app/Controllers/Admin/OrderAdminController.php` | Xem & cập nhật trạng thái đơn hàng |
| `app/Controllers/Admin/AccountController.php` | Quản lý tài khoản admin |
| `app/Controllers/Admin/DashboardController.php` | Thống kê tổng quan: doanh thu, đơn hàng |
| `app/Controllers/Admin/SettingController.php` | Cài đặt thông tin website |
| `app/Controllers/Admin/ProfileController.php` | Hồ sơ cá nhân admin |
| `app/Controllers/Admin/ContactController.php` | Xem tin nhắn liên hệ từ khách |
| `app/Controllers/Admin/UploadController.php` | Xử lý upload ảnh |

### Models
| File | Chức năng |
|------|-----------|
| `app/Models/Tour.php` | Query CRUD tour |
| `app/Models/Category.php` | Query CRUD danh mục |
| `app/Models/Order.php` | Query xem & cập nhật đơn hàng |
| `app/Models/AccountAdmin.php` | Query tài khoản admin |
| `app/Models/Role.php` | Query phân quyền |
| `app/Models/Contact.php` | Query tin nhắn liên hệ |
| `app/Models/SettingWebsiteInfo.php` | Query cài đặt web |
| `app/Models/BaseModel.php` | **Class cha** – chứa các hàm query dùng chung |

### Routes liên quan (trong `routes/web.php`)
```
GET  /admin                       → DashboardController
GET  /admin/tours                 → TourAdminController@index
POST /admin/tours/create          → TourAdminController@store
POST /admin/tours/edit            → TourAdminController@update
POST /admin/tours/delete          → TourAdminController@destroy

GET  /admin/categories            → CategoryAdminController@index
POST /admin/categories/create     → CategoryAdminController@store
POST /admin/categories/edit       → CategoryAdminController@update
POST /admin/categories/delete     → CategoryAdminController@destroy

GET  /admin/orders                → OrderAdminController@index
POST /admin/orders/edit           → OrderAdminController@update

GET  /admin/settings              → SettingController@index
POST /admin/settings/edit         → SettingController@update
```

---

## 📚 Kiến thức cần học

### 1. PHP Căn Bản (bắt buộc)
- [ ] Biến, mảng, hàm, vòng lặp
- [ ] Hàm xử lý chuỗi: `trim()`, `htmlspecialchars()`, `str_replace()`
- [ ] Hàm xử lý mảng: `array_map()`, `array_filter()`
- [ ] `$_POST`, `$_GET`, `$_FILES`, `$_SESSION`
- [ ] OOP PHP: Class, kế thừa (`extends`), `$this`, access modifier

### 2. MySQL & PDO (bắt buộc)
- [ ] Câu lệnh SELECT, INSERT, UPDATE, DELETE
- [ ] JOIN bảng (INNER JOIN, LEFT JOIN)
- [ ] Prepared Statements (chống SQL Injection)
- [ ] Kết nối DB qua PDO trong PHP
- [ ] `ORDER BY`, `LIMIT`, `WHERE`, `GROUP BY`

### 3. Kiến trúc MVC (bắt buộc)
- [ ] Flow request: Route → Controller → Model → View
- [ ] Cách đọc `routes/web.php` để biết route nào gọi controller nào
- [ ] Cách `BaseModel.php` hoạt động (hàm `find`, `findAll`, `create`, `update`, `delete`)

### 4. Upload File (cần cho quản lý Tour)
- [ ] Xử lý `$_FILES` trong PHP
- [ ] Validate loại file & kích thước
- [ ] Lưu file vào `public/uploads/`

### 5. Xác thực & Session (cần cho Admin)
- [ ] `session_start()`, `$_SESSION`
- [ ] Kiểm tra đăng nhập trong Middleware
- [ ] Phân quyền Admin vs User thường

---

## ✅ Nhiệm vụ cụ thể cần hoàn thành

```
[ ] Hiểu và giải thích được luồng: thêm 1 tour mới từ form → lưu DB
[ ] Giải thích cách phân cấp danh mục (cha/con) hoạt động
[ ] Hiểu OrderAdminController: cách thay đổi trạng thái đơn
[ ] Nắm được DashboardController tính thống kê thế nào
[ ] Giải thích BaseModel.php hoạt động ra sao
```

---
---

# 👤 THÀNH VIÊN 2 — Phụ trách: TRẢI NGHIỆM NGƯỜI DÙNG (Client Side)

## 🎯 Phạm vi công việc
Chịu trách nhiệm toàn bộ **trang người dùng (Client)**: trang chủ, xem tour, giỏ hàng, đặt hàng, đăng nhập, tìm kiếm.

---

## 📁 Files cần nắm vững

### Controllers (Client)
| File | Chức năng |
|------|-----------|
| `app/Controllers/Client/HomeController.php` | Trang chủ: hiển thị tour nổi bật, danh mục |
| `app/Controllers/Client/TourController.php` | Xem chi tiết 1 tour |
| `app/Controllers/Client/CategoryController.php` | Lọc tour theo danh mục |
| `app/Controllers/Client/SearchController.php` | Tìm kiếm tour theo từ khóa |
| `app/Controllers/Client/CartController.php` | Giỏ hàng: thêm/xóa tour |
| `app/Controllers/Client/OrderController.php` | Đặt tour, xem lịch sử đơn hàng, email xác nhận |
| `app/Controllers/Client/ContactController.php` | Gửi tin nhắn liên hệ |

### Models
| File | Chức năng |
|------|-----------|
| `app/Models/Tour.php` | Query tìm kiếm, lọc, lấy tour theo ID/danh mục |
| `app/Models/Order.php` | Query tạo đơn hàng, lấy lịch sử |
| `app/Models/Category.php` | Query lấy danh mục để hiển thị menu |
| `app/Models/City.php` | Query danh sách tỉnh/thành phố |
| `app/Models/ForgotPassword.php` | Query token đặt lại mật khẩu |
| `app/Models/BaseModel.php` | **Class cha** – chứa các hàm query dùng chung |

### Các file hỗ trợ khác
| File | Chức năng |
|------|-----------|
| `app/Helpers/` | Các hàm tiện ích dùng chung |
| `app/Middleware/` | Kiểm tra đăng nhập trước khi vào trang |
| `app/Core/` | Router, Request, Response (không cần sửa) |

### Routes liên quan (trong `routes/web.php`)
```
GET  /                            → HomeController@index
GET  /tours/:slug                 → TourController@show
GET  /categories/:slug            → CategoryController@show
GET  /search                      → SearchController@index

GET  /cart                        → CartController@index
POST /cart/add                    → CartController@add
POST /cart/remove                 → CartController@remove

GET  /orders                      → OrderController@index
POST /orders/create               → OrderController@store
GET  /orders/:code                → OrderController@show

GET  /contact                     → ContactController@index
POST /contact/send                → ContactController@store
```

---

## 📚 Kiến thức cần học

### 1. PHP Căn Bản (bắt buộc)
- [ ] Biến, mảng, hàm, vòng lặp
- [ ] `$_POST`, `$_GET`, `$_SESSION`, `$_COOKIE`
- [ ] OOP PHP: Class, kế thừa, `$this`
- [ ] Hàm xử lý chuỗi và mảng thông dụng

### 2. MySQL & PDO (bắt buộc)
- [ ] Câu lệnh SELECT với WHERE, ORDER BY, LIMIT
- [ ] JOIN bảng để lấy thông tin liên quan
- [ ] Prepared Statements (chống SQL Injection)
- [ ] `ORDER BY RAND() LIMIT 6` (dùng cho tour ngẫu nhiên trang chủ)

### 3. Session & Giỏ Hàng (quan trọng)
- [ ] `session_start()`, `$_SESSION`
- [ ] Cách lưu giỏ hàng vào Session
- [ ] Thêm/xóa sản phẩm trong Session array

### 4. Xử lý Đơn Hàng (quan trọng)
- [ ] Luồng đặt hàng: Giỏ hàng → Nhập thông tin → Xác nhận → Lưu DB
- [ ] Tạo mã đơn hàng unique (`uniqid()`, `md5()`)
- [ ] Gửi email xác nhận đơn hàng (PHPMailer/SMTP)

### 5. Tìm Kiếm & Lọc (cần hiểu)
- [ ] Xây dựng câu query động theo điều kiện lọc
- [ ] Xử lý tham số trên URL: `$_GET['keyword']`, `$_GET['city']`
- [ ] Phân trang kết quả (LIMIT + OFFSET)

### 6. Bảo mật cơ bản
- [ ] Validate dữ liệu đầu vào từ form
- [ ] `htmlspecialchars()` để chống XSS
- [ ] Kiểm tra Middleware trước các trang cần đăng nhập

---

## ✅ Nhiệm vụ cụ thể cần hoàn thành

```
[ ] Giải thích luồng: khách đặt 1 tour từ đầu đến cuối
[ ] Giải thích cách giỏ hàng lưu trong Session
[ ] Hiểu SearchController lọc tour theo điều kiện thế nào
[ ] Giải thích OrderController gửi email xác nhận thế nào
[ ] Nắm CategoryController hiển thị tour theo danh mục con thế nào
```

---
---

## 🔗 PHẦN DÙNG CHUNG (Cả 2 cần hiểu)

| File/Thư mục | Người cần nắm |
|---|---|
| `bootstrap.php` | Cả 2 — khởi tạo app, load env |
| `routes/web.php` | Cả 2 — xem route nào đi đâu |
| `app/Models/BaseModel.php` | Cả 2 — hàm query tái sử dụng |
| `app/Core/` | Cả 2 — hiểu cơ bản, không cần sửa |
| `config/database.php` | Cả 2 — cấu hình kết nối DB |
| `.env` | Cả 2 — biến môi trường (DB, SMTP...) |
| `database/` | Cả 2 — schema SQL của dự án |

---

## 💬 GHI CHÚ THÊM

> **Về công cụ debug:**
> - Dùng `var_dump()` hoặc `print_r()` để in giá trị khi debug
> - Bật error reporting: `error_reporting(E_ALL)` trong file `.env`

> **Về database:**
> - Tool quản lý: **phpMyAdmin** (tại `localhost/phpmyadmin`)
> - Tên database xem trong file `.env`

> **Về môi trường:**
> - Chạy local bằng **XAMPP**
> - Web root: `htdocs/tour/public/`
> - PHP version: 8.x+

> **Cách test nhanh một route:**
> Mở trình duyệt và vào `http://localhost/tour/public/[route]`
