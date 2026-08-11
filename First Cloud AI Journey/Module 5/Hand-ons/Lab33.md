# BÀI LAB 33: MÃ HÓA DỮ LIỆU Ở TRẠNG THÁI LƯU TRỮ VỚI AWS KMS

+ **Tổng quan:**
  - Trong kiến trúc điện toán đám mây, bảo mật dữ liệu ở trạng thái nghỉ (**Encryption at Rest**) là yêu cầu bắt buộc để đáp ứng các tiêu chuẩn tuân thủ doanh nghiệp.
  - Bài lab này hướng dẫn bạn cách khởi tạo khóa mã hóa tùy chỉnh CMK qua **AWS Key Management Service (KMS)**, áp dụng mã hóa cho thùng chứa **Amazon S3**, kích hoạt nhật ký kiểm toán **AWS CloudTrail** và sử dụng **Amazon Athena** để truy vấn log truy cập đã mã hóa.

---

### I. Các bước chuẩn bị (Khởi tạo quyền và tài khoản thử nghiệm)

+ **Nhiệm vụ 1: Tạo Policy và IAM Role**
  1. Truy cập vào **IAM Console**, chọn mục **Policies** -> **Create policy** để tạo quyền truy cập dữ liệu S3 và CloudTrail.
  2. Nạp cấu hình JSON cung cấp các quyền đọc ghi tài nguyên thô theo đúng chuẩn bảo mật, đặt tên policy và lưu lại.
  3. Vào mục **Roles** -> **Create role**. Chọn loại service điều khiển là **CloudTrail/S3** để tạo một **IAM Role** phục vụ cho việc ghi log nội bộ.

+ **Nhiệm vụ 2: Tạo IAM Group và Tài khoản con (User)**
  1. Vào mục **User groups** -> **Create group** để tạo nhóm quản trị dùng chung. Gán các chính sách quyền hạn vừa tạo vào group này.
  2. Chuyển sang mục **Users** -> **Create user** để tạo một tài khoản con dùng để test. Thêm user này vào group vừa tạo, thiết lập mật khẩu và lưu file `.csv` chứa Credentials đăng nhập.

---

### II. Khởi tạo khóa mã hóa tùy chỉnh trên AWS KMS

+ **Mục tiêu:** Tạo một khóa gốc đối xứng (Symmetric Master Key) do khách hàng quản lý (Customer Managed Key) để điều khiển việc mã hóa và giải mã.
+ **Các bước thực hiện:**
  1. Trên thanh tìm kiếm **AWS Management Console**, nhập từ khóa **KMS** và truy cập dịch vụ **Key Management Service**.
  2. Tại giao diện quản trị, ở menu trái chọn mục **Customer managed keys** và nhấn chọn nút **Create key**.
  3. Tiến hành cấu hình qua các bước:
     - **Bước 1 (Configure key):** Tại mục **Key type**, tích chọn **Symmetric** (Khóa đối xứng). Tại mục **Key usage**, giữ mặc định chọn **Encrypt and decrypt**. Nhấn **Next**.
     - **Bước 2 (Add labels):** Nhập tên định danh khóa tại mục **Alias**: `kms-key-encrypt-decrypt`. Nhập mô tả tại mục **Description**: `Lab33`. Thêm nhãn quản lý tại mục **Tags**: nhập Key = `owner`, Value = tên của bạn (Ví dụ: `longthg`). Nhấn **Next**.
     - **Bước 3 (Define key administrative permissions):** Tìm kiếm và tích chọn **IAM Role** hoặc **User Admin** (Ví dụ chọn `kms-key-role`) để cấp quyền quản trị, xóa, sửa khóa này. Nhấn **Next**.
     - **Bước 4 (Define key usage permissions):** Chọn các user/role được phép sử dụng khóa này để mã hóa/giải mã dữ liệu (Tích chọn `kms-key-role`). Nhấn **Next**.
     - **Bước 5 (Review):** Kiểm tra lại cấu trúc Key Policy dạng JSON hiển thị tự động và nhấn **Finish** để hoàn tất tạo khóa. Trạng thái khóa sẽ báo xanh: *"Enabled"*.

---

### III. Thiết lập mã hóa và Upload file trên Amazon S3

+ **Mục tiêu:** Tạo một S3 Bucket mới và bắt buộc áp dụng khóa KMS vừa tạo để mã hóa tự động mọi dữ liệu được tải lên.
+ **Các bước thực hiện:**
  1. Truy cập dịch vụ **Amazon S3** từ thanh tìm kiếm Console.
  2. Nhấn nút **Create bucket**, nhập tên thùng chứa duy nhất (Ví dụ: `bucket-encrypted-lab33-xxxx`).
  3. Kéo xuống phân hệ **Default encryption** (Mã hóa mặc định):
     - Tích chọn **Server-side encryption with AWS Key Management Service keys (SSE-KMS)**.
     - Tại mục **AWS KMS key**, chọn tùy chọn **Choose from your Customer managed keys (CMK)**, sau đó chọn đúng tên khóa `kms-key-encrypt-decrypt` đã tạo ở mục II.
  4. Giữ nguyên các thông số cấu hình khác và nhấn **Create bucket**.
  5. Vào Bucket vừa tạo, thực hiện nhấn **Upload**, chọn một file dữ liệu thô từ máy tính (Ví dụ: `photo.jpg`) và nhấn **Upload** để đưa file lên hệ thống.
  => **Kết quả:** Dữ liệu này lập tức được mã hóa ngầm dưới dạng phong bì thông qua Data Key của KMS.

---

### IV. Thiết lập kiểm toán với AWS CloudTrail và Amazon Athena

+ **Mục tiêu:** Tạo một kênh theo dõi ghi nhận lại toàn bộ nhật ký gọi API mã hóa của KMS, sau đó dùng Athena để cấu hình bảng và truy vấn bằng SQL trực tiếp trên S3.
+ **Các bước thực hiện:**
  1. **Cấu hình AWS CloudTrail:**
     - Tìm kiếm dịch vụ **CloudTrail** trên Console, chọn mục **Trails** -> **Create trail**.
     - Đặt tên cho Trail, cấu hình chọn lưu trữ log file đầu ra vào một **S3 Bucket** chỉ định chuyên chứa log.
     - Tại phần **Data events**, tích chọn theo dõi các hành động của dịch vụ **KMS** để ghi lại lịch sử gọi khóa. Nhấn khởi tạo trail.
  2. **Truy vấn bằng Amazon Athena:**
     - Truy cập vào dịch vụ **Amazon Athena** từ thanh tìm kiếm Console.
     - Tại giao diện **Query Editor**, thực hiện chạy lệnh SQL định nghĩa cấu trúc bảng dữ liệu (`CREATE TABLE`) trỏ trực tiếp vị trí dữ liệu nguồn tới đường dẫn S3 Bucket chứa file log của CloudTrail.
     - Chạy một câu lệnh truy vấn mẫu:
       ```sql
       SELECT * FROM tên_bảng WHERE eventname = 'Decrypt';
       ```
       => **Kết quả:** Kiểm tra chi tiết xem tài khoản nào đã gọi API giải mã dữ liệu của KMS vào thời gian nào.

---

### V. Dọn dẹp tài nguyên Lab (Clean up)

+ **Các bước thực hiện:**
  1. Truy cập vào **S3 Console**, chọn Bucket đã tạo, thực hiện **Empty bucket** để xóa sạch các object con, sau đó thực hiện lệnh **Delete bucket**.
  2. Truy cập vào **CloudTrail Console**, chọn Trail vừa tạo, nhấn chọn **Delete**.
  3. Vào **KMS Console** -> **Customer managed keys** -> Tích chọn khóa `kms-key-encrypt-decrypt` -> Chọn **Key actions** -> **Schedule key deletion**.
  4. Cấu hình thời gian chờ xóa tối thiểu (ví dụ: 7 ngày) và nhấn xác nhận để dọn sạch môi trường Lab.