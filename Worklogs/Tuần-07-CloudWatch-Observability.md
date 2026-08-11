# Minh chứng & Tài liệu Thực hiện - Tuần 7: CloudWatch Observability cho ECS Fargate & AWS Budgets

### 📌 Thông tin Chung
- **Thời gian:** 27/07/2026 - 31/07/2026
- **Chủ đề chính:** CloudWatch Container Insights cho ECS Fargate, Centralized Logging với log driver `awslogs`, CloudWatch Logs Insights Queries, Cảnh báo Chi phí AWS Budgets.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Nghiên cứu CloudWatch Observability cho ECS Fargate:**
   - Nghiên cứu cơ chế thu thập metrics/logs của CloudWatch Container Insights dành cho ECS Cluster.
   - Cấu hình driver `awslogs` trong `ecs-task-def.json` để đẩy toàn bộ stdout/stderr của container Nginx web app về CloudWatch Log Groups.

2. **Kích hoạt & Cấu hình Container Insights:**
   - Chạy lệnh kích hoạt Container Insights cho ECS cluster:
     ```bash
     aws ecs update-cluster-settings \
       --cluster devsecops-factory \
       --settings name=containerInsights,value=enabled \
       --region ap-southeast-1
     ```
   - Xác nhận các Log Groups liên quan đến ECS task, service và application container bắt đầu đổ về AWS CloudWatch console.

3. **Xây dựng Log Queries & Dashboard:**
   - Biên soạn các câu lệnh truy vấn CloudWatch Logs Insights để lọc nhanh log ứng dụng web (Lỗi HTTP 5xx, Exception traces, DAST scan patterns).
   - Dựng Dashboard giám sát tài nguyên CPU/RAM và Network I/O trực quan cho ECS Tasks.

4. **Chụp Minh chứng Observability & AWS Budgets:**
   - Chụp giao diện Dashboard Container Insights và Log Streams.
   - Chụp ảnh cấu hình cảnh báo chi phí AWS Budgets tại các ngưỡng 50%, 80%, 100%.

5. **Hoàn thiện Báo cáo Observability & Kế hoạch Dọn dẹp:**
   - Viết phần Observability trong báo cáo tổng kết.
   - Xây dựng quy trình tối ưu và dọn dẹp tài nguyên AWS (scale ECS Fargate service `desired-count` về 0) sau khi kết thúc dự án.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Project Tasks Specification:** [tasks.md](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md)
- **Worklog Gốc Tuần 7:** [Worklog Tuần 7](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.7-Week7/_index.vi.md)
