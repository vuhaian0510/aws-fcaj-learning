# Minh chứng & Tài liệu Thực hiện - Tuần 5: Amazon ECS Fargate, S3 Reports & Lambda Aggregator

### 📌 Thông tin Chung
- **Thời gian:** 13/07/2026 - 17/07/2026
- **Chủ đề chính:** Khởi tạo hạ tầng Serverless Container **Amazon ECS Fargate** (thay thế EKS để tối ưu chi phí ~$72/tháng), Cấu hình ECS Services & ALB, Tạo Amazon S3 Security Report Bucket & Hàm AWS Lambda Aggregator.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Khởi tạo Amazon ECS Fargate Cluster (Thay thế EKS):**
   - **Lý do thay thế:** EKS control plane tốn phí cố định ~$72/tháng không phù hợp ngân sách sinh viên. ECS Fargate hoạt động theo mô hình Serverless Container (không có phí cluster cố định, chỉ tính tiền khi task thực sự chạy).
   - Khởi tạo ECS Cluster `devsecops-factory` với Fargate Capacity Provider.
   - Cấu hình 2 ECS Services: `tetris-staging` và `tetris-production`.
   - Gắn Application Load Balancer (ALB) tiếp nhận và điều phối lưu lượng web.

2. **Cấu hình ECS Task Definition (`ecs-task-def.json`):**
   - Định nghĩa container web app React Nginx non-root listening tại cổng `8080`.
   - Cấu hình log driver `awslogs` đẩy trực tiếp log container về CloudWatch Log Group `/ecs/tetris-app`.
   - Khai báo IAM Task Execution Role và Task Role chuẩn bảo mật.

3. **Xây dựng S3 Security Report Bucket:**
   - Tạo S3 Bucket tập trung lưu trữ các báo cáo kiểm thử an ninh từ pipeline Jenkins.
   - Phân chia các nhánh thư mục prefix:
     * `reports/secrets/` (Gitleaks scan results)
     * `reports/sca/` (Trivy filesystem scan results)
     * `reports/sast/` (SonarQube / Semgrep scan results)
     * `reports/container/` (Trivy image scan results)
     * `reports/dast/` (OWASP ZAP scan results)
   - Cấu hình mã hóa server-side (SSE-S3), bật Versioning và Lifecycle Policy tự xóa report sau 30 ngày.

4. **Xây dựng AWS Lambda Security Aggregator:**
   - Phát triển hàm Lambda (Python runtime) tự động kích hoạt khi có file JSON report mới upload lên S3 bucket.
   - Lambda tự động parse dữ liệu báo cáo, lọc ra các rủi ro mức High/Critical và xuất log kết quả về CloudWatch Logs.

5. **Quy trình Tối ưu Chi phí (Teardown Strategy):**
   - Thiết lập quy trình scale ECS Fargate services về `0` ngay sau khi kết thúc demo:
     ```bash
     aws ecs update-service --cluster devsecops-factory \
       --service tetris-staging --desired-count 0
     ```

---

### 📊 Kết quả Đạt được & Minh chứng
- **Project Tasks Specification:** [tasks.md](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md)
- **Worklog Gốc Tuần 5:** [Worklog Tuần 5](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.5-Week5/_index.vi.md)
