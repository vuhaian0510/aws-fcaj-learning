# Bài Lab 25: Triển khai hệ thống lưu trữ chia sẻ với Amazon FSx for Windows File Server

> [!NOTE]
> **Bối cảnh thực tế:** Trong môi trường doanh nghiệp sử dụng hệ điều hành Windows, nhu cầu chia sẻ file dung lượng lớn giữa nhiều phòng ban đòi hỏi một hệ thống File Server có độ sẵn sàng cao, hỗ trợ giao thức SMB, định dạng đĩa NTFS và tích hợp sâu với hệ thống quản lý định danh tập trung (Active Directory).
>
> **Mục tiêu bài Lab:** Thực hành khởi tạo dịch vụ AWS Managed Microsoft Active Directory làm gốc xác thực định danh, cấu hình hệ thống tệp tin Amazon FSx for Windows File Server và thực hiện mount ổ đĩa mạng chia sẻ đồng thời lên nhiều máy chủ Windows EC2, áp dụng tính năng chống trùng lặp dữ liệu (Data Deduplication) để tối ưu hóa chi phí lưu trữ.

---

## I. Khởi tạo hệ thống quản lý định danh tập trung (AWS Managed AD)

**Mục tiêu:** Tạo một thư mục Active Directory được quản lý hoàn toàn bởi AWS để làm nền tảng xác thực tài khoản và phân quyền truy cập cho hệ thống Windows File Server.

### Các bước thực hiện:
1. Trên thanh tìm kiếm **AWS Management Console**, gõ `Directory Service` và truy cập vào dịch vụ.
2. Nhấn nút **Set up directory**, tại trang lựa chọn loại hình, tích chọn **AWS Managed Microsoft AD**.
3. Điền các thông số cấu hình cơ bản cho Directory:
   - **Fully qualified domain name (FQDN):** Nhập tên miền nội bộ của doanh nghiệp (Ví dụ: `corp.awsstudygroup.com`).
   - **Directory NetBIOS name:** Hệ thống tự động bốc từ FQDN (Ví dụ: `CORP`).
   - **Admin password:** Thiết lập mật khẩu tối cao cho tài khoản quản trị AD.
4. Tại phân hệ **VPC and subnets**, lựa chọn VPC mặc định của bạn và chọn 2 Subnet nằm ở 2 Vùng sẵn sàng (AZ) khác nhau để đảm bảo tính sẵn sàng cao (High Availability).
5. Nhấn **Next**, xem lại thông tin và nhấn **Create directory**.

   > [!NOTE]
   > Hệ thống sẽ mất khoảng 20-30 phút để khởi tạo hoàn tất cấu hình AD.

---

## II. Triển khai hệ thống tệp tin Amazon FSx for Windows File Server

**Mục tiêu:** Tạo một phân hệ lưu trữ chia sẻ chuẩn NTFS, hỗ trợ giao thức SMB và kết nối trực tiếp vào Active Directory vừa tạo.

### Các bước thực hiện:
1. Truy cập dịch vụ **FSx** từ thanh tìm kiếm Console, nhấn nút **Create file system**.
2. Tại trang lựa chọn loại hệ thống tệp, tích chọn **Amazon FSx for Windows File Server** và nhấn **Next**.
3. Cấu hình thông số đĩa (**File system details**):
   - **File system name:** Đặt tên nhận diện (Ví dụ: `win-file-share-lab25`).
   - **Deployment type:** Chọn **Single-AZ** cho môi trường Lab thử nghiệm (hoặc *Multi-AZ* cho production).
   - **Storage type:** Chọn loại ổ đĩa **SSD** để tối ưu hiệu năng đọc ghi.
   - **Storage capacity:** Cấp phát dung lượng đĩa tối thiểu (Ví dụ: `32 GiB`).
4. Cấu hình Mạng và Bảo mật (**Network & security**):
   - Chọn đúng **VPC** và **Subnet** trùng khớp với phân hệ hạ tầng mạng của Active Directory.
   - Tại mục **VPC Security Groups**, chọn hoặc tạo một Security Group đã mở sẵn cổng `445 (SMB)` để máy trạm có thể kết nối.
5. Xác thực định danh (**Windows authentication**):
   - Tích chọn **AWS Managed Microsoft Active Directory**.
   - Tại ô thả xuống, chọn đúng tên miền `corp.awsstudygroup.com` đã tạo ở mục I.
6. Giữ nguyên các cấu hình mặc định khác, nhấn **Next** và chọn **Create file system** để hệ thống khởi tạo.

---

## III. Kết nối (Mount) ổ đĩa mạng chia sẻ trên máy trạm Windows EC2

**Mục tiêu:** Đăng nhập vào các máy chủ Windows EC2 thuộc tên miền AD để tiến hành map ổ đĩa chia sẻ thông qua giao thức SMB và kiểm tra tính đồng bộ dữ liệu.

### Các bước thực hiện:
1. **Gia nhập máy chủ EC2 vào Domain:** Khởi chạy các máy chủ ảo Windows EC2, tiến hành cấu hình đưa các máy chủ này gia nhập (Join domain) vào mạng lưới Active Directory `corp.awsstudygroup.com`.
2. **Lấy câu lệnh kết nối mẫu:** Truy cập vào **FSx Console**, click vào hệ thống file vừa tạo, nhấn nút **Attach** ở góc phải để copy câu lệnh Command Prompt mẫu được AWS gợi ý sẵn.
3. **Thực thi Mount ổ đĩa:**
   - Sử dụng Remote Desktop (RDP) đăng nhập vào máy chủ Windows EC2 bằng tài khoản user thuộc Domain AD.
   - Mở công cụ **Command Prompt (CMD)** và dán đoạn mã lệnh đã copy có cấu trúc tương tự sau:
     ```cmd
     net use Z: \\fs-0123456789abcdef.corp.awsstudygroup.com\share /user:CORP\Admin
     ```
4. **Kiểm tra tính đồng bộ:**
   - Mở **File Explorer** trên máy EC2 thứ nhất, truy cập vào ổ đĩa mạng `Z:`, tiến hành tạo một thư mục dữ liệu chung.
   - Đăng nhập vào máy EC2 thứ hai, tiến hành mount cùng câu lệnh trên và xác nhận thư mục dữ liệu vừa tạo ở máy thứ nhất đã hiển thị đồng bộ theo thời gian thực (Real-time).

---

## IV. Dọn dẹp tài nguyên Lab (Clean up)

Để tránh phát sinh chi phí không mong muốn, hãy thực hiện dọn dẹp theo các bước sau:

1. Trên giao diện **File Explorer** của các máy trạm Windows, nhấn chuột phải vào ổ đĩa `Z:` và chọn **Disconnect** để gỡ ánh xạ đĩa mạng.
2. Truy cập vào **FSx Console**, tích chọn hệ thống file `win-file-share-lab25`, nhấn vào mục **Actions** $\rightarrow$ chọn **Delete file system**. Hệ thống sẽ hỏi bạn có muốn tạo bản sao lưu cuối cùng (Final backup) hay không, tích chọn **No** và xác nhận xóa.
3. Truy cập vào **Directory Service Console**, chọn thư mục domain `corp.awsstudygroup.com`, nhấn **Actions** $\rightarrow$ chọn **Delete** để hủy bỏ Active Directory, triệt tiêu hoàn toàn chi phí phát sinh sau bài thực hành.