# BÀI LAB 18: KÍCH HOẠT VÀ TỐI ƯU HÓA AWS SECURITY HUB

+ **Bối cảnh thực tế:** Trong các mô hình vận hành doanh nghiệp lớn, việc kiểm tra các lỗ hổng bảo mật hạ tầng và giám sát tính tuân thủ (Compliance) thường bị phân mảnh do kỹ sư phải chuyển đổi qua lại giữa hàng loạt công cụ (Tường lửa, Endpoint bảo vệ, Quét mã lỗi).
+ **Giải pháp:** **AWS Security Hub** ra đời làm một cổng dashboard tập trung duy nhất, tự động hóa quá trình quét, gom tụ và trực quan hóa các rủi ro kỹ thuật theo các tiêu chuẩn ngành bảo mật đám mây.
+ **Mục tiêu thực hành:** Thực hiện cấu hình kích hoạt **AWS Security Hub** trên giao diện Console và kích hoạt bộ quy chuẩn đánh giá tự động liên tục.

---

### I. Cấu hình kích hoạt AWS Config (Điều kiện tiền đề)

+ **Mục tiêu:** **Security Hub** hoạt động dựa trên việc đánh giá trạng thái tài nguyên được ghi nhận bởi **AWS Config**. Do đó, bắt buộc phải kích hoạt **AWS Config** trước khi bật **Security Hub**.
+ **Các bước thực hiện:**
  1. Trên thanh tìm kiếm của **AWS Management Console**, gõ **AWS Config** và truy cập vào dịch vụ.
  2. Tại trang chào mừng, nhấn chọn nút **1-click setup** để áp dụng cấu hình mặc định một cách nhanh chóng.
  3. Hệ thống sẽ tự động tạo một **S3 Bucket** trung gian (Ví dụ: `config-bucket-2025...`) để lưu trữ cấu hình.
  4. Nhấn **Confirm** để hoàn tất cấu hình. Đảm bảo trạng thái hệ thống báo xanh: *"Service linked role created successfully"* và *"S3 bucket created successfully"*.

---

### II. Kích hoạt AWS Security Hub trên Console

+ **Mục tiêu:** Kích hoạt dịch vụ **Security Hub** tại Region đang vận hành hạ tầng (Ví dụ: `us-east-1` or `ap-southeast-1`).
+ **Các bước thực hiện:**
  1. Quay trở lại thanh tìm kiếm Console, nhập từ khóa **Security Hub** và click chọn dịch vụ.
  2. Tại giao diện giới thiệu **AWS Security Hub**, nhấn nút màu cam: **Go to Security Hub**.
  3. Hệ thống chuyển tới trang **Enable AWS Security Hub**. Tại đây, AWS tự động cấu hình các quyền hạn **Service-linked role** cần thiết để **Security Hub** có thể thu thập dữ liệu tài nguyên.

---

### III. Lựa chọn các tiêu chuẩn bảo mật (Security Standards)

+ **Mục tiêu:** Tích chọn các bộ quy tắc tiêu chuẩn để **Security Hub** làm thước đo chấm điểm cho tài khoản.
+ **Các bước thực hiện:**
  1. Tại phân hệ **Security standards** trên trang kích hoạt, tiến hành tích chọn các tiêu chuẩn mong muốn:
     - **AWS Foundational Security Best Practices v1.0.0**: Bộ quy tắc thực nghiệm tối ưu cốt lõi từ chuyên gia AWS.
     - **CIS AWS Foundations Benchmark v1.4.0**: Tiêu chuẩn an toàn thông tin quốc tế cấu hình cho AWS.
     - **PCI DSS v3.2.1**: Tiêu chuẩn an toàn bắt buộc nếu tài khoản có chứa các workload xử lý dữ liệu thẻ ngân hàng.
  2. Giữ nguyên các tùy chọn mặc định tại mục **AWS Integrations** (Tích hợp chia sẻ dữ liệu tự động với **Amazon GuardDuty**, **Amazon Inspector** nếu có).
  3. Kéo xuống cuối trang và nhấn nút **Enable Security Hub** để kích hoạt.

---

### IV. Phân tích Dashboard và theo dõi điểm số (Security Score)

+ **Mục tiêu:** Đọc hiểu giao diện quản trị, xem điểm số tổng quan và bóc tách các phát hiện rủi ro (**Findings**).
+ **Các bước thực hiện:**
  1. Sau khi bấm kích hoạt, hệ thống chuyển hướng thẳng về giao diện **Summary** (Tổng quan) của **Security Hub**.
     => *Lưu ý kỹ thuật: Hệ thống cần một khoảng thời gian (có thể lên tới từ 30 phút đến 2 tiếng) để quét toàn bộ tài nguyên và cập nhật điểm số chính xác tại mục **Security score**. Trong thời gian này, các tiêu chuẩn sẽ hiển thị trạng thái **"No data"**.*
  2. Khi quá trình quét hoàn tất, kỹ sư tiến hành kiểm tra:
     - Xem điểm số `%` tổng quan (Ví dụ: **81% Passed**).
     - Vào mục **Insights** hoặc **Findings** ở menu trái để bóc tách các lỗi cấu hình cụ thể (Ví dụ: Phát hiện tài nguyên bị đánh dấu **FAILED** do chưa bật mã hóa **KMS**).

---

### V. Dọn dẹp tài nguyên Lab (Clean up)

+ **Mục tiêu:** Tắt dịch vụ đúng quy trình sau khi hoàn thành bài thực hành để tránh phát sinh chi phí không mong muốn trong môi trường thử nghiệm.
+ **Các bước thực hiện:**
  1. Tại giao diện **Security Hub Console**, kéo xuống mục **Settings** ở menu bên trái.
  2. Chuyển sang tab **General** trong phần cấu hình.
  3. Tìm và nhấn nút **Disable Security Hub** để hủy kích hoạt dịch vụ hoàn toàn.