# BÀI LAB 48: CẤP QUYỀN CHO ỨNG DỤNG TRUY CẬP DỊCH VỤ AWS VỚI IAM ROLE

+ **Tổng quan:**
  - Trong workshop này, chúng ta sẽ học cách cấp quyền truy cập cho ứng dụng chạy trên máy chủ có thể truy cập tới các dịch vụ khác của AWS.
  - Phân tích và so sánh:
    * Phương pháp cấp quyền qua cặp mã khóa Access Key (tại sao không nên dùng).
    * Phương pháp ủy quyền an toàn thông qua IAM Role (Instance Profile) trên EC2.

---

### I. Chuẩn bị tài nguyên hạ tầng nền tảng

+ **Mục tiêu:** Khởi tạo một máy chủ ảo EC2 và một thùng chứa S3 độc lập làm môi trường giả lập ứng dụng truyền tải dữ liệu.
+ **Các bước thực hiện:**
  1. **Tạo EC2 Instance**: Truy cập **EC2 Console**, tiến hành khởi chạy một máy chủ Linux cơ bản, cấu hình Network/Subnet phù hợp và giữ máy chủ ở trạng thái hoạt động (Running).
  2. **Tạo S3 Bucket**: Truy cập **S3 Console**, nhấn **Create bucket**, đặt tên thùng chứa duy nhất trên toàn cầu (Global Unique Name) và tiến hành khởi tạo mà không cần cấu hình thêm các tùy chọn nâng cao.

---

### II. Phương pháp 1: Sử dụng cặp mã khóa cố định (Access Key)

+ **Mục tiêu:** Tạo một IAM User, sinh mã Access Key/Secret Key để cấu hình thủ công vào máy chủ/mã nguồn ứng dụng và thực hiện truyền tải dữ liệu lên S3 nhằm nhận diện rủi ro bảo mật thực tế.
+ **Các bước thực hiện:**
  1. **Tạo IAM User**: Truy cập **IAM Console** -> **Users** -> **Create user**. Đặt tên cho user và tiến hành gán (attach) trực tiếp chính sách quyền hạn `AmazonS3FullAccess`.
  2. **Sinh mã Access Key**: 
     - Vào tab **Security credentials** của user vừa tạo, tại mục **Access keys** nhấn **Create access key**.
     - Chọn use case phù hợp (ví dụ: CLI hoặc Application) và tải tệp tin định dạng `.csv` chứa mã Access Key và Secret Access Key về máy tính cá nhân.
  3. **Cấu hình và Kiểm tra**:
     - Kết nối vào máy chủ EC2 qua Session Manager hoặc SSH.
     - Thực hiện cấu hình credentials cố định bằng lệnh:
       ```bash
       aws configure
       # Nhập Access Key và Secret Access Key khi được yêu cầu
       ```
     - Chạy lệnh upload thử một tệp tin dữ liệu lên S3 Bucket đã chuẩn bị ở bước I:
       ```bash
       aws s3 cp test.txt s3://<YOUR_BUCKET_NAME>/
       ```
+ **Hệ quả & Rủi ro kỹ thuật:**
  - Kết quả: Ứng dụng chạy thành công nhờ quyền quản trị dài hạn được cấp cho IAM User.
  => **RỦI RO LỚN**: Thông tin cặp mã khóa này bị lưu vết cố định (hardcode) trên máy chủ hoặc trong code. Nếu vô tình đồng bộ mã nguồn ứng dụng lên các kho chứa công khai như GitHub, hacker có thể dùng bot tự động quét sạch mã khóa này để chiếm quyền kiểm soát, phá hoại tài nguyên hoặc đào tiền ảo, gây ra gánh nặng tài chính khổng lồ cho doanh nghiệp.

---

### III. Phương pháp 2: Ủy quyền bảo mật thông qua IAM Role trên EC2

+ **Mục tiêu:** Thay thế hoàn toàn phương pháp sử dụng Access Key nguy hiểm bằng cách dùng cơ chế sinh token bảo mật tạm thời thông qua việc gán trực tiếp một IAM Role vào cấu hình của EC2 Instance (Instance Profile).
+ **Các bước thực hiện:**
  1. **Tạo IAM Role**: Truy cập **IAM Console** -> **Roles** -> **Create role**. Chọn loại thực thể tin cậy (**Trusted entity type**) là **AWS service** và dịch vụ sử dụng (Use case) là **EC2**.
  2. **Gán Policy cho Role**: Tại bước gán quyền (**Add permissions**), tìm kiếm và tích chọn chính sách `AmazonS3FullAccess` (hoặc cấu hình quyền tối thiểu tùy theo yêu cầu thực tế) và đặt tên cho Role này (ví dụ: `EC2-S3-Access-Role`).
  3. **Gán Role (Instance Profile) vào EC2**:
     - Quay trở lại giao diện **EC2 Console**, tích chọn máy chủ ảo đã tạo ở bước I.
     - Nhấn vào mục **Actions** -> **Security** -> **Modify IAM role**.
     - Chọn tên Role vừa khởi tạo và nhấn **Update IAM role** để gán trực tiếp vai trò vào máy chủ.
  4. **Thực hiện truyền dữ liệu an toàn**:
     - Kết nối lại vào terminal của máy chủ EC2.
     - Tiến hành xóa bỏ hoàn toàn các file cấu hình Access Key cũ bằng lệnh:
       ```bash
       rm -rf ~/.aws/credentials
       ```
     - Chạy lại lệnh upload tệp tin lên S3 Bucket và xác nhận hệ thống vẫn truyền tải dữ liệu thành công hoàn toàn tự động:
       ```bash
       aws s3 cp test.txt s3://<YOUR_BUCKET_NAME>/
       ```
+ **Lợi ích kỹ thuật:**
  - Mã nguồn ứng dụng hoàn toàn sạch, không chứa bất kỳ thông tin Credentials cố định nào.
  => **Bảo mật tuyệt đối**: Dịch vụ **AWS Security Token Service (STS)** và **AWS SDK** sẽ tự động chịu trách nhiệm sinh ra, rotate (luân chuyển) và nạp các mã khóa bảo mật tạm thời (Temporary Credentials) trực tiếp vào RAM máy chủ để ứng dụng gọi API, đảm bảo an toàn tuyệt đối kể cả khi mã nguồn bị rò rỉ ra ngoài.