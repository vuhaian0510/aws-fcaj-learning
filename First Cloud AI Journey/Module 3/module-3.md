# AWS FCAJ - Module 3: Dịch vụ Điện toán Máy chủ ảo và Lưu trữ khối (Compute VM & Storage)

### 1. Tổng quan Module 3

+ Nội dung tập trung:
  - Các dịch vụ tính toán (Compute).
  - Các giải pháp lưu trữ chia sẻ cấp độ file (Network File Share).
  - Dịch vụ máy chủ ảo cấu hình sẵn (Amazon Lightsail).
  - Công cụ di trú và khôi phục thảm họa (AWS MGN).

---

### 2. Amazon Elastic Compute Cloud (Amazon EC2)

+ Khái niệm & Đặc điểm cốt lõi:
  - Khái niệm: Cung cấp các máy chủ ảo (gọi là EC2 Instances) trên đám mây, có vai trò tương đương với máy chủ vật lý truyền thống hoặc máy ảo ở môi trường On-premises.
  - Các ưu điểm vượt trội:
    * Provision nhanh: Khởi tạo và phân bổ tài nguyên gần như ngay lập tức.
    * Elasticity (Tính co giãn): Tự động tăng hoặc giảm số lượng máy chủ dựa theo nhu cầu thực tế của lượng người dùng.
      => Giúp tối ưu hóa chi phí.
    * Nâng cấp phần cứng tự động: AWS cam kết liên tục cập nhật các thế hệ CPU, chipset mới mạnh mẽ hơn với giá thành rẻ hơn theo thời gian.
      => Xóa bỏ chi phí khấu hao và thay thế phần cứng lớn của mô hình truyền thống.

+ Sơ đồ kiến trúc phân lớp của EC2:
  - Khái niệm: Hoạt động dựa trên sự phân tách độc lập giữa sức mạnh tính toán và lưu trữ.
  - Mô hình: `[Hardware Node (AWS)] -> [Hypervisor Layer] -> [EC2 Instances]`
  - Các thành phần:
    * Hardware Node: Hạ tầng phần cứng vật lý do AWS quản lý hoàn toàn. Người dùng định hướng vị trí qua Availability Zone (AZ) và tùy chọn Placement Options (đặt các instance gần nhau để giảm latency, hoặc đặt xa nhau trên các hardware node khác nhau để cô lập lỗi phần cứng).
    * Hypervisor Layer: Lớp ảo hóa quản lý phần cứng. AWS hỗ trợ các dạng như PV, HVM và công nghệ mới nhất là KVM Nitro.
      => Hệ thống ảo hóa Nitro giúp tăng hiệu năng từ 30% đến 40% so với thế hệ cũ.
    * Amazon Machine Image (AMI): File template (ảnh đĩa) chứa hệ điều hành (Root Volume), quyền truy cập (Permission) và cấu hình thiết bị (Block Device Mapping).
      => Dùng để khởi tạo hàng loạt instance đồng nhất.

+ Phân loại cấu hình (Instance Types) & Chipset:
  - Khái niệm: Cấu hình máy chủ được quyết định qua việc chọn Instance Type, quy định số lượng vCPU, RAM, băng thông mạng (Network Bandwidth) và hiệu năng lưu trữ.
  - Các loại chip xử lý:
    * Chip Intel: Chi phí cao nhất.
    * Chip AMD: Tiết kiệm chi phí khoảng 10% so với Intel.
    * Chip ARM (AWS Graviton 1, 2, 3): Mang lại hiệu năng trên giá thành (Price/Performance) tối ưu nhất.
      => Giúp tiết kiệm chi phí lên tới 40%.
    * Lưu ý khi dùng chip ARM: Chỉ chạy trên hệ điều hành Linux và yêu cầu kiểm thử xem ngôn ngữ lập trình/thư viện của ứng dụng có hỗ trợ kiến trúc ARM hay không.

+ Cơ chế bảo mật và Đăng nhập (Key Pairs):
  - Khái niệm: Sử dụng mã hóa bất đối xứng bao gồm một cặp Public Key (AWS lưu) và Private Key (người dùng tải về máy dưới dạng `.pem` hoặc `.ppk`) để bảo mật thông tin đăng nhập.
  - Cơ chế truy cập:
    * Với Linux Instance: Sử dụng Private Key qua các SSH Client (như PuTTY, Terminal) để kết nối trực tiếp.
    * Với Windows Instance: Password của tài khoản administrator mặc định bị mã hóa.
      => Người dùng phải submit file Private Key lên AWS Management Console để giải mã lấy password thô, sau đó sử dụng giao thức Remote Desktop (RDP) để truy cập.

---

### 3. Hệ thống lưu trữ cho EC2: EBS & Instance Store

+ Amazon Elastic Block Store (Amazon EBS):
  - Khái niệm: Là dịch vụ lưu trữ dạng khối (Block Storage), cung cấp các ổ đĩa ảo (EBS Volumes) gán vào EC2.
  - Đặc điểm kỹ thuật:
    * Kiến trúc độc lập: EBS hoạt động tách biệt với EC2 và kết nối thông qua một mạng riêng độc lập (EBS Network), không đi chung đường với card mạng Internet của EC2.
      => Giúp bảo đảm hiệu năng đọc/ghi không bị ảnh hưởng bởi traffic mạng bên ngoài.
    * Độ sẵn sàng cực cao (99.999%): Dữ liệu trong 1 EBS Volume được tự động sao chép (replicate) sang 3 node lưu trữ vật lý khác nhau trong cùng một Availability Zone.
      => Vượt trội hơn hẳn so với cơ chế chạy RAID 10 truyền thống.
    * Tính phí: Tính theo dung lượng cấp phát (Provisioned Capacity).
      => Cần cấp phát vừa đủ và mở rộng dần để tránh lãng phí.
    * EBS Multi-Attach: Tính năng (chạy trên Nitro Hypervisor) cho phép gán một ổ đĩa EBS đồng thời vào nhiều EC2 instance.
  - Cơ chế sao lưu (Snapshot):
    * Lưu trữ trực tiếp vào Amazon S3.
    * Snapshot đầu tiên luôn là Full Backup.
    * Các snapshot tiếp theo hoạt động theo cơ chế Incremental Backup.
      => Chỉ sao lưu phần dữ liệu thay đổi để giảm chi phí.

+ EC2 Instance Store:
  - Khái niệm: Là vùng đĩa NVMe cục bộ được gắn vật lý trực tiếp vào Hardware Node chạy EC2.
  - Hiệu năng: Tốc độ cực cao, độ trễ siêu thấp, hỗ trợ hàng triệu IOPS mỗi giây.
  - Rủi ro mất dữ liệu (Ephemeral Storage):
    * Dữ liệu KHÔNG được replicate dự phòng.
      => Nếu thực hiện lệnh Stop / Shutdown máy chủ, dữ liệu trên Instance Store sẽ bị xóa sạch hoàn toàn.
      => Dữ liệu không bị xóa nếu chỉ thực hiện Restart / Reboot hoặc khi hệ thống bị lỗi crash phần mềm/hypervisor.
  - Ứng dụng thích hợp: Dùng làm bộ nhớ đệm (Caching, Buffer), swap string, hoặc lưu file log tạm thời (các dữ liệu có thể tái tạo lại được).
    => Không dùng cho dữ liệu quan trọng nếu chưa có kiến trúc replicate thủ công sang EBS.

---

### 4. Quản trị và Tự động hóa nâng cao trên EC2

+ EC2 User Data (Khởi tạo tự động):
  - Khái niệm: Là một đoạn Script (Bash script trên Linux, Batch/PowerShell trên Windows) được hệ thống chạy tự động duy nhất một lần khi máy chủ khởi tạo lần đầu tiên từ AMI.
  - Mục đích: Kết hợp với một bản ảnh gốc chuẩn hóa bảo mật (Golden Image), nạp các đoạn User Data khác nhau để tự động cài đặt ứng dụng.
    => Biến các instance thành các vai trò khác nhau (Web Server, App Server, Database Server) mà không cần cấu hình thủ công.
  - Truy cập thông tin User Data: Có thể lấy thông tin User Data nội bộ từ địa chỉ IP Link-local: `http://169.254.169.254/latest/user-data`.

+ EC2 Metadata (Thông tin cấu hình hệ thống):
  - Khái niệm: Là tập hợp các thông tin liên quan đến chính bản thân instance đó (như Private IP, Public IP, Instance ID, Security Group, Hostname).
  - Mục đích: Sử dụng cho các đoạn script tự động hóa chạy bên trong máy chủ.
    => Ví dụ: Lấy đúng Instance ID từ bên ngoài AWS để chạy lệnh script tự động tăng dung lượng ổ đĩa EBS.
  - Truy cập thông tin Metadata: Qua địa chỉ IP Link-local: `http://169.254.169.254/latest/meta-data`.

+ EC2 Auto Scaling:
  - Khái niệm: Tự động tăng (Scale out) hoặc giảm (Scale in) số lượng máy chủ EC2 dựa theo các điều kiện giám sát thực tế (Scaling Policy - như lượng kết nối hoạt động Active Connection Count, phần trăm CPU sử dụng).
  - Cơ chế vận hành:
    * Khi traffic vượt ngưỡng, Auto Scaling Group (ASG) khởi tạo thêm instance (có thể cấu hình phân bổ nằm trên nhiều AZ khác nhau để đảm bảo tính sẵn sàng cao).
    * Tự động đăng ký (Register) các máy chủ mới này vào bộ cân bằng tải Elastic Load Balancer (ELB) để chia tải, hạ nhiệt cho hệ thống.
    * Khi sự kiện kết thúc, traffic giảm xuống, ASG sẽ tiến hành hủy đăng ký (De-register) trên ELB và tiến hành xóa bỏ (Terminate) các instance thừa để tối ưu chi phí.
  - Phối hợp các Mô hình giá (Pricing Options) trong ASG:
    * On-Demand: Trả theo giờ/giây sử dụng, chi phí cao nhất.
      => Phù hợp cho các tải chạy ngắn hạn (dưới 6 tiếng/ngày).
    * Reserved Instances (RI) / Savings Plans: Cam kết sử dụng dài hạn 1-3 năm để nhận mức chiết khấu cao.
      => Thích hợp áp dụng cho số lượng instance tối thiểu (Minimum size) của ASG luôn phải chạy 24/7.
    * Spot Instances: Thuê tài nguyên dư thừa với giá rẻ (giảm đến 90%), nhưng AWS có quyền tắt máy chủ trong vòng 2 phút báo trước.
      => Thích hợp cho tầng xử lý tải co giãn trên cùng nếu ứng dụng được thiết kế có khả năng chịu lỗi cao.

---

### 5. Dịch vụ máy chủ cấu hình sẵn: Amazon Lightsail

+ Khái niệm: Là dịch vụ điện toán đám mây trọn gói, đơn giản hóa, gộp chung cả máy chủ, lưu trữ, và băng thông mạng vào một mức giá cố định theo tháng (chỉ từ 3.5 USD/tháng).
+ Đặc điểm & Vận hành:
  - Băng thông: Đi kèm định mức dữ liệu truyền tải (Data Transfer Out) với giá thành rẻ hơn rất nhiều so với EC2 (ví dụ: gói 3.5 USD có sẵn 500GB Data Transfer Out).
  - Vận hành: Chạy trong một môi trường VPC độc lập do Lightsail quản lý.
    => Để kết nối Lightsail Instance với các tài nguyên thuộc VPC thông thường của EC2, sử dụng tính năng One-Click VPC Peering.
  - Khả năng mở rộng: Hỗ trợ tạo snapshot và cung cấp tính năng Convert snapshot sang EC2 khi ứng dụng phát triển vượt ngưỡng, yêu cầu các tính năng nâng cao của EC2.
+ Use case: Phù hợp cho các ứng dụng tải nhẹ, website nhỏ, môi trường Dev/Test không yêu cầu CPU tải cao liên tục quá 2 giờ/ngày.

---

### 6. Các giải pháp Lưu trữ mạng chia sẻ (Network File Share)

+ Amazon Elastic File System (Amazon EFS):
  - Giao thức: Sử dụng giao thức NFSv4 (Network File System version 4).
  - Hệ điều hành hỗ trợ: Chỉ hỗ trợ Linux.
  - Đặc điểm nổi bật:
    * Cho phép gán đồng thời vào hàng ngàn EC2 instance trên nhiều AZ khác nhau.
    * Quy mô tự động mở rộng lên đến cấp độ Petabyte.
    * Hỗ trợ kết nối mở rộng cho cả các máy chủ On-premises thông qua AWS Direct Connect hoặc VPN.
  - Cơ chế tính phí: Tính theo dung lượng thực dụng (dung lượng thực tế ghi vào ổ đĩa), dùng bao nhiêu trả bấy nhiêu.
    => Không tính theo dung lượng cấp phát ảo ban đầu.

+ Amazon FSx (FSx cho Windows File Server):
  - Giao thức: Sử dụng giao thức SMB (Server Message Block) và định dạng ổ đĩa NTFS.
  - Hệ điều hành hỗ trợ: Hỗ trợ cả Windows và Linux.
  - Cơ chế tính phí: Tính theo dung lượng thực tế sử dụng.
  - Tính năng đặc trưng - Data Deduplication (Chống trùng lặp dữ liệu): FSx tự động phân tích dữ liệu ở cấp độ Block (Khối) để phát hiện và loại bỏ các phần dữ liệu trùng lặp (chỉ lưu 1 bản và tạo con trỏ trỏ tới).
    => Tính năng này giúp tiết kiệm từ 30% đến 50% chi phí lưu trữ cho các hệ thống File Server doanh nghiệp.

---

### 7. Dịch vụ dịch chuyển hạ tầng và Phục hồi sự cố: AWS MGN

+ Khái niệm: AWS Application Migration Service (MGN) là công cụ chuyên dụng để di trú (migrate) các máy chủ vật lý hoặc máy ảo từ môi trường On-premises lên AWS, đồng thời làm giải pháp khắc phục thảm họa (Disaster Recovery - DR).
+ Cơ chế hoạt động:
  - Cài đặt tác nhân (Agent) lên các máy chủ nguồn ở On-premises.
  - Dữ liệu từ các ổ đĩa nguồn (SAN, NAS, Local Disk) được replicate liên tục (đồng bộ hoặc bất đồng bộ) lên một vùng đệm gọi là Staging VPC trên AWS.
  - Tại Staging VPC, AWS chỉ sử dụng các máy chủ quản lý luồng dữ liệu cấu hình siêu nhỏ (Replication Servers - 1 máy nhỏ có thể cân tải đón dữ liệu cho 25 máy chủ nguồn lớn) kết nối với các ổ đĩa EBS Volumes tương ứng.
    => Cơ chế này giúp doanh nghiệp tối ưu chi phí làm DR vì chỉ tốn chủ yếu là tiền lưu trữ đĩa thô, không cần duy trì dàn máy chủ lớn chạy 24/7.
  - Khi thực hiện Cut-over (chuyển giao dịch vụ chính thức) hoặc khi On-premises gặp thảm họa, MGN sẽ tự động đọc cấu hình chỉ định để khởi chạy (Launch) các máy chủ mục tiêu Target EC2 Instances với cấu hình chuẩn xác.
    => Map các ổ đĩa EBS đã được đồng bộ sang để hệ thống hoạt động ngay lập tức.