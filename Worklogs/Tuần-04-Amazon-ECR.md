# Minh chứng & Tài liệu Thực hiện - Tuần 4: Amazon ECR & Authentication

### 📌 Thông tin Chung
- **Thời gian:** 06/07/2026 - 10/07/2026
- **Chủ đề chính:** Nghiên cứu Amazon ECR, Docker Authentication qua AWS CLI, Quy trình phân phối ảnh tự động (Image Delivery Flow) & Quét bảo mật container.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Nghiên cứu Amazon ECR & Authentication Mechanism:**
   - Tìm hiểu chuyên sâu dịch vụ Amazon ECR (Elastic Container Registry).
   - Phân biệt Public Repositories (Public Gallery) và Private Repositories.
   - Nghiên cứu phương thức xác thực bảo mật IAM qua lệnh `aws ecr get-login-password` thông qua AWS CLI để tạo Bearer token 12h cho Docker Client.

2. **Phối hợp Xây dựng Quy trình Đóng gói & Phân phối Ảnh:**
   - Thống nhất cấu trúc thư mục Dockerfile và đặt tên Docker registry.
   - Thiết kế sơ đồ luồng đẩy ảnh tự động (Image Delivery Flow):
     `Jenkins CI -> Trivy Security Scan -> Amazon ECR Private Repo`.
   - Xác định các biến môi trường cấu hình: `REGISTRY`, `REPO_NAME`, `IMAGE_NAME`, `IMAGE_TAG`.

3. **Tài liệu hóa Quy trình CI/CD ECR & Dynamic Tagging:**
   - Ghi nhận cấu hình ECR, áp dụng quy tắc đặt tag động theo định dạng Git Commit SHA (`${GIT_COMMIT}`) để chống ghi đè image cũ.
   - Cấu hình tính năng **Scan on Push** trên ECR để tự động phát hiện lỗ hổng bảo mật hệ điều hành container.
   - Viết hướng dẫn cấu hình `ci/Jenkinsfile` và tích hợp ECR Credentials vào Jenkins Credentials Provider.

4. **Chụp Ảnh Minh chứng & Cập nhật Báo cáo:**
   - Chụp ảnh giao diện ECR repository hoạt động thực tế trên AWS Console.
   - Chụp kết quả ECR Scan Findings (Security Vulnerabilities Report).
   - Bổ sung tài liệu vào báo cáo dự án.

5. **Lập Checklist Kiểm thử ECR:**
   - Lập test plan kiểm tra khả năng login, push, pull từ máy cục bộ và Jenkins worker node.
   - Đánh giá tổng kết tuần 4.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Jenkins Pipeline Code:** [Jenkinsfile](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/ci/Jenkinsfile)
- **Worklog Gốc Tuần 4:** [Worklog Tuần 4](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.4-Week4/_index.vi.md)
