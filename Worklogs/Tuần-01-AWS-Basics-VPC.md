# Minh chứng & Tài liệu Thực hiện - Tuần 1: Điện toán Đám mây AWS & Mạng Amazon VPC

### 📌 Thông tin Chung
- **Thời gian:** 15/06/2026 - 19/06/2026
- **Chủ đề chính:** Tổng quan Đám mây & AWS Cloud Basics, IAM Identity Center (Lab 12), Mạng Amazon VPC, Tường lửa bảo mật (Security Group & NACL), Kết nối liên VPC & Hybrid Networking.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Nghiên cứu AWS Module 1 & Thực hành Lab 12:**
   - **Lý thuyết:** Điện toán đám mây & 4 lợi ích cốt lõi; Sự khác biệt của AWS (Economies of Scale, Leadership Principles); Hạ tầng toàn cầu (Region, AZ, Edge Location); Công cụ quản lý (Console, CLI, SDK); Tối ưu chi phí & gói AWS Support.
   - **Thực hành Lab 12 (Identity Center):** Kích hoạt AWS Organizations, cấu hình AWS IAM Identity Center tập trung, khởi tạo User/Group, phân quyền qua Permission Sets, thiết lập Time-based Access Control và kiểm tra đăng nhập qua Portal & CLI.

2. **Triển khai AWS Intermediate Workshop & VPC Basics:**
   - **Identity & CLI:** Cài đặt và cấu hình AWS CLI v2.
   - **Mạng cơ bản (Amazon VPC):** Khái niệm VPC (vùng mạng cô lập logic, Multi-AZ) và phân chia Subnet (Public/Private Subnet, 5 IP Reserved của AWS).
   - **Thành phần mạng:** Route Table, Internet Gateway (IGW), Elastic Network Interface (ENI), Elastic IP (EIP), NAT Gateway và VPC Endpoint (Interface/Gateway Endpoint qua AWS PrivateLink).

3. **Quản trị Bảo mật & Bảo mật mạng VPC:**
   - Khung lý thuyết AWS CAF (Directive) và NIST CSF (Identify).
   - Phân tích cơ chế tường lửa ảo Security Group (Stateful ở cấp ENI) và Network ACL (Stateless ở cấp Subnet).
   - Cấu hình VPC Flow Logs ghi nhận nhật ký lưu lượng IP.

4. **Kết nối liên VPC & Xử lý sự cố:**
   - Troubleshooting lỗi cấu hình CLI, OIDC xác thực và phân quyền IAM cho CloudWatch Logs.
   - So sánh VPC Peering (kết nối trực tiếp 1-1, non-transitive) và Transit Gateway (TGW - Hub-and-Spoke).

5. **Mạng hỗn hợp Hybrid & Dịch vụ Cân bằng tải (ELB):**
   - Kết nối On-premises lên đám mây: VPN Site-to-Site, Client VPN, AWS Direct Connect.
   - Elastic Load Balancing (ELB): Application Load Balancer (ALB - Layer 7), Network Load Balancer (NLB - Layer 4), và Gateway Load Balancer (GWLB - Layer 3).

---

### 📊 Kết quả Đạt được & Minh chứng
- **Module 1 Document:** [Module 1](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%201/module-1.md)
- **Module 2 Document:** [Module 2](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%202/module-2.md)
- **Lab 12 Document:** [Lab 12](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%201/Hand-ons/Lab12.md)
- **Worklog Gốc Tuần 1:** [Worklog Tuần 1](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.1-Week1/_index.vi.md)
