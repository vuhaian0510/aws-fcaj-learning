# Minh chứng & Tài liệu Thực hiện - Tuần 9: Hugo Workshop Website & Cloud Teardown

### 📌 Thông tin Chung
- **Thời gian:** 10/08/2026 - 14/08/2026
- **Chủ đề chính:** Xây dựng Workshop Website trên Hugo framework (`FCAJ-workshop-template`), Tích hợp 100% nội dung dự án, Rà soát song ngữ (VI/EN), Build Production (`hugo --minify`), Bàn giao đồ án & Dọn dẹp tài nguyên AWS ECS Fargate, ALB, ECR, S3 (Cloud Teardown).
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Khởi tạo Hugo Website & Menu Navigation:**
   - Cấu hình `config.toml` và trọng số weight cho thanh điều hướng menu (Info, Worklog, Proposal, Blogs, Events, Workshop, Self-evaluation, Feedback).

2. **Tích hợp Nội dung Dự án lên Website:**
   - Đưa thông tin sinh viên, proposal, worklog 9 tuần, 3 bài blogs kỹ thuật, sự kiện ngoại khóa, workshop labs và trang tự đánh giá/phản hồi lên `content/`.
   - Nhúng sơ đồ kiến trúc hệ thống Serverless DevSecOps và sơ đồ CI/CD chất lượng cao.

3. **Rà soát Song ngữ & Kiểm thử UI/Links:**
   - So khớp chéo bản dịch tiếng Việt và tiếng Anh trên tất cả các trang.
   - Rà soát hyperlinks, tables và responsive layout trên mobile/desktop.

4. **Build Production & Nộp Báo cáo:**
   - Chạy lệnh `hugo --minify` thành công, kiểm tra zero broken links.
   - Đóng gói và bàn giao đồ án đúng thời hạn (14/08/2026).

5. **Dọn dẹp Tài nguyên Đám mây AWS (Teardown):**
   - Scale ECS Fargate services về `0`:
     ```bash
     aws ecs update-service --cluster devsecops-factory \
       --service tetris-staging --desired-count 0
     aws ecs update-service --cluster devsecops-factory \
       --service tetris-production --desired-count 0
     ```
   - Xóa Load Balancers (ALB), ECR repositories, S3 buckets để dừng triệt để chi phí.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Project Tasks Specification:** [tasks.md](file:///d:/AWS%20FCAJ/CICD-DevSecOps-using-AWS-services/tasks.md)
- **Hugo Site Repository:** [fcj-workshop-template](file:///d:/AWS%20FCAJ/fcj-workshop-template/)
- **Worklog Gốc Tuần 9:** [Worklog Tuần 9](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.9-Week9/_index.vi.md)
