# BÀI LAB 44: CẤU HÌNH IAM ROLE & TIỀN TỐ ĐIỀU KIỆN (IP & THỜI GIAN)

+ **Tổng quan:**
  - Trong các bài lab trước, chúng ta đã quen với việc khởi tạo IAM User và cấp quyền quản trị. Tuy nhiên, để đảm bảo an toàn theo nguyên tắc cấp quyền tối thiểu (Least Privilege), doanh nghiệp thường siết chặt vai trò qua cơ chế Role Condition.
  - Trong bài lab này, chúng ta tiến hành thiết lập một hệ thống quản lý định danh gồm Group, User riêng biệt cho từng phân hệ dịch vụ, và cấu hình một Role quản trị đặc thù.
  => Tiến hành tăng cường hàng rào bảo mật bằng cách thêm các điều kiện (Conditions) trong chính sách tin cậy, giới hạn chặt chẽ dải IP mạng và khung thời gian doanh nghiệp cho phép Switch Role.

---

### I. Tạo IAM Group và Attach chính sách

+ **Mục tiêu:** Tạo một nhóm quản trị tập trung sở hữu các quyền hạn được định nghĩa sẵn bởi AWS (AWS Managed Policies) phục vụ cho hạ tầng core và database.
+ **Các bước thực hiện:**
  1. Truy cập vào **IAM Console**, chọn mục **User groups** ở thanh điều hướng bên trái và nhấn **Create group**.
  2. Đặt tên cho nhóm: `ec2-rds-admin-group`.
  3. Tại mục **Attach permissions policies**, tìm kiếm và tích chọn lần lượt 3 chính sách:
     - `AmazonEC2FullAccess`
     - `AmazonRDSFullAccess`
     - `DatabaseAdministrator`
  4. Nhấn **Create group** để hoàn tất cấu hình nhóm.

---

### II. Tạo và phân quyền cho các IAM User

+ **Mục tiêu:** Khởi tạo hệ thống 4 người dùng với các mức phân quyền logic khác nhau để kiểm tra, cô lập tầm ảnh hưởng và thử nghiệm tính năng chuyển đổi vai trò.
+ **Danh sách các tài khoản cần tạo:**
  - `EC2-admin-user`: Tài khoản được gắn chính sách trực tiếp `AmazonEC2FullAccess` để chuyên trách quản trị máy chủ ảo.
  - `RDS-admin-user`: Tài khoản được gắn chính sách trực tiếp `AmazonRDSFullAccess` để chuyên trách quản trị cơ sở dữ liệu.
  - `Group-user`: Tài khoản được add trực tiếp vào nhóm `ec2-rds-admin-group` đã tạo ở bước I để thừa hưởng toàn bộ quyền quản trị EC2 và RDS.
  - `No-permission-user`: Tài khoản trống, hoàn toàn không được attach bất kỳ policy hay add vào group nào (sử dụng làm đối tượng thử nghiệm Switch Role Condition).
+ **Thao tác kiểm tra sau khi tạo:**
  1. Sử dụng trình duyệt ẩn danh hoặc một profile khác để đăng nhập lần lượt bằng thông tin Credentials của từng user.
  2. Thử truy cập vào giao diện dịch vụ **Amazon EC2** hoặc **Amazon RDS** để xác nhận:
     - Tài khoản có quyền sẽ hiển thị đầy đủ dashboard/tài nguyên.
     - Tài khoản không có quyền (`No-permission-user`) sẽ bị báo lỗi *"API Error / Unauthorized"*.

---

### III. Khởi tạo Admin IAM Role (Trust Relationship)

+ **Mục tiêu:** Tạo một vai trò trung gian có toàn quyền quản trị hạ tầng để cho phép các user không có quyền thực hiện ủy quyền tạm thời (Assume Role).
+ **Các bước thực hiện:**
  1. Tại giao diện **IAM Console**, chọn mục **Roles** và nhấn **Create role**.
  2. Tại giao diện cấu hình thực thể tin cậy (**Trusted entity type**), chọn **Custom trust policy** để tự định nghĩa tài khoản được phép assume.
  3. Thiết lập file cấu hình JSON, trong đó trường `Principal` chỉ định rõ ARN của tài khoản `No-permission-user` được phép đảm nhận vai trò này:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/No-permission-user"
           },
           "Action": "sts:AssumeRole"
         }
       ]
     }
     ```
     => *Lưu ý: Thay thế `<ACCOUNT_ID>` bằng ID tài khoản AWS gồm 12 chữ số của bạn.*
  4. Tại bước gán quyền (**Add permissions**), chọn chính sách `AdministratorAccess` (hoặc `lab44-RoleFullAccess` tùy phiên bản Lab) để gán cho Role.
  5. Đặt tên cho Role: `lab44-RoleFullAccess` và hoàn tất khởi tạo.

---

### IV. Cấu hình Switch Role và kiểm tra sơ bộ

+ **Mục tiêu:** Đăng nhập bằng tài khoản không có quyền và thực hiện chuyển đổi sang vai trò Admin nhằm lấy thông tin chứng thực tạm thời (Temporary Credentials).
+ **Các bước thực hiện:**
  1. Đăng nhập vào **AWS Console** bằng tài khoản `No-permission-user`.
  2. Nhấn vào tên User ở góc trên cùng bên phải màn hình, chọn tính năng **Switch role**.
  3. Nhập đầy đủ thông tin:
     - **Account ID**: Điền 12 số ID tài khoản AWS của bạn (hoặc Alias tương ứng).
     - **Role**: Nhập tên chính xác của role vừa tạo: `lab44-RoleFullAccess`.
     - **Display name / Color**: Tùy chỉnh tên hiển thị và màu sắc nhận diện khi ở chế độ quản trị.
  4. Nhấn **Switch Role**. 
     => Hệ thống sẽ đổi màu thanh trạng thái. 
     => Truy cập thử vào dịch vụ **Amazon RDS** để xác nhận bạn đã có toàn quyền quản trị hệ thống một cách hợp lệ thông qua Role.

---

### V. Cấu hình điều kiện giới hạn IP (IP Condition Multi-value)

+ **Mục tiêu:** Thắt chặt bảo mật bằng cách cấu hình điều kiện chỉ cho phép người dùng thực hiện Switch Role khi kết nối từ dải địa chỉ IP mạng nội bộ của doanh nghiệp.
+ **Các bước thực hiện:**
  1. Quay trở lại tài khoản Admin gốc, truy cập vào **IAM Console** -> **Roles** -> chọn Role `lab44-RoleFullAccess`.
  2. Chuyển sang tab **Trust relationships** và chọn **Edit trust policy**.
  3. Tiến hành bổ sung khối lệnh điều kiện `Condition` vào cấu hình JSON sử dụng toán tử kiểm tra dải IP:
     - Cấu hình toán tử: `"NotIpAddress"` hoặc `"IpAddress"`.
     - Khai báo khóa điều kiện toàn cầu của AWS: `"aws:SourceIp"`.
     - Điền dải IP cho phép dưới dạng CIDR (Ví dụ: `"128.169.4.20/32"` hoặc một mạng cụ thể).
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/No-permission-user"
           },
           "Action": "sts:AssumeRole",
           "Condition": {
             "IpAddress": {
               "aws:SourceIp": "128.169.4.20/32"
             }
           }
         }
       ]
     }
     ```
  4. Nhấn **Update policy**.
+ **Thực hiện kiểm tra:** Đăng nhập lại bằng `No-permission-user` trên một đường truyền mạng có IP khác với IP đã khai báo trong `Condition`, thực hiện Switch Role và xác nhận hệ thống hiển thị thông báo từ chối truy cập: *"Invalid information in one or more fields / Access Denied"*.

---

### VI. Cấu hình điều kiện giới hạn thời gian (Time-based Condition)

+ **Mục tiêu:** Giới hạn khung giờ hiệu lực của Role, chỉ cho phép nhân sự Switch Role và thực thi tác vụ quản trị trong một khoảng thời gian cố định (ví dụ: trong ca trực hoặc trước khi thời hạn bảo trì kết thúc).
+ **Các bước thực hiện:**
  1. Tại trang chi tiết của Role `lab44-RoleFullAccess`, nhấn vào **Edit trust policy** trong tab **Trust relationships**.
  2. Nhấn nút **Add a condition (optional)** trên giao diện đồ họa hoặc thêm trực tiếp vào khối mã JSON:
     - **Condition key**: Chọn khóa thời gian hệ thống `"aws:CurrentTime"`.
     - **Operator**: Chọn toán tử so sánh ngày tháng phù hợp như `"DateGreaterThan"` (Sau thời điểm) và `"DateLessThan"` (Trước thời điểm).
     - **Value**: Nhập chuỗi thời gian định dạng chuẩn ISO 8601 (Ví dụ: `"2026-07-02T08:00:00Z"`).
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/No-permission-user"
           },
           "Action": "sts:AssumeRole",
           "Condition": {
             "DateGreaterThan": {
               "aws:CurrentTime": "2026-07-02T08:00:00Z"
             },
             "DateLessThan": {
               "aws:CurrentTime": "2026-07-02T18:00:00Z"
             }
           }
         }
       ]
     }
     ```
     => *Lưu ý: Đảm bảo cấu hình cú pháp JSON không bị lỗi dấu phẩy hay đóng mở ngoặc nhọn.*
  3. Nhấn **Update policy**.
+ **Thực hiện kiểm tra:** Chờ đợi đến khi thời gian hệ thống vượt quá mốc thời gian quy định trong lệnh `"DateLessThan"`, thử thực hiện thao tác Switch Role bằng tài khoản con và xác nhận hệ thống hoàn toàn chặn quyền truy cập, báo lỗi do không thỏa mãn điều kiện thời gian thực tế.