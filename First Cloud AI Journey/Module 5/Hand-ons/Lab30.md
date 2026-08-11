# BÀI LAB 30: GIỚI HẠN QUYỀN CỦA USER VỚI IAM PERMISSION BOUNDARY

+ **Tổng quan:**
  - Sau khi đã hiểu về các thành phần IAM User, Group, và Role, chúng ta tiếp tục khám phá một tính năng nâng cao để quản trị an toàn doanh nghiệp: **IAM Permission Boundary**.
  - **IAM Permission Boundary (Rào dải giới hạn quyền):** Là một tính năng quản trị tối cao cho phép người quản trị thiết lập ngưỡng quyền hạn tối đa (*Maximum Permissions*) mà một IAM User hoặc IAM Role có thể đạt được.
  - **Quyền hạn có hiệu lực (Effective Permissions):** Quyền thực tế mà user có thể thực thi chính là giao điểm giao thoa (phần chung) giữa **Identity-based Policy** (Chính sách dựa trên định danh) và **Permission Boundary** (Rào dải giới hạn quyền).
+ **Bối cảnh & Tại sao cần sử dụng:**
  - Khi số lượng nhân sự và tài nguyên doanh nghiệp tăng lên liên tục, việc chỉnh sửa hoặc tạo mới hàng loạt policy rất dễ dẫn đến lỗi cấu hình, tạo ra lỗ hổng cho phép người dùng trái phép tự nâng cao đặc quyền (Privilege Escalation).
  - Áp dụng Permission Boundary giúp thiết lập một khung bảo mật cố định, chặn đứng nguy cơ leo thang đặc quyền mà không cần thay đổi các chính sách hiện tại.
  => *Ví dụ thực tế:* Cho dù bạn gắn một chính sách quyền có phạm vi rất rộng như `AdministratorAccess` cho một user, nhưng nếu Permission Boundary của user đó chỉ cho phép dịch vụ EC2, thì user đó tuyệt đối không thể thực hiện các hành động trên S3, RDS hay bất kỳ dịch vụ nào khác.

---

### I. Tạo chính sách giới hạn địa lý (Restriction Policy)

+ **Mục tiêu:** Khởi tạo một **IAM Policy** đóng vai trò làm khung giới hạn, chỉ cho phép người dùng thực thi toàn quyền thao tác trên dịch vụ EC2 tại một Region duy nhất được chỉ định (Ví dụ: `ap-southeast-1` - Singapore).
+ **Các bước thực hiện:**
  1. Đăng nhập vào **AWS Management Console**, tìm kiếm và truy cập dịch vụ **IAM**.
  2. Tại thanh điều hướng bên trái, chọn mục **Policies** và nhấn nút **Create policy**.
  3. Trên giao diện cấu hình, chuyển sang tab **JSON**.
  4. Xóa bỏ đoạn mã mặc định và copy đoạn mã cấu hình điều kiện khu vực địa lý sau:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "EC2RestrictRegion",
                 "Effect": "Allow",
                 "Action": "ec2:*",
                 "Resource": "*",
                 "Condition": {
                     "StringEquals": {
                         "aws:RequestedRegion": "ap-southeast-1"
                     }
                 }
             }
         ]
     }
     ```
  5. Nhấn **Next**. Tại trang **Review and create**, đặt tên cho chính sách là `ec2-admin-restrict-region` và nhấn **Create policy** để hoàn tất.

---

### II. Khởi tạo IAM User và áp dụng Permission Boundary

+ **Mục tiêu:** Tạo một tài khoản người dùng con, gán chính sách quyền hạn thông thường nhưng áp thêm rào dải giới hạn vừa tạo để kiểm soát quyền tối đa.
+ **Các bước thực hiện:**
  1. Tại **IAM Console**, chọn mục **Users** ở menu trái và nhấn **Create user**.
  2. Đặt tên cho người dùng: `Limited-EC2-User`, bật quyền truy cập **AWS Management Console** và thiết lập mật khẩu đăng nhập an toàn.
  3. Tại bước **Set permissions**, chọn mục **Attach policies directly** và gán chính sách toàn quyền EC2 thông thường: `AmazonEC2FullAccess`.
  4. Kéo xuống phân hệ mở rộng **Set permissions boundary** (Rào dải kiểm soát quyền):
     - Tích chọn ô **Use a permissions boundary to control the maximum user permissions**.
     - Tìm kiếm và chọn chính sách tùy chỉnh `ec2-admin-restrict-region` đã tạo ở mục I làm khung giới hạn.
  5. Nhấn **Next**, xem lại cấu trúc phân quyền và nhấn **Create user** để hoàn tất.

---

### III. Thử nghiệm và đánh giá quyền hạn có hiệu lực

+ **Mục tiêu:** Đăng nhập bằng tài khoản giới hạn để xác thực cơ chế hoạt động của **Permission Boundary** trên các khu vực địa lý khác nhau.
+ **Các bước thực hiện:**
  1. Sử dụng trình duyệt ẩn danh, đăng nhập vào **AWS Console** bằng thông tin tài khoản của `Limited-EC2-User`.
  2. Trên thanh menu trên cùng, chuyển vùng quốc gia sang **Singapore (ap-southeast-1)**. Truy cập dịch vụ **EC2**, tiến hành khởi chạy thử một instance và xác nhận hệ thống hoạt động bình thường (do thỏa mãn cả policy cho phép lẫn khung permission boundary).
  3. Tiếp tục đổi vùng quốc gia sang một Region khác (Ví dụ: **N. Virginia - us-east-1** hoặc **Tokyo - ap-northeast-1**).
  4. Vào lại giao diện **EC2**, bạn sẽ lập tức thấy hệ thống hiển thị hàng loạt thông báo lỗi đỏ: *"API Error / Unauthorized"*. Thử tạo máy chủ sẽ bị chặn hoàn toàn.
  => **Kết luận:** Mặc dù User sở hữu chính sách `AmazonEC2FullAccess` (cho phép tạo EC2 trên toàn thế giới), nhưng rào dải **Permission Boundary** đã siết chặt quyền tối đa chỉ ở Singapore, khiến mọi hành động ở các Region khác bị vô hiệu hóa hoàn toàn.

---

### IV. Dọn dẹp tài nguyên Lab (Clean up)

+ **Các bước thực hiện:**
  1. Đăng xuất tài khoản giới hạn, đăng nhập lại bằng tài khoản Admin gốc.
  2. Vào **IAM Console** -> **Users** -> Chọn và tiến hành xóa bỏ tài khoản `Limited-EC2-User`.
  3. Vào mục **Policies** -> Tìm kiếm chính sách `ec2-admin-restrict-region` -> Chọn **Actions** và nhấn **Delete** để dọn sạch môi trường Lab.