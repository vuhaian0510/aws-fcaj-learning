# BÀI LAB 02: BẮT ĐẦU VỚI IAM VÀ KIẾN TRÚC ỦY QUYỀN BẢO MẬT (IAM ROLES)

+ **Bối cảnh thực tế:** Trong môi trường doanh nghiệp, việc sử dụng tài khoản Root hoặc chia sẻ các cặp mã khóa cố định (Access Keys) hằng ngày tiềm ẩn rủi ro an ninh rất lớn. Thao tác sai sót từ tài khoản có quyền quá rộng có thể dẫn đến việc rò rỉ dữ liệu hoặc mã hóa ransomware phá hoại hệ thống.
+ **Mục tiêu bài Lab:**
  - Hệ thống hóa cấu trúc phân quyền thông qua IAM Group và IAM User theo nguyên tắc đặc quyền tối thiểu (Least Privilege).
  - Thiết lập một cấu trúc phân quyền động, cho phép người dùng có quyền hạn thấp (`OperatorUser`) thực hiện chuyển đổi vai trò (**Switch Role**) để nhận quyền quản trị tạm thời.
  => Triển khai hàng rào bảo mật nâng cao bao gồm xác thực đa lớp (MFA) và luân chuyển thông tin chứng thực tự động.

---

### I. Tạo IAM Group và thiết lập chính sách quản trị (Admin Group)

+ **Mục tiêu:** Tạo dựng một nhóm quản lý tập trung để tự động áp dụng các tập chính sách quyền hạn đồng nhất xuống mọi nhân sự thuộc nhóm.
+ **Các bước thực hiện:**
  1. Đăng nhập vào **AWS Management Console** bằng tài khoản Root hoặc Admin hiện tại.
  2. Tìm kiếm và truy cập vào dịch vụ **IAM (Identity and Access Management)**.
  3. Tại thanh điều hướng bên trái, chọn **User groups** và nhấn nút **Create group**.
  4. Đặt tên cho nhóm tại mục **User group name**: `Admin-Group`.
  5. Tại phân hệ **Attach permissions policies**, tìm kiếm chính sách quản trị hệ thống sẵn có của AWS: `AdministratorAccess` và tích chọn.
  6. Kéo xuống cuối trang và nhấn **Create group**.

---

### II. Khởi tạo IAM User và gán vào nhóm quản trị

+ **Mục tiêu:** Tạo tài khoản định danh cá nhân với cơ chế mật khẩu mạnh và gán vào nhóm để thừa hưởng quyền hạn an toàn.
+ **Các bước thực hiện:**
  1. Tại giao diện điều hướng IAM Console, chọn mục **Users** và nhấn **Create user**.
  2. Đặt tên cho người dùng quản trị: `Admin-User`.
  3. Tích chọn mục **Provide user access to the AWS Management Console** để cấp quyền đăng nhập giao diện web cho user.
  4. Cấu hình xác thực:
     - Chọn **I want to create an IAM user** (Tạo người dùng thông thường).
     - Tại mục **Console password**, chọn **Custom password** và nhập mật khẩu mạnh đáp ứng tiêu chuẩn doanh nghiệp.
     - Tích chọn **Users must create a new password at next sign-in** để bắt buộc nhân sự phải đổi mật khẩu riêng trong lần đầu đăng nhập.
  5. Nhấn **Next** chuyển sang bước **Set permissions**.
  6. Tại đây, chọn mục **Add user to group** và tích chọn vào nhóm `Admin-Group` đã khởi tạo ở bước I.
  7. Nhấn **Next**, xem lại cấu trúc thiết lập và nhấn **Create user** để hoàn tất.
  => *Lưu ý bảo mật: Tải file chứa đường dẫn đăng nhập dạng Alias/Account ID và thông tin mật khẩu ban đầu về máy bảo mật.*

---

### III. Triển khai bảo mật nâng cao cho hệ thống định danh

+ **Mục tiêu:** Kích hoạt cơ chế xác thực đa lớp (MFA) để ngăn chặn rủi ro bị chiếm quyền kiểm soát khi lộ mật khẩu thô.
+ **Các bước thực hiện:**
  1. Đăng xuất tài khoản hiện tại và dùng đường dẫn URL vừa lưu để đăng nhập bằng tài khoản `Admin-User`. Hệ thống sẽ yêu cầu đổi mật khẩu mới ngay lập tức.
  2. Sau khi đăng nhập thành công, tại giao diện IAM Dashboard, nhấn vào mục **Add MFA** (hoặc vào phần **Security credentials** của chính user).
  3. Nhấn **Assign MFA device**.
  4. Đặt tên thiết bị định danh và chọn **Authenticator app** (Sử dụng ứng dụng xác thực ảo trên thiết bị di động như Google Authenticator hoặc Microsoft Authenticator).
  5. Nhấn **Next** để hiển thị mã QR Code trên màn hình.
  6. Dùng ứng dụng trên điện thoại quét mã QR, sau đó nhập liên tiếp 2 mã số xác thực (MFA code 1 và MFA code 2) thay đổi theo thời gian thực vào giao diện AWS.
  7. Nhấn **Add MFA** để hoàn thành hàng rào bảo mật.

---

### IV. Khởi tạo IAM Role và tài khoản vận hành thấp (Operator)

+ **Mục tiêu:** Tạo một tài khoản không có quyền hạn hệ thống (`OperatorUser`) và một vai trò trung gian (`IAM Role Admin`) làm tiền đề cho quy trình ủy quyền động an toàn.
+ **Các bước thực hiện:**
  1. **Tạo Operator User**:
     - Lặp lại các thao tác tại mục II để tạo một user mới tên là `OperatorUser`.
     - Tại bước gán quyền (**Set permissions**), **KHÔNG** gán user này vào bất kỳ group nào và không attach bất kỳ policy nào.
     - Tiến hành nhấn hoàn tất khởi tạo user trống.
  2. **Tạo IAM Role Admin**:
     - Vào mục **Roles** trên IAM Console và nhấn **Create role**.
     - Tại bước chọn thực thể tin cậy (**Trusted entity type**), chọn **Custom trust policy** để tự khai báo quyền hạn truy cập.
     - Trong khối mã JSON của chính sách tin cậy (*Trust Policy*), tại trường dữ liệu `"Principal"`, cấu hình trỏ chính xác đến mã định danh ARN của tài khoản `OperatorUser` vừa tạo:
       ```json
       {
         "Version": "2012-10-17",
         "Statement": [
           {
             "Effect": "Allow",
             "Principal": {
               "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/OperatorUser"
             },
             "Action": "sts:AssumeRole"
           }
         ]
       }
       ```
       => *Lưu ý: Thay thế `<ACCOUNT_ID>` bằng ID tài khoản AWS gồm 12 chữ số của bạn.*
     - Nhấn **Next**, tại trang gán policy quyền hạn thực thi, tìm kiếm và chọn chính sách `AdministratorAccess`.
     - Nhấn **Next**, đặt tên cho vai trò này là `lab02-Admin-Role` và hoàn tất khởi tạo.

---

### V. Thực thi chuyển đổi vai trò và kiểm tra quyền hạn tạm thời (Assume Role)

+ **Mục tiêu:** Đăng nhập dưới vai trò user có quyền hạn thấp, thực hiện thao tác đổi vai trò (**Switch Role**) để nhận quyền quản trị cao cấp tạm thời từ AWS STS và tự động hết hạn, đảm bảo nguyên tắc an toàn thông tin.
+ **Các bước thực hiện:**
  1. Đăng xuất và tiến hành đăng nhập lại vào AWS Console bằng tài khoản `OperatorUser`.
  2. **Kiểm tra quyền gốc**: Thử truy cập vào dịch vụ máy chủ ảo Amazon EC2 hoặc Amazon S3, hệ thống sẽ báo lỗi đỏ toàn diện do tài khoản này không có bất kỳ quyền hạn nào.
  3. Nhấn vào tên hiển thị của `OperatorUser` ở góc trên cùng bên phải thanh menu điều hướng của AWS Console, chọn tính năng **Switch role**.
  4. Nhập đầy đủ và chính xác các thông tin yêu cầu tại form chuyển đổi:
     - **Account ID**: Nhập mã số gồm 12 số định danh tài khoản AWS của bạn.
     - **Role**: Nhập tên chính xác của role quản trị trung gian đã tạo ở mục IV: `lab02-Admin-Role`.
     - **Display name / Color**: Tùy chỉnh tên hiển thị quản trị và lựa chọn màu sắc nhận diện hệ thống (Ví dụ: chọn màu Đỏ).
  5. Nhấn **Switch Role**.
  => **Xác nhận kết quả**: Thanh menu hệ thống lập tức chuyển sang màu sắc bạn đã chọn. AWS Security Token Service (STS) đã xóa bỏ quyền hạn cũ và thay thế hoàn toàn bằng quyền `AdministratorAccess` tạm thời của Role. Thử truy cập lại dịch vụ Amazon EC2 hoặc Amazon S3, lúc này bạn đã có toàn quyền thao tác vận hành tài nguyên hệ thống một cách hợp lệ.