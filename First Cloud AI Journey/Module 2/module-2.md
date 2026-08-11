# AWS FCAJ - Module 2: Amazon VPC & Bảo mật mạng, Cân bằng tải (ELB)

## 1. Khái niệm cốt lõi về Amazon VPC & Thành phần mạng

* **Amazon VPC (Virtual Private Cloud)**:
  - Khái niệm: Mạng ảo riêng biệt, cô lập logic trên hạ tầng đám mây AWS. Là xương sống (backbone) đóng vai trò thiết yếu trong việc kết nối các tài nguyên và dịch vụ khác nhau.
  - Đặc điểm: VPC bao quát nhiều Availability Zone (AZ) để chạy tài nguyên trên nhiều AZ khác nhau (Multi-AZ) nhằm đảm bảo tính sẵn sàng cao. Khi tạo VPC, cần khai báo 1 lớp mạng CIDR IPv4 (bắt buộc) và IPv6 (tùy chọn).
  - Giới hạn & Mục đích: Giới hạn mặc định ban đầu là 5 VPC trên mỗi AWS Region trên mỗi tài khoản AWS. Mục đích chính thường dùng để phân tách các môi trường (Production/Dev/Test/Staging).

* **Subnet (Mạng con)**:
  - Khái niệm: Dải IP nhỏ phân chia từ dải IP tổng của VPC. Không gian mạng của Subnet bắt buộc phải là tập con của VPC CIDR. Mỗi Subnet bắt buộc nằm trọn trong một AZ cụ thể.
  - Phân loại: Public Subnet và Private Subnet bản chất kỹ thuật giống hệt nhau. Sự khác biệt (quy ước) là nếu một subnet được cấu hình định tuyến ra Internet thông qua Internet Gateway thì gọi là Public Subnet, các tài nguyên bên trong có quyền truy cập ra internet.
  - > [!IMPORTANT]
    > **5 địa chỉ IP được AWS giữ lại (Reserved IPs):** Trong mỗi Subnet, AWS luôn giữ lại 5 địa chỉ IP cho các mục đích quản trị hệ thống và định tuyến.
    > 
    > Ví dụ với Subnet có CIDR `10.10.1.0/24`:
    > * `10.10.1.0`: Địa chỉ Network (Network Address).
    > * `10.10.1.1`: Địa chỉ cho bộ định tuyến (Router).
    > * `10.10.1.2`: Địa chỉ cho dịch vụ DNS server của AWS.
    > * `10.10.1.3`: Địa chỉ dành riêng cho các tính năng tương lai của AWS.
    > * `10.10.1.255`: Địa chỉ Broadcast (Broadcast Address).
    > 
    > *=> Do đó, các IP này không thể được sử dụng để gán cho các máy chủ EC2 hay tài nguyên của bạn.*

* **Elastic Network Interface (ENI)**:
  - Khái niệm: Card mạng ảo gắn trực tiếp vào máy chủ ảo. Địa chỉ IP của máy chủ không gán trực tiếp vào máy chủ mà gán qua ENI (được tự động tạo và gán khi khởi tạo máy chủ).
  - Đặc điểm: Có thể tháo rời và di chuyển sang EC2 Instance khác trong cùng Subnet mà vẫn giữ nguyên cấu hình (IP Private, Elastic IP, địa chỉ MAC).

* **Elastic IP (EIP)**:
  - Khái niệm: Địa chỉ public IPv4 tĩnh, có thể liên kết với một ENI.
  - Đặc điểm: Không thay đổi khi máy chủ ảo restart/stop. EIP sẽ bị tính phí nếu không được liên kết với một instance đang chạy.

* **Route Table (Bảng định tuyến)**:
  - Khái niệm: Tập hợp các quy tắc định tuyến (routes) xác định hướng đi của luồng dữ liệu trong VPC. Bảng định tuyến sẽ được gán (associate) vào Subnet.
  - Đặc điểm:
    * Khi tạo VPC, AWS luôn tự động tạo ra một **Default Route Table** (Main Route Table). Bảng này không thể bị xóa và mặc định chứa một route cục bộ (`VPC-CIDR -> local`) cho phép tất cả các Subnet trong VPC liên lạc với nhau.
    * Có thể tạo thêm **Custom Route Table** để cấu hình định tuyến tùy chỉnh (ví dụ cấu hình Public Subnet), nhưng không bao giờ xóa được rule default (`local`).

* **Internet Gateway (IGW)**:
  - Khái niệm: Cổng kết nối 2 chiều (in/out) giữa VPC và Internet. Được quản lý hoàn toàn bởi AWS, tự động mở rộng theo chiều ngang (scale out), không cần quản trị hạ tầng mạng.
  - > [!IMPORTANT]
    > **Yêu cầu bắt buộc để máy chủ trong Public Subnet ra được Internet:**
    > 1. Máy chủ phải được gán địa chỉ **IP Public** (hoặc Elastic IP).
    > 2. Bảng định tuyến (**Route Table**) của Subnet chứa máy chủ phải có quy tắc định tuyến trỏ đến Internet Gateway (`0.0.0.0/0 -> IGW-ID`).

* **NAT Gateway**:
  - Khái niệm: Cho phép các tài nguyên trong Private Subnet truy cập ra ngoài Internet (để tải bản vá, gọi API bên ngoài...) nhưng ngăn chặn hoàn toàn chiều ngược lại (Internet không thể chủ động kết nối vào máy chủ).
  - Cơ chế hoạt động:
    * **NAT Gateway phải nằm trong Public Subnet** (để có thể đi qua IGW ra ngoài).
    * Tạo Custom Route Table cho Private Subnet, cấu hình route entry trỏ đến NAT Gateway (`0.0.0.0/0 -> nat-ID`).
    * Luồng truyền thông: `EC2 (Private Subnet) -> NAT Gateway (Public Subnet) -> Internet Gateway -> Internet`.

* **VPC Endpoint**:
  - Khái niệm: Cho phép kết nối các tài nguyên nằm trong VPC tới các dịch vụ AWS khác mà không cần đi qua Internet (kết nối qua mạng private nội bộ của AWS nhờ công nghệ AWS PrivateLink).
  - Phân loại:
    * **Interface Endpoint**: Sử dụng một card mạng ảo (ENI) trong VPC với địa chỉ IP Private để kết nối với dịch vụ mục tiêu.
    * **Gateway Endpoint**: Sử dụng cơ chế cập nhật Route Table để định tuyến trực tiếp tới dịch vụ (chỉ hỗ trợ **Amazon S3** và **DynamoDB**).
    * *Lý do sử dụng:* Kết nối từ VPC ra ngoài S3 thông qua IP Public sẽ tốn chi phí truyền dữ liệu và tốc độ kết nối chậm hơn so với việc đi qua S3 Gateway Endpoint sử dụng IP Private nội bộ.

---

## 2. Cơ chế bảo mật và các giải pháp kết nối liên VPC

* **Security Group (SG)**:
  - Khái niệm: Tường lửa ảo ở cấp độ card mạng (ENI) của từng tài nguyên.
  - Đặc điểm:
    * **Stateful** (lưu trạng thái): Khi cho phép lưu lượng đi vào (Inbound), hệ thống tự động cho phép lưu lượng phản hồi đi ra (Outbound) tương ứng và ngược lại.
    * Quy tắc: Chỉ hỗ trợ cấu hình "cho phép" (Allow), mặc định từ chối mọi truy cập khác.
    * Có thể hạn chế theo giao thức, dải IP nguồn/đích, cổng kết nối, hoặc tham chiếu tới một Security Group khác.
    * Mặc định khi tạo mới: Chặn mọi truy cập đến (Inbound) và cho phép mọi truy cập đi (Outbound).

* **Network ACL (NACL)**:
  - Khái niệm: Tường lửa bảo mật ở cấp độ Subnet (ảnh hưởng tới mọi máy chủ nằm trong Subnet đó cùng một lúc).
  - Đặc điểm:
    * **Stateless** (không lưu trạng thái): Phải cấu hình tường minh luật cho cả 2 chiều Inbound và Outbound.
    * Quy tắc: Hỗ trợ cả luật **Allow** (Cho phép) và **Deny** (Từ chối).
    * Đọc luật theo thứ tự ưu tiên số hiệu rule từ trên xuống dưới (thỏa rule nào trước sẽ áp dụng rule đó).
    * Mặc định khi tạo mới: Cho phép mọi truy cập đến và đi.

* **VPC Flow Logs**:
  - Khái niệm: Tính năng ghi nhận nhật ký lưu lượng IP đến và đi từ các giao diện mạng (ENI) trong VPC.
  - Đặc điểm:
    * Logs có thể xuất bản lên Amazon CloudWatch Logs hoặc S3.
    * **Không ghi nhận nội dung của gói tin**, chỉ capture các metadata quan trọng như: Account ID, ENI ID, Source IP, Destination IP, Port, Action (Accept/Reject)...
    * Dùng để kiểm tra các truy cập bị từ chối và phát hiện hành vi bất thường.

* **VPC Peering**:
  - Khái niệm: Kết nối trực tiếp 1-1 giữa hai VPC, truyền tin qua IP Private với độ trễ thấp và bảo mật cao.
  - Đặc điểm:
    * Không hỗ trợ định tuyến chuyển tiếp (Non-transitive routing) (VPC A peering B, B peering C thì A không tự động kết nối được với C).
    * Không hỗ trợ nếu hai VPC bị trùng lặp dải IP (Overlap IP address space).
    * Phải cấu hình Route Table thủ công. Có thể cấu hình kết nối ở mức cả VPC hoặc chỉ giới hạn ở mức 1 subnet cụ thể.
    * Hạn chế: Khó mở rộng khi số lượng VPC lớn. Ví dụ: Với 30 VPC cần kết nối mesh hoàn toàn, ta phải tạo đến 435 kết nối Peering (Công thức $N(N-1)/2$), gây tốn rất nhiều công sức cấu hình.

* **Transit Gateway (TGW)**:
  - Khái niệm: Bộ định tuyến trung tâm (Hub) kết nối hàng loạt VPC & On-premises, giúp đơn giản hóa mạng lưới định tuyến phức tạp.
  - Cơ chế hoạt động:
    * Sử dụng **Transit Gateway Attachment (TGA)** để gán các subnet của từng VPC vào Transit Gateway.
    * TGA hoạt động ở quy mô Availability Zone (AZ). Khi một subnet thuộc một AZ có kết nối TGA với TGW, tất cả các subnet khác trong cùng AZ đó đều có thể kết nối với TGW.
    * Phải cấu hình Route Table của Subnet trỏ đến ID của TGW, đồng thời cấu hình Route Table bên trong TGW trỏ đến các TGA tương ứng.

---

## 3. Giải pháp kết nối mạng Hybrid và Dịch vụ cân bằng tải (ELB)

* **Kết nối Hybrid (Mạng hỗn hợp)**:
  - **VPN Site-to-Site**: Kết nối mã hóa IPsec VPN an toàn giữa toàn bộ site On-premises và AWS VPC.
    * Yêu cầu 2 đầu kết nối: **Customer Gateway (CGW)** ở phía khách hàng (thiết bị cứng hoặc phần mềm) và **Virtual Private Gateway (VGW)** ở phía AWS (chia làm 2 endpoint nằm ở 2 AZ khác nhau để dự phòng).
  - **VPN Client-to-Site**: Cho phép thiết bị cá nhân (Client) kết nối trực tiếp vào VPC.
    * Thực tế chi phí rất cao (tính tiền theo giờ chạy). AWS khuyến nghị nên dùng giải pháp VPN của bên thứ ba (Third-party) nếu muốn tối ưu chi phí.
  - **AWS Direct Connect**: Đường truyền vật lý riêng biệt, độc quyền từ On-premises đến AWS qua đối tác viễn thông (Viettel, FPT, CMC...).
    * Độ trễ cực thấp và ổn định (khoảng 20ms - 30ms từ Việt Nam).
    * Phân loại kết nối: **Dedicated Connections** (do AWS cung cấp trực tiếp) và **Hosted Connections** (thông qua Direct Connect Partners, cho phép linh hoạt thay đổi băng thông theo nhu cầu).
    * *Lý do sử dụng:* Không tự động mã hóa dữ liệu. Cần cấu hình chồng thêm VPN Site-to-Site để bảo mật tối đa.

* **Elastic Load Balancing (ELB)**:
  - Khái niệm: Dịch vụ cân bằng tải được quản lý bởi AWS, phân phối lưu lượng cho nhiều EC2 Instance, Container hoặc IP.
  - Vị trí: Có thể đặt ở Public Subnet (để nhận request từ Internet) hoặc Private Subnet.
  - Định danh: ELB được cấp một tên miền DNS. Người dùng truy cập Load Balancer bắt buộc thông qua DNS này (ngoại trừ Network Load Balancer hỗ trợ gán IP tĩnh).
  - Tính năng chính:
    * **Health check**: Giám sát trạng thái của các server đích, tự động loại bỏ các server lỗi khỏi luồng phân phối traffic.
    * **Sticky Sessions (Session Affinity)**: Giữ cố định kết nối của người dùng vào một server cụ thể trong suốt phiên làm việc, cần thiết khi ứng dụng lưu trữ trạng thái người dùng (Session state) cục bộ trên server.
    * **Access Logs**: Nhật ký truy cập chi tiết của ELB được lưu trữ an toàn trên **Amazon S3** phục vụ phân tích và xử lý lỗi.
  - Phân loại Load Balancer:
    - **Application Load Balancer (ALB) [Layer 7 - Tầng ứng dụng]**:
      * Hoạt động với giao thức HTTP/HTTPS.
      * Hỗ trợ **Path-based routing** (định tuyến dựa trên đường dẫn URL, ví dụ: `/mobile`, `/desktop` trỏ tới các Target Group khác nhau) và Host-based routing.
      * Hỗ trợ chuyển traffic tới Target ngoài VPC (qua IP address), EC2, Lambda, Container.
    - **Network Load Balancer (NLB) [Layer 4 - Tầng giao vận]**:
      * Hoạt động với giao thức TCP, TLS, UDP.
      * Hỗ trợ set **IP tĩnh** cho từng AZ kết nối.
      * Hiệu năng cao nhất, xử lý hàng triệu request/giây với độ trễ cực thấp.
      * Hỗ trợ chuyển traffic tới Target ngoài VPC, EC2, Container.
    - **Classic Load Balancer (CLB)**: Dòng cũ hoạt động ở cả Layer 4 và Layer 7, kết hợp tính năng của ALB và NLB ở dạng cơ bản. Không khuyến nghị cho thiết kế mới.
    - **Gateway Load Balancer (GWLB) [Layer 3 - Tầng mạng]**:
      * Lắng nghe toàn bộ IP packets và forward tới các Virtual Appliance (tường lửa ảo bên thứ ba).
      * Sử dụng giao thức **GENEVE** trên cổng **6081**.
      * Cho phép route traffic tới các virtual appliance được AWS hỗ trợ.
