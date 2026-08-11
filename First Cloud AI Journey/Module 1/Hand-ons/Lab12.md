# Bài Lab 12: Thiết lập và quản lý định danh tập trung với AWS IAM Identity Center

> [!NOTE]
> **Bối cảnh thực tế:** Khi doanh nghiệp mở rộng quy mô sử dụng Cloud, việc quản lý hàng trăm tài khoản người dùng và mật khẩu riêng lẻ trên nhiều tài khoản AWS khác nhau (Multi-accounts) trở nên cực kỳ phức tạp và dễ dẫn đến rủi ro lộ lọt an ninh.
>
> **Mục tiêu bài Lab:** Thực hành cấu hình khởi tạo dịch vụ **AWS IAM Identity Center** (tên gọi cũ là *AWS Single Sign-On*) kết hợp với **AWS Organizations** nhằm thiết lập hệ thống quản lý định danh tập trung, phân chia quyền hạn truy cập các tài khoản con dựa trên nhóm bộ phận (Group) và vai trò dự án (Roles). Triển khai cơ chế kiểm soát quyền hạn tối thiểu (Least Privilege) có điều kiện về thời gian (Time-based access control) phục vụ cho chiến lược an toàn **Zero Trust**.

---

## I. Các bước chuẩn bị và kích hoạt hệ thống (Preparation)

**Mục tiêu:** Kích hoạt môi trường quản trị tổ chức đa tài khoản và bật tính năng định danh tập trung Identity Center từ tài khoản Master (Management Account).

### Các bước thực hiện:
1. Đăng nhập vào **AWS Management Console** bằng tài khoản Admin gốc (tài khoản chưa tham gia bất kỳ tổ chức nào).
2. **Kích hoạt AWS Organizations:** Tìm kiếm dịch vụ **AWS Organizations**, nhấn nút **Create an organization** để tự động thiết lập cấu trúc cây thư mục quản trị đa tài khoản. Tài khoản hiện tại sẽ đóng vai trò là *Master account* (Management Account).
3. **Kích hoạt IAM Identity Center:** Tìm kiếm và truy cập dịch vụ **IAM Identity Center**.
4. Tại giao diện chính, nhấn nút màu cam **Enable** (Kích hoạt). Hệ thống sẽ tự động liên kết với AWS Organizations và khởi tạo một thư mục lưu trữ định danh mặc định (Identity Store).
5. Ghi lại đường dẫn đăng nhập tập trung **AWS access portal URL** được hệ thống cấp phát (Ví dụ: `https://d-1234567890.awsapps.com/start`).

---

## II. Quản lý định danh: Khởi tạo User và Group

**Mục tiêu:** Tạo dựng cấu trúc phòng ban (Group) và tài khoản nhân sự (User) trực tiếp bên trong kho lưu trữ của Identity Center.

### Các bước thực hiện:
1. Tại menu bên trái của Identity Center, chọn mục **Groups** $\rightarrow$ nhấn **Create group**. Đặt tên nhóm là `Cloud-Admin-Group` và nhấn tạo. Tiếp tục tạo một nhóm thứ hai tên là `Operator-Group`.
2. Chuyển sang mục **Users** $\rightarrow$ nhấn **Add user**.
3. Điền thông tin chi tiết cho nhân sự:
   - **Username:** `admin-john`
   - **Email address:** Nhập email thực tế của bạn để nhận mã kích hoạt mật khẩu.
   - **First name / Last name:** Nhập tên nhân sự.
4. Nhấn **Next**, tại trang gán nhóm, tích chọn đưa user `admin-john` vào nhóm `Cloud-Admin-Group`. Nhấn **Next** và nhấn **Add user** để hoàn tất.
5. Tạo thêm một user thứ hai là `operator-alex` và gán vào nhóm `Operator-Group`.
6. Kiểm tra hộp thư email đã khai báo, click vào link xác nhận của AWS gửi về để tiến hành đặt mật khẩu mới cho các User vừa tạo.

---

## III. Thiết lập bộ quyền và ánh xạ truy cập (Customer Managed Policies)

**Mục tiêu:** Khởi tạo các tập hợp quyền hạn (Permission Sets) từ các chính sách quản trị sẵn có hoặc tự biên soạn, sau đó gán Nhóm người dùng vào các AWS Account thành viên chỉ định.

### Các bước thực hiện:
1. Tại menu bên trái Identity Center, chọn mục **Permission sets** $\rightarrow$ nhấn **Create permission set**.
2. **Tạo bộ quyền Administrator Access:**
   - Chọn **Predefined permission set**, nhấn **Next**.
   - Tìm kiếm và tích chọn chính sách quản trị cao nhất **AdministratorAccess**.
   - Đặt tên bộ quyền là `Admin-PermSet` và hoàn tất tạo.
3. **Tạo bộ quyền Custom:**
   - Chọn **Custom permission set** để tự biên soạn quyền hạn tối thiểu.
   - Tiến hành nạp đoạn mã JSON chỉ cho phép đọc ghi dịch vụ EC2, đặt tên là `EC2-Operator-PermSet`.
4. Điều hướng tới mục **AWS accounts** ở menu trái. Giao diện sẽ hiển thị cây thư mục cấu trúc các tài khoản con thuộc AWS Organizations.
5. Tích chọn vào tài khoản con muốn phân quyền (Ví dụ: `Development-Account`), sau đó nhấn nút **Assign users or groups**.
6. Thực hiện ánh xạ:
   - Tab **Groups:** Tích chọn nhóm `Cloud-Admin-Group`. Nhấn **Next**.
   - Tab **Permission sets:** Tích chọn bộ quyền `Admin-PermSet`. Nhấn **Next**.
   - Xem lại cấu trúc và nhấn **Submit**. 
   
   > [!NOTE]
   > AWS sẽ tự động triển khai (deploy) một IAM Role ngầm xuống tài khoản con để sẵn sàng phục vụ.
7. Lặp lại thao tác tương tự để ánh xạ nhóm `Operator-Group` vào bộ quyền `EC2-Operator-PermSet` tại tài khoản thành viên chỉ định.

---

## IV. Nâng cao: Sử dụng Time-based Access Control (Điều kiện thời gian)

**Mục tiêu:** Thực thi chiến lược Zero Trust bằng cách giới hạn khung giờ hiệu lực của bộ quyền, chỉ cho phép nhân sự sử dụng quyền hạn trong ca làm việc quy định.

### Các bước thực hiện:
1. Vào mục **Permission sets**, click chọn bộ quyền `EC2-Operator-PermSet`.
2. Tại mục cấu hình chính sách, tiến hành chỉnh sửa hoặc bổ sung khối lệnh điều kiện hạn chế thời gian (`Condition`) vào đoạn mã cấu hình JSON.
3. Khai báo khóa điều kiện toàn cầu về thời gian của AWS: `"aws:CurrentTime"`.
4. Sử dụng các toán tử so sánh thời gian như `"DateGreaterThan"` và `"DateLessThan"` để khoanh vùng khung giờ (Ví dụ: chỉ cho phép thao tác từ 08:00 AM đến 17:00 PM UTC). Nhấn lưu thay đổi để cập nhật bộ quyền.

---

## V. Truy cập hệ thống và kiểm tra quản trị (AWS CLI & Portal)

**Mục tiêu:** Đăng nhập một lần qua User Portal để truy cập đa tài khoản trên cả giao diện Web và công cụ dòng lệnh AWS CLI.

### Các bước thực hiện:
1. Mở một trình duyệt mới, truy cập vào đường dẫn **AWS access portal URL** đã lưu ở mục I.
2. Đăng nhập bằng tài khoản `admin-john` và mật khẩu đã thiết lập. Sau khi vượt qua hàng rào xác thực, giao diện cổng thông tin portal sẽ hiển thị danh sách các AWS Account mà User này được phép tiếp cận.
3. Click vào tên tài khoản `Development-Account`, bạn sẽ thấy tùy chọn gán liền với bộ quyền `Admin-PermSet`.
4. Nhấp vào dòng **Management console** để mở giao diện Web của tài khoản con với toàn quyền quản trị mà không cần nhập lại password.
5. **Cấu hình AWS CLI:** Tại dòng bộ quyền trên portal, nhấn vào mục **Command line or programmatic access**. Copy đoạn mã cấu hình biến môi trường tạm thời (gồm `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, và `AWS_SESSION_TOKEN`). Mở terminal máy tính, dán đoạn mã này vào để thực hiện các câu lệnh kiểm tra tài nguyên một cách an toàn thông qua CLI.

---

## VI. Dọn dẹp tài nguyên (Clean up)

Để tránh phát sinh chi phí không mong muốn, hãy thực hiện dọn dẹp theo các bước sau:

1. Đăng xuất khỏi cổng thông tin Portal, đăng nhập lại bằng tài khoản Admin tối cao ban đầu.
2. Truy cập vào **IAM Identity Center Console** $\rightarrow$ mục **AWS accounts** $\rightarrow$ Chọn tài khoản con và tiến hành **Remove** (Xóa bỏ) các liên kết ánh xạ giữa Groups và Permission Sets.
3. Vào mục **Permission sets**, chọn các bộ quyền `Admin-PermSet`, `EC2-Operator-PermSet` và thực hiện lệnh **Delete**.
4. Vào mục **Users** và **Groups**, tích chọn các tài khoản `admin-john`, `operator-alex` cùng các nhóm tương ứng để thực hiện xóa sạch dữ liệu khỏi Identity Store.
5. Nếu không còn nhu cầu quản lý đa tài khoản, truy cập dịch vụ **AWS Organizations** để thực hiện gỡ bỏ hoàn toàn các tài khoản thành viên con, tránh phát sinh các chi phí lưu trữ dữ liệu log ngầm của AWS CloudTrail.