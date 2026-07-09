# 📘 GIẢI THÍCH CHI TIẾT CÁC FILE CODE CỦA NGƯỜI THỨ 2 (CLIENT BACKEND)

Tài liệu này giải thích chi tiết chức năng, ý nghĩa và logic của từng file (Controller & Model) được phân công cho người phụ trách mảng Client (Người thứ 2). Bạn có thể dùng tài liệu này để tự tin trả lời khi thầy cô yêu cầu "Mở file X lên và giải thích code".

---

## 1. CÁC FILE CONTROLLER (NẰM TẠI `app/Controllers/Client/`)

Đây là nơi nhận Request từ trình duyệt, xử lý logic, gọi Model lấy data và gọi View để render giao diện.

### 1.1. `HomeController.php`
- **Chức năng:** Quản lý dữ liệu hiển thị cho Trang chủ (`/`).
- **Logic code chính:**
  - Lấy danh sách tour nổi bật (Section 2, 4, 6) thông qua hàm `Tour::find()` với điều kiện `deleted = false` và `status = 'active'`, giới hạn số lượng (`limit`).
  - Sử dụng `TourHelper::enrichTourList()` để đính kèm thêm dữ liệu phụ cho các tour trước khi hiển thị (chuyển đổi ngày tháng, format giá).
  - Lấy danh sách "Các điểm đến Hot" (Hot City): Code lặp qua danh sách thành phố (`City::find()`), đếm số lượng tour có ở thành phố đó (`Tour::countByDestination`), nếu thành phố nào có tour thì lưu vào danh sách, sau đó dùng hàm `shuffle()` trộn ngẫu nhiên và lấy ra 6 thành phố hiển thị ở phần tìm kiếm.

### 1.2. `TourController.php`
- **Chức năng:** Quản lý trang Chi tiết 1 Tour (`/tour/detail/:slug`).
- **Logic code chính:**
  - Lấy `$slug` từ URL, tìm thông tin tour bằng `Tour::findOne(['slug' => $slug])`. Nếu không thấy, đẩy về trang chủ.
  - Gọi `CategoryHelper::getCategoryParent()` để lấy ra dải Breadcrumb (Ví dụ: Trang chủ > Châu Á > Nhật Bản > Tên Tour) dựa vào ID Category của tour.
  - Format lại ngày tháng (`departureDate`) sang dạng `d/m/Y`.
  - Lấy danh sách các Thành phố (City) đi qua trong tour này từ cột `locations` (vốn lưu dưới dạng JSON mảng ID) và gán tên thành phố để hiển thị ra cho khách chọn "Điểm khởi hành".

### 1.3. `CategoryController.php`
- **Chức năng:** Hiển thị danh sách các tour thuộc một Danh mục (`/category/:slug`).
- **Logic code chính:**
  - Code phức tạp nhất ở đây là đệ quy danh mục con: Gọi `CategoryHelper::getCategoryChild()` để gom tất cả ID của danh mục con. Ví dụ vào "Miền Bắc" thì phải hiển thị cả tour của "Hà Nội", "Sapa", "Hạ Long". Code nhét mảng ID này vào `$query['categoryIds']`.
  - Quyết định danh sách "Thành phố khởi hành": Code kiểm tra xem danh mục gốc là "Trong nước" hay "Nước ngoài". Nếu "Nước ngoài", nó chỉ cho phép chọn khởi hành từ 3 sân bay lớn (ID 1, 2, 3 tương ứng HN, HCM, Đà Nẵng).

### 1.4. `SearchController.php`
- **Chức năng:** Xử lý kết quả tìm kiếm đa điều kiện (`/search`).
- **Logic code chính:**
  - Lấy toàn bộ tham số người dùng truyền lên URL (`$request->query()`).
  - Hàm `normalizeQuery()`: Đôi khi người dùng gõ chữ "Hà Nội" thay vì chọn ID thành phố. Hàm này gọi `City::findByName()` để lấy ID của Hà Nội gán ngược lại vào câu query.
  - Đẩy toàn bộ cục `$query` đó vào `Tour::search($query)` để Model tự xử lý SQL.

### 1.5. `CartController.php`
- **Chức năng:** Quản lý xác thực giỏ hàng.
- **Logic code chính:**
  - Hàm `cart()`: Đơn giản là gọi View render giao diện giỏ hàng.
  - Hàm `detailPost()` (API): Trình duyệt gửi lên mảng các ID Tour trong LocalStorage. Controller dùng vòng lặp `foreach` query lại `Tour::findOne()` để lấy **Giá tiền mới nhất** và **Số lượng vé hiện còn (stock)**. Sau đó bọc thành JSON trả về để JS cập nhật giao diện. (Tuyệt đối không lấy giá do LocalStorage gửi lên).

### 1.6. `OrderController.php`
- **Chức năng:** Xử lý tạo đơn hàng, trừ kho, gửi mail, thanh toán VNPay.
- **Logic code chính:**
  - Hàm `createPost()`: Tự động sinh mã đơn `OD...`, query DB lấy giá gốc, nhân số lượng ra `subTotal`. Gọi `Tour::updateOne` để **trừ kho** (`stockAdult - quantityAdult`). Lưu đơn vào bảng `orders`. Gọi `MailHelper` gửi email xác nhận.
  - Hàm `paymentVNPay()`: Nếu khách chọn VNPay, code nhặt các cấu hình `vnpay_tmncode`, `vnpay_secret` từ `.env`. Sắp xếp mảng tham số, tạo mã băm Hash bằng thuật toán `sha512` và chuyển hướng (Redirect) khách sang trang VNPay.
  - Hàm `paymentVNPayResult()`: Trang VNPay trả khách về. Code lấy mảng params VNPay trả ra, **tự tạo lại mã băm Hash một lần nữa** và so sánh với `vnp_SecureHash` của VNPay. Khớp thì update trạng thái `paid`.

### 1.7. `ContactController.php`
- **Chức năng:** Xử lý khách đăng ký nhận bản tin hoặc gửi liên hệ.
- **Logic code chính:** Nhận email, check `Contact::findOne()` xem đã đăng ký chưa, chưa thì `Contact::insert()`. Trả về JSON.

---

## 2. CÁC FILE MODEL (NẰM TẠI `app/Models/`)

Đây là tầng giao tiếp trực tiếp với cơ sở dữ liệu MySQL, được thiết kế theo cấu trúc kế thừa (OOP) rất bài bản để tái sử dụng code.

### 2.1. `BaseModel.php` (Class Cha - Rất quan trọng)
Tất cả các Model khác đều `extends BaseModel`. Nó chứa các hàm dùng chung:
- `buildWhere($filters)`: Tự động phân tích mảng dữ liệu thành chuỗi `WHERE field = ? AND field2 = ?`.
- `findOne()`, `find()`: Các hàm gọi dữ liệu, sắp xếp (`ORDER BY`), phân trang (`LIMIT`).
- `insert()`, `updateOne()`, `deleteOne()`: Các hàm xử lý thay đổi dữ liệu.
- **Bảo mật:** Tất cả các hàm này đều gom giá trị biến vào mảng `$params` và dùng `$stmt->execute($params)` của PDO (Prepared Statements). Điều này giúp triệt tiêu hoàn toàn lỗi **SQL Injection**.
- `$jsonFields`: Mảng đặc biệt chứa tên các cột lưu kiểu JSON trong DB. `BaseModel` sẽ tự động `json_decode` các cột này khi truy vấn ra để dễ dùng.

### 2.2. `Tour.php`
- **Map Dữ Liệu:** Hàm `map()` khai báo các cột ép kiểu rõ ràng (cột giá thành `float`, tồn kho thành `int`). Khai báo `$jsonFields = ['images', 'locations', 'destination', 'schedules']` để BaseModel tự parse mảng JSON.
- **Dynamic Query (Hàm `search`):** Hàm mạnh nhất của Model này. Khởi tạo câu SQL `SELECT * FROM tours...`. Dùng `if` để nối chuỗi điều kiện `AND` tùy theo việc Controller truyền xuống tham số gì (Lọc theo Giá, Lọc theo Sức chứa, Điểm đến). Đặc biệt dùng `JSON_CONTAINS(locations, ?, "$")` để tìm trong mảng JSON của MySQL.

### 2.3. `Order.php`
- Tương tự Tour, có `jsonFields = ['items']` để tự động parse danh sách tour khách đặt từ chuỗi JSON thành Mảng khi gọi ra View. Tính toán các kiểu dữ liệu `subTotal`, `total` là `float`.

### 2.4. `Category.php`
- Map các trường dữ liệu của danh mục: `parent` (ID của danh mục cha), `slug`, `avatar`. Phục vụ việc đệ quy trong CategoryController.

### 2.5. `City.php`
- Map tên và ID thành phố.
- Cung cấp thêm hàm `findByName(string $name)`: Sử dụng `LOWER(name) = LOWER(?)` để tìm kiếm tên thành phố không phân biệt chữ hoa chữ thường. Giúp SearchController đổi từ khóa chữ ("Hà Nội") sang ID thành phố hợp lệ.

---
*Khi thầy cô hỏi, bạn chỉ cần đọc qua tài liệu này, mở đúng file tương ứng lên, chỉ tay vào dòng code và giải thích giống như các gạch đầu dòng ở trên. Đảm bảo thầy cô sẽ gật gù hài lòng!*
