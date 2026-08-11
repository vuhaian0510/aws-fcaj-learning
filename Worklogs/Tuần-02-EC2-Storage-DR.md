# Minh chứng & Tài liệu Thực hiện - Tuần 2: Máy chủ Amazon EC2, Bộ lưu trữ & Phục hồi Sự cố (DR)

### 📌 Thông tin Chung
- **Thời gian:** 22/06/2026 - 26/06/2026
- **Chủ đề chính:** Quản trị EC2 & Nitro Hypervisor, Đĩa EBS & Instance Store, Auto Scaling Group (ASG), Amazon S3 & Glacier, Shared Storage (EFS & FSx), Storage Gateway (Lab 24), Managed AD (Lab 25), S3 Hosting & CloudFront (Lab 57), Disaster Recovery & AWS Backup.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **AWS Compute & EC2 Administration:**
   - **Compute Theory:** Kiến trúc máy chủ ảo EC2, Nitro Hypervisor, Instance Types (Intel, AMD, AWS Graviton), Key Pairs bảo mật (SSH/RDP).
   - **Storage for Compute:** Phân biệt EBS (Block Storage, Multi-Attach) và Instance Store (NVMe Ephemeral); quản lý AMI và Incremental Snapshots.
   - **Hands-on Practice:** Khởi tạo EC2 Instance, kết nối SSH/RDP, định dạng và mount đĩa EBS mới.

2. **HA Operations, Shared Storage & Migration:**
   - **Automation & Scaling:** User Data script tự động hóa web server, EC2 Metadata (`169.254.169.254`), Auto Scaling Group (ASG) kết hợp Load Balancer (ELB).
   - **Amazon Lightsail:** Gói điện toán cố định, One-Click VPC Peering.
   - **Shared Storage:** Amazon EFS (NFSv4) và Amazon FSx for Windows Server (SMB/NTFS, Data Deduplication).
   - **Migration:** AWS Application Migration Service (AWS MGN), Replicate về Staging VPC và Cut-over.

3. **AWS Storage & Disaster Recovery:**
   - **Amazon S3 & Glacier:** Độ bền 11 số 9, S3 Storage Classes, Lifecycle Management, S3 Access Points, Static Website Hosting, CORS, Gateway Endpoints, Versioning (Delete Marker).
   - **Glacier Vaults & Vault Lock:** 3 tốc độ khôi phục (Expedited, Standard, Bulk) và Vault Lock compliance.
   - **Hybrid Storage & Snow Family:** Snowball, Snowball Edge, Snowmobile; AWS Storage Gateway (File, Volume, Tape Gateway).
   - **Disaster Recovery (DR) & AWS Backup:** Chỉ số RTO/RPO, 4 chiến lược DR (Backup & Restore, Pilot Light, Warm Standby, Multi-site Active-Active), AWS Backup.

4. **Thực hành Labs (Lab 57, Lab 24, Lab 25):**
   - **Lab 57 (S3 Static Website & CloudFront):** Tạo S3 Bucket, bật Website Hosting, nạp Bucket Policy, tích hợp CloudFront Distribution (OAI).
   - **Lab 24 (AWS Storage Gateway File Gateway):** Khởi tạo File Gateway trên EC2, cấu hình SMB File Share kết nối S3, mount đĩa mạng `Z:` trên client.
   - **Lab 25 (Amazon FSx for Windows File Server):** Khởi tạo AWS Managed Microsoft AD, triển khai FSx for Windows, join EC2 vào Active Directory và mount đĩa SMB.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Module 3 Document:** [Module 3](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%203/module-3.md)
- **Module 4 Document:** [Module 4](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%204/module-4.md)
- **Lab 24 Document:** [Lab 24](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab24.md)
- **Lab 25 Document:** [Lab 25](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab25.md)
- **Lab 57 Document:** [Lab 57](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%204/Hand-ons/Lab57.md)
- **Worklog Gốc Tuần 2:** [Worklog Tuần 2](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.2-Week2/_index.vi.md)
