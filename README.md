# Workshop Booking Portal (PHPLab02)

**Môn học:** PHP / Web Back-end Fundamentals  
**Chủ đề:** Form Input, Validation, Sanitization, Cookies & Sessions  
**Sinh viên thực hiện:** Phạm Quốc Thịnh  
**Mã số sinh viên (MSSV):** 24110247  
**GitHub Repository:** [https://github.com/PhamQuocThinhPQT/PHPLab02](https://github.com/PhamQuocThinhPQT/PHPLab02)  

---

## 1. Giới thiệu dự án

Workshop Booking Portal là ứng dụng web xây dựng bằng PHP thuần (vanilla PHP), minh họa toàn diện các cơ chế nền tảng của Web Back-end:
- **GET Request & Query Parameters:** Tìm kiếm theo từ khóa (`q`), lọc danh mục workshop whitelist (`category`).
- **POST Request & HTTP Verbs:** Gửi dữ liệu form qua body, phân biệt `$_GET`, `$_POST`, `$_REQUEST` và `$_SERVER['REQUEST_METHOD']`.
- **Validation & Sanitization:** Xử lý và kiểm tra dữ liệu nghiêm ngặt phía server (`normalize_text`, `filter_var`, độ dài byte, giới hạn số vé và kiểm tra khóa hợp lệ).
- **Output Escaping:** Sử dụng hàm `e()` bọc `htmlspecialchars` với cờ `ENT_QUOTES | ENT_SUBSTITUTE` chuẩn mã hóa UTF-8 chống lỗ hổng Cross-Site Scripting (XSS).
- **Session State & PRG Pattern:** Lưu trữ danh sách đặt chỗ tạm thời trong `$_SESSION['bookings']`, chống gửi lặp form (duplicate submission) bằng kỹ thuật POST/Redirect/GET (HTTP 303 See Other).
- **CSRF Defense:** Bảo vệ tất cả các thao tác thay đổi trạng thái (state-changing) bằng token chống Cross-Site Request Forgery (CSRF) và so sánh an toàn thời gian qua `hash_equals()`.
- **Theme Cookies:** Quản lý lựa chọn giao diện sáng/tối (Light/Dark theme) lưu trong cookie trình duyệt thời hạn 30 ngày với cờ `HttpOnly`, `SameSite=Lax`.
- **Hai chức năng mở rộng mới:**
  - **Chức năng A (View Booking Detail - `booking_detail.php`):** Đọc booking theo mã tham chiếu `ref` qua GET an toàn, hiển thị chi tiết không làm thay đổi trạng thái server (idempotent / safe method). Xử lý an toàn khi `ref` sai/thiếu.
  - **Chức năng B (Edit Booking - `booking_edit.php`):** Tải dữ liệu booking qua GET, cập nhật qua POST có CSRF và server-side validation. Tính toán lại tổng tiền dựa trên đơn giá catalog ở server (chống giả mạo client).

---

## 2. Cấu trúc thư mục dự án

```text
C:\xampp\htdocs\PHPLab02\
├── index.php             # Trang chủ catalog, tìm kiếm GET, đổi theme
├── booking.php           # Form đặt vé mới (GET hiển thị, POST xử lý & validate)
├── booking_detail.php    # [Chức năng mới A] Xem chi tiết booking qua GET ?ref=
├── booking_edit.php      # [Chức năng mới B] Sửa booking (GET tải, POST lưu & recalculate)
├── dashboard.php         # Trang My Bookings liệt kê danh sách đặt chỗ từ session
├── preference.php        # Action POST lưu cookie theme (Light / Dark)
├── logout.php            # Action POST xóa sạch Session và hủy session cookie
├── request_demo.php      # Trang demo phân biệt $_GET, $_POST, $_REQUEST, REQUEST_METHOD
├── README.md             # Tài liệu hướng dẫn và mô tả hệ thống
├── lib/
│   ├── bootstrap.php     # Khởi tạo session, nạp helper & catalog, quản lý CSRF & theme
│   └── functions.php     # Các hàm tiện ích: e(), normalize_text(), validate_booking(), ...
├── data/
│   └── workshops.php     # Dữ liệu nguồn danh mục workshop (associative array)
└── assets/
    └── style.css         # Stylesheet responsive, hỗ trợ Light & Dark theme
```

---

## 3. Hướng dẫn cài đặt và chạy ứng dụng

### Cách 1: Chạy qua XAMPP Apache
1. Đảm bảo mã nguồn nằm tại thư mục `C:\xampp\htdocs\PHPLab02`.
2. Khởi động Apache trong **XAMPP Control Panel**.
3. Mở trình duyệt (hoặc cửa sổ ẩn danh) và truy cập đường dẫn:
   ```text
   http://localhost/PHPLab02/
   ```

### Cách 2: Chạy qua PHP Built-in Server
1. Mở cửa sổ dòng lệnh (Terminal / PowerShell) tại thư mục `C:\xampp\htdocs\PHPLab02`.
2. Thực thi lệnh:
   ```bash
   php -S localhost:8000
   ```
3. Mở trình duyệt và truy cập:
   ```text
   http://localhost:8000/
   ```

---

## 4. Test Matrix & Kết quả kiểm thử

| Test ID | Thao tác thực hiện | Kỳ vọng (Expected Result) | Thực tế (Actual Result) | Đánh giá |
| :--- | :--- | :--- | :--- | :---: |
| **T01** | View ref hợp lệ (`?ref=BK-...`) | Hiển thị đầy đủ, chính xác thông tin bản ghi | Hiển thị đúng ref, attendee, workshop, qty, fee, amount | **PASS** |
| **T02** | View ref không tồn tại hoặc rỗng | Báo "Booking not found", không phát sinh PHP Notice/Warning | Hiện panel "Booking not found", có nút quay lại My Bookings | **PASS** |
| **T03** | Edit hợp lệ: thay đổi số vé (2 → 3) | Amount tính lại theo đơn giá server; cập nhật đúng ref | Amount = unit_fee × 3; flash message thông báo thành công | **PASS** |
| **T04** | Edit email không đúng định dạng | Từ chối, báo lỗi inline, không đổi Session | Báo "Invalid email format.", Session giữ nguyên giá trị cũ | **PASS** |
| **T05** | Edit số vé sai (0, 5, 1.5) | Bị từ chối với thông báo "Quantity must be 1 to 4" | Lỗi hiển thị tại trường Tickets, form giữ dữ liệu nhập | **PASS** |
| **T06** | POST thiếu hoặc sai CSRF token | Server trả về mã HTTP 403 Forbidden | Trả về HTTP 403 Forbidden "Invalid CSRF token" | **PASS** |
| **T07** | Nhấn Refresh (F5) sau khi cập nhật thành công | Không xuất hiện hộp thoại xác nhận gửi lại POST (PRG) | Trình duyệt chỉ gửi lại GET tới trang đích (HTTP 303 PRG) | **PASS** |
| **T08** | Tạo 2 booking, chỉ chỉnh sửa 1 booking | Chỉ bản ghi mục tiêu thay đổi, bản ghi còn lại nguyên vẹn | Booking được chọn cập nhật đúng, booking kia giữ nguyên | **PASS** |
| **T09** | Mở trang My Bookings trên cửa sổ ẩn danh (Private) | Không nhìn thấy session của trình duyệt chính | Dashboard báo "No bookings yet" (phiên cô lập hoàn toàn) | **PASS** |

---

## 5. Bản quyền & Cam kết học thuật

Dự án được xây dựng và hoàn thiện bởi sinh viên **Phạm Quốc Thịnh** (MSSV: **24110247**) tuân thủ đầy đủ chuẩn mực học thuật, cam kết thực hiện độc lập dựa trên tài liệu hướng dẫn và đặc tả yêu cầu môn học.
