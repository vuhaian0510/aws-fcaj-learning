# Minh chứng & Tài liệu Thực hiện - Tuần 3: Bảo mật, IAM, Cơ sở Dữ liệu & CI/CD EKS

### 📌 Thông tin Chung
- **Thời gian:** 29/06/2026 - 03/07/2026
- **Chủ đề chính:** Shared Responsibility Model, IAM Governance, Cognito, KMS (Lab 33), Security Hub (Lab 18), IAM Labs (Lab 02, Lab 44, Lab 48, Lab 30), Database RDBMS vs NoSQL, RDS & Aurora, Redshift, ElastiCache, CI/CD EKS với CodePipeline & GitHub.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Bảo mật & Quản trị Định danh (Security & Identity):**
   - **Triết lý & Trách nhiệm:** Shared Responsibility Model (Security OF vs IN the Cloud), triết lý "Job Zero".
   - **Kiểm soát Truy cập:** IAM Least Privilege, STS Temporary Credentials, AWS Organizations SCPs, Identity Center SSO.
   - **Định danh Ứng dụng:** Amazon Cognito (User Pools & Identity Pools).
   - **Bảo vệ Dữ liệu & Tuân thủ:** Envelope Encryption với AWS KMS và giám sát qua AWS Security Hub.

2. **Hands-on Security Labs (Labs 02, 44, 48, 18, 30, 33):**
   - **Lab 02:** Khởi tạo IAM User/Group, cấu hình MFA và Switch Role.
   - **Lab 44:** Trust Policy ràng buộc điều kiện Source IP và time-window.
   - **Lab 48:** IAM Role (Instance Profile) cho EC2 instance.
   - **Lab 18:** AWS Security Hub đánh giá tuân thủ tự động.
   - **Lab 30:** IAM Permission Boundary kiểm soát giới hạn quyền tối đa.
   - **Lab 33:** KMS CMK đối xứng, mã hóa phong bì S3 và CloudTrail-Athena log analytics.

3. **Dịch vụ Cơ sở dữ liệu và Lưu trữ bộ nhớ đệm:**
   - **Khái niệm cơ bản & Phân loại:** Primary/Foreign Key, Normalization, Indexing, Partitioning, Buffer; RDBMS vs NoSQL (ACID vs BASE); OLTP vs OLAP.
   - **Dịch vụ AWS Database:** Amazon RDS (Multi-AZ & Read Replicas), Amazon Aurora (Cluster Volume 6 copies trên 3 AZs, Backtrack, Fast Cloning, Global DB), Amazon Redshift (MPP, Columnar Storage, Redshift Spectrum).
   - **Bộ nhớ đệm & Di trú:** Amazon ElastiCache (Memcached & Redis), Caching Logic; AWS SCT & AWS DMS.

4. **Thực hành CI/CD trên EKS với CodePipeline và GitHub:**
   - Khởi tạo cụm EKS đơn giản, deploy ứng dụng web.
   - Tự động hóa build, test, deploy với AWS CodePipeline kết nối GitHub repository.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Module 5 Document:** [Module 5](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/module-5.md)
- **Module 6 Document:** [Module 6](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%206/module-6.md)
- **Lab 02 Document:** [Lab 02](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab02.md)
- **Lab 18 Document:** [Lab 18](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab18.md)
- **Lab 30 Document:** [Lab 30](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab30.md)
- **Lab 33 Document:** [Lab 33](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab33.md)
- **Lab 44 Document:** [Lab 44](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab44.md)
- **Lab 48 Document:** [Lab 48](file:///d:/AWS%20FCAJ/aws-fcaj-learning/First%20Cloud%20AI%20Journey/Module%205/Hand-ons/Lab48.md)
- **Worklog Gốc Tuần 3:** [Worklog Tuần 3](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.3-Week3/_index.vi.md)
