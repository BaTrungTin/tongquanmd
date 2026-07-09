# 🎤 TÀI LIỆU ÔN TẬP THUYẾT TRÌNH BẢO VỆ ĐỒ ÁN (PHẦN CLIENT BACKEND)

Tài liệu này tóm tắt lại các điểm sáng nhất trong phần Backend của Client (trang người dùng) và tổng hợp những câu hỏi "hóc búa" mà giảng viên thường hay hỏi nhất khi bảo vệ đồ án, kèm theo cách trả lời ghi điểm và đường dẫn file để bạn mở ra show cho thầy cô xem lúc bảo vệ.

---

## 📌 TÓM TẮT NHỮNG ĐIỂM SÁNG TRONG CODE ĐỂ BẠN TỰ HÀO "KHOE" KHI THUYẾT TRÌNH:

1. **Kiến trúc linh hoạt:** Không dùng giỏ hàng bằng PHP Session cổ điển. Dự án dùng **LocalStorage kết hợp AJAX (Fetch API)** giúp web cực mượt nhưng vẫn đảm bảo tính bảo mật khi đối soát giá tiền trực tiếp từ Database.
2. **Dynamic Query (Truy vấn động):** Trong chức năng Tìm kiếm/Lọc tour, code không fix cứng SQL mà viết thuật toán tự động nối chuỗi `AND` tùy theo việc khách hàng lọc bao nhiêu điều kiện.
3. **Bảo mật SQL Injection:** Toàn bộ dữ liệu truyền vào Model đều bọc qua **Prepared Statement (`?`)** trong PDO, không bao giờ nối chuỗi biến trực tiếp vào SQL.
4. **Đệ quy Danh mục:** Thuật toán tự tìm tất cả danh mục con để query `IN (ids)` giúp click vào "Châu Á" thì ra cả tour "Hàn Quốc", "Nhật Bản".
5. **Thanh toán bảo mật (VNPay):** Hiểu rõ cơ chế xác thực Hash Signature (chữ ký số HMAC-SHA512) của VNPay để chống hacker can thiệp làm giả kết quả thanh toán.

---

## ❓ CÁC CÂU HỎI THƯỜNG GẶP VÀ CÁCH TRẢ LỜI

### Câu 1: Em lưu giỏ hàng ở đâu? Lỡ người dùng đổi giá tiền trong giỏ hàng thành 0đ rồi mới gửi lên Server thì sao?
**Cách trả lời:**
- "Dạ, để web mượt và không tạo rác trong DB, em lưu tạm giỏ hàng ở **LocalStorage** của trình duyệt. Nó chỉ lưu mã Tour (ID) và Số lượng khách chọn."
- "Tuy nhiên, em **KHÔNG BAO GIỜ tin tưởng giá tiền gửi lên từ Client**. Khi khách ấn nút 'Đặt Tour', Server chỉ lấy ID Tour và Số lượng, sau đó Server sẽ **tự động chọc vào Database (bảng tours) để lấy ra giá gốc mới nhất** rồi nhân lên. Nên dù hacker có sửa giá ở LocalStorage thành 0đ thì khi lên Server cũng sẽ bị tính lại về giá chuẩn ạ."
> **📁 File liên quan để mở cho thầy xem:** 
> - File xử lý Backend: `app/Controllers/Client/OrderController.php` (Hàm `createPost`)
> - File xử lý Frontend lưu LocalStorage: `public/assets/js/script.js` 

### Câu 2: Em làm chức năng Lọc (Search) tour như thế nào? Nếu người dùng chỉ chọn giá mà không chọn địa điểm thì code em xử lý ra sao?
**Cách trả lời:**
- "Dạ em dùng kỹ thuật **Dynamic Query (Truy vấn động)** trong file `Tour::search()`. Em khởi tạo câu SQL gốc là `SELECT * FROM tours WHERE deleted = 0`. Sau đó, em lấy các tham số từ URL xuống, nếu tham số nào tồn tại (`!empty`) thì em mới lấy câu SQL nối thêm (`.=`) với mệnh đề `AND`. 
- Nhờ vậy, nếu user bỏ trống địa điểm thì SQL không có câu lệnh check địa điểm, hệ thống vẫn linh hoạt trả về đúng kết quả lọc theo giá."
> **📁 File liên quan để mở cho thầy xem:** 
> - Nơi gọi hàm: `app/Controllers/Client/SearchController.php`
> - Nơi chứa thuật toán tạo chuỗi SQL động: `app/Models/Tour.php` (Hàm `search`)

### Câu 3: Làm sao để em chống được SQL Injection trong trang web này?
**Cách trả lời:**
- "Dạ tất cả các truy vấn Database của bọn em đều dùng thư viện **PDO của PHP** và bắt buộc phải dùng **Prepared Statements**."
- "Thay vì đưa thẳng biến vào câu SQL (rất dễ bị hacker gõ mã độc `OR 1=1`), code của bọn em thay các giá trị đó bằng dấu chấm hỏi `?`. Sau đó truyền các biến vào 1 mảng `$params` và gọi lệnh `$stmt->execute($params)`. Kỹ thuật này tách biệt hoàn toàn câu lệnh SQL và dữ liệu, giúp triệt tiêu 100% lỗi SQL Injection ạ."
> **📁 File liên quan để mở cho thầy xem:** 
> - Truy vấn động an toàn: `app/Models/Tour.php` (Hàm `search`)
> - Các hàm cơ bản an toàn (`findOne`, `insert`, `update`): `app/Models/BaseModel.php`

### Câu 4: Giải thích luồng hoạt động của chức năng Thanh toán VNPay? Làm sao biết thanh toán thành công thật hay do hacker tự gõ URL báo thành công?
**Cách trả lời:**
- "Luồng VNPay của em gồm 2 bước. Bước 1: Khi tạo đơn, Backend tạo ra một mã băm (Hash) dựa trên tổng tiền + mã đơn + Secret Key (mã bí mật chỉ Server và VNPay biết). Sau đó redirect khách qua web VNPay."
- "Bước 2: Khi khách thanh toán xong, VNPay gọi lại trang web của em (hàm `paymentVNPayResult`) kèm theo mã Response Code và 1 chuỗi Hash mới của họ. Server của em phải **tự tạo lại mã Hash từ dữ liệu nhận được, so sánh với mã Hash VNPay gửi**. Chỉ khi 2 chuỗi Hash này khớp nhau hoàn toàn 100% VÀ Response Code là '00' thì em mới cập nhật trạng thái đơn là Đã thanh toán (`paid`). Nên hacker không có Secret Key sẽ không thể tạo link giả để lừa Server được ạ."
> **📁 File liên quan để mở cho thầy xem:** 
> - Nơi xử lý chữ ký Hash bảo mật: `app/Controllers/Client/OrderController.php` (Hàm `paymentVNPay` và `paymentVNPayResult`)

### Câu 5: Khi click vào một danh mục cha (Ví dụ: Châu Á), làm sao để web hiện ra được cả các tour thuộc danh mục con (Hàn Quốc, Nhật Bản...)?
**Cách trả lời:**
- "Dạ vì mỗi tour thường chỉ lưu ID của danh mục con trực tiếp của nó. Nên khi user click vào danh mục cha, Server của em sẽ gọi hàm `CategoryHelper::getCategoryChild()` để đệ quy tìm TẤT CẢ các danh mục con trực thuộc nó."
- "Sau đó, em gom ID của danh mục cha + các ID của danh mục con vào một mảng. Cuối cùng truyền mảng này vào câu truy vấn SQL dùng mệnh đề `WHERE category IN (?, ?, ?)` để lấy ra mọi tour liên quan ạ."
> **📁 File liên quan để mở cho thầy xem:** 
> - Nơi xử lý logic lấy id danh mục: `app/Controllers/Client/CategoryController.php` (Hàm `list`)
> - Hàm đệ quy danh mục con: `app/Helpers/CategoryHelper.php` (Hàm `getCategoryChild`)

### Câu 6: Request từ trình duyệt đi theo luồng (flow) nào trong mô hình MVC của em?
**Cách trả lời:**
- "Dạ, luồng đi chuẩn của trang web em là: Khách hàng gõ URL -> file `routes/web.php` bắt URL đó và điều hướng vào đúng **Controller** tương ứng (vd: `TourController`)."
- "Tại Controller, code sẽ gọi đến **Model** (ví dụ `Tour::findOne()`) để lấy dữ liệu từ MySQL."
- "Model trả dữ liệu về cho Controller. Controller bọc dữ liệu lại rồi đẩy ra cho file **View (Pug)** render thành giao diện HTML trả về cho người dùng ạ."
> **📁 File liên quan để mở cho thầy xem (Trình diễn 1 luồng ví dụ):** 
> 1. Router bắt URL: `routes/web.php`
> 2. Controller xử lý: `app/Controllers/Client/TourController.php`
> 3. Model lấy data: `app/Models/Tour.php`
> 4. View hiển thị: `views/client/pages/tour-detail.pug`

---
*Chúc bạn thuyết trình thật tự tin và đạt điểm tuyệt đối nhé! Cứ nắm chắc nguyên lý "KHÔNG tin tưởng Client" và "Lấy data từ DB ra tính toán" là sẽ vượt qua mọi câu hỏi hóc búa của thầy cô! Hãy mở đúng file ra cho thầy cô xem lúc trả lời để tăng tính thuyết phục nhé!*
