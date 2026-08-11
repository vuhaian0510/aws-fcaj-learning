# Bài Lab 24: Triển khai giải pháp Hybrid Storage với AWS File Storage Gateway

> [!NOTE]
> **Bối cảnh thực tế:** AWS Storage Gateway là giải pháp lưu trữ hỗn hợp (Hybrid Cloud) cho phép mở rộng dung lượng lưu trữ tại chỗ (on-premises) lên đám mây thông qua các giao thức chia sẻ file tiêu chuẩn như SMB hoặc NFS.
> 
> **Mục tiêu bài Lab:** Thực hành triển khai File Gateway bằng cách dùng máy chủ EC2 giả lập thiết bị appliance cục bộ, thiết lập kênh kết nối tới Amazon S3 bucket và thực hiện mount ổ đĩa mạng trực tiếp từ máy client.

---

## I. Chuẩn bị tài nguyên nền tảng (S3 Bucket & Gateway Appliance)

**Mục tiêu:** Tạo S3 Bucket làm nơi lưu trữ gốc và khởi chạy một EC2 instance cấu hình sẵn AMI của AWS Storage Gateway để giả lập thiết bị lưu trữ tại chỗ.

### Các bước thực hiện:
1. **Tạo Amazon S3 Bucket:**
   - Truy cập vào **S3 Console**.
   - Nhấn **Create bucket** để tạo một thùng chứa mới (ví dụ: `filegw-bucket-lab24`) làm phân hệ lưu trữ đối tượng gốc cho các tệp tin chia sẻ.
2. **Khởi tạo EC2 Storage Gateway Appliance:**
   - Khởi chạy một **EC2 Instance** sử dụng bản ảnh đĩa (AMI) chuyên dụng do AWS Storage Gateway cung cấp nhằm giả lập một thiết bị lưu trữ vật lý đặt tại trung tâm dữ liệu cục bộ (local).
   - Lấy thông tin và ghi lại hai địa chỉ của máy chủ Gateway này:
     - **Private IP** (Ví dụ: `172.31.24.55`)
     - **Public IP** (Ví dụ: `18.142.250.220`)
3. **Cấu hình tường lửa (Security Group Rules):**
   - Vào phân hệ **Security Group** gắn liền với máy chủ EC2 Gateway vừa tạo, tiến hành chỉnh sửa cấu hình **Inbound rules** (Quy tắc chiều đi vào).
   - Bắt buộc mở các cổng dịch vụ kết nối mạng tiêu chuẩn để phục vụ File Share bao gồm:
     - **Cổng 445**: Giao thức SMB
     - **Cổng 111**: Giao thức NFS/RPC
     - **Cổng 2049**: Giao thức NFS
     - **Cổng 20048**: Custom UDP (phụ trợ cho NFS)

---

## II. Thiết lập AWS Storage Gateway và khởi tạo File Share

**Mục tiêu:** Đăng ký kích hoạt Gateway với AWS Cloud và khởi tạo phân hệ File Share sử dụng giao thức SMB trỏ tới S3 bucket.

### Các bước thực hiện:
1. **Khởi tạo Gateway trên Console:**
   - Truy cập vào giao diện quản trị dịch vụ **Storage Gateway**.
   - Chọn lệnh khởi tạo và nhập địa chỉ IP mạng để hệ thống tiến hành kích hoạt và nhận diện thiết bị ảo hóa `lab24-filesgw`.
2. **Khởi tạo phân hệ File Share:**
   - Tại menu điều hướng phía bên trái của Storage Gateway, chọn mục **File shares** và nhấn nút **Create file share**.
   - Tại mục cấu hình **Amazon S3 bucket**, nhập chính xác tên hoặc ARN của S3 Bucket đã chuẩn bị ở bước I.
   - Tại mục **Gateway**, chọn đúng tên Gateway appliance đang hoạt động (`lab24-filesgw`).
   - Tại mục chọn giao thức chia sẻ, tích chọn chuẩn cấu hình **SMB (Server Message Block)** dành cho môi trường máy chủ Windows hoặc **NFS** dành cho môi trường Linux.
   - Hoàn tất khởi tạo, hệ thống sẽ hiển thị trạng thái **Available** và cung cấp sẵn mã lệnh kết nối mẫu (**Example Commands**).

---

## III. Thực thi Mount ổ đĩa mạng tại máy Client (On-premises Computer)

**Mục tiêu:** Ánh xạ đĩa mạng (Mount) từ máy khách tới File Share trên Storage Gateway và kiểm nghiệm khả năng tự động đồng bộ hóa đối tượng lên Amazon S3.

### Các bước thực hiện:
1. Sử dụng **Command Prompt (CMD)** hoặc **Terminal** trên máy khách để thực thi câu lệnh kết nối theo mẫu hệ thống gợi ý.

#### Cú pháp lệnh trên hệ điều hành Windows:
```cmd
net use [DriveLetter]: \\[IP_Address]\[FileShare_Name] /user:[Gateway_ID]\smbguest
```

> [!IMPORTANT]  
> **Một số lưu ý bắt buộc khi map ổ đĩa mạng cục bộ:**
> 
> * **Lưu ý 1 (Tên ổ đĩa):** Lựa chọn ký tự tên ổ đĩa mạng đại diện (Ví dụ: `Z:`) sao cho ký tự này chưa tồn tại và chưa bị trùng lặp trong danh sách ổ đĩa cục bộ của máy tính cá nhân.
> * **Lưu ý 2 (Địa chỉ mạng):** Thay thế dải địa chỉ IP mặc định trong câu lệnh bằng địa chỉ Public IP (hoặc Private IP tùy cấu hình định tuyến mạng) của máy chủ EC2 Storage Gateway Appliance.
> * **Lưu ý 3 (Tài khoản và Mật khẩu):** Xác thực quyền đăng nhập chính xác bằng tài khoản người dùng hệ thống `smbguest` và nạp password bảo mật đã thiết lập tại mục cấu hình *User Access* của File Share trước đó.

#### Kiểm tra và đánh giá đồng bộ dữ liệu thực tế:
1. Sau khi dòng lệnh CMD báo kết nối thành công, truy cập vào **File Explorer** trên máy tính, bạn sẽ thấy ổ đĩa mạng mang ký tự `Z:` hiển thị như một phân vùng ổ cứng thông thường.
2. Tiến hành tạo mới một tệp tin văn bản bất kỳ bên trong ổ đĩa này (Ví dụ: `haha.txt`).
3. Quay trở lại bảng điều khiển dịch vụ **Amazon S3** trên AWS Console, truy cập vào thùng chứa mục tiêu và xác nhận tệp tin văn bản `haha.txt` đã tự động được hệ thống Storage Gateway dịch chuyển, đồng bộ hóa thành một Object tĩnh an toàn trên đám mây.

---

## IV. Dọn dẹp tài nguyên Lab (Clean up)

Để tránh phát sinh chi phí không mong muốn, hãy thực hiện dọn dẹp theo các bước sau:

1. Thực hiện ngắt kết nối và gỡ bỏ hoàn toàn ánh xạ ổ đĩa mạng `Z:` tại máy tính cá nhân.
2. Truy cập giao diện **Storage Gateway Console**, tích chọn File Share vừa tạo, nhấn vào mục **Actions** và chọn **Delete** để xóa bỏ điểm chia sẻ.
3. Tiếp tục vào mục **Gateways**, chọn gateway appliance `lab24-filesgw` và tiến hành hủy kích hoạt hệ thống.
4. Vào phân hệ quản trị **EC2 Console** để thực hiện lệnh **Terminate** (Xóa bỏ) instance máy chủ ảo hóa.
5. Vào **S3 Console** để làm rỗng (**Empty**) và xóa S3 Bucket nhằm triệt tiêu hoàn toàn chi phí phát sinh sau bài thực hành.