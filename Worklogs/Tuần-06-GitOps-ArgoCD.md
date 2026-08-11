# Minh chứng & Tài liệu Thực hiện - Tuần 6: GitOps Continuous Delivery với Argo CD

### 📌 Thông tin Chung
- **Thời gian:** 20/07/2026 - 24/07/2026
- **Chủ đề chính:** Nguyên lý GitOps, Triển khai Argo CD, Quy trình Staging Auto-Sync, Production Manual Approval Gate, Quản lý Git Credentials.
- **Thành viên thực hiện:** AWS FCAJ Team Member

---

### 🎯 Các Công việc đã Triển khai Chi tiết

1. **Nghiên cứu Nguyên lý GitOps & Argo CD:**
   - Tìm hiểu mô hình chuyển giao phần mềm kéo (Pull-based deployment) qua GitOps.
   - So sánh ưu thế của GitOps so với CD truyền thống (ngăn chặn Configuration Drift).
   - Nghiên cứu cấu trúc khai báo Argo CD `Application` Custom Resource Definition (CRD).

2. **Thiết kế Kiểm thử Staging Auto-Sync:**
   - Thiết kế test case kiểm thử quy trình tự động đồng bộ từ Git manifest repo lên EKS staging namespace ngay khi Jenkins đẩy tag Commit SHA mới.
   - Xác định tiêu chuẩn đánh giá ứng dụng: `Synced` (manifest khớp Git) và `Healthy` (Pods sẵn sàng).

3. **Xây dựng Quy trình Manual Approval Gate cho Production:**
   - Soạn thảo tài liệu quy trình phê duyệt thủ công trong Jenkins pipeline trước khi đẩy tag sang nhánh production.
   - Đảm bảo kiểm soát an toàn chất lượng mã nguồn trước khi chuyển giao sản phẩm lên Production.

4. **Chụp Minh chứng GitOps & Bảo mật Credentials:**
   - Chụp ảnh minh chứng giao diện Argo CD hiển thị 2 ứng dụng Staging và Production ở trạng thái Synced & Healthy.
   - Kiểm tra mã hóa và bảo mật Git credentials (SSH Key / Personal Access Token) trong Jenkins Credentials Provider.

5. **Tổng hợp & Đánh giá Tuần 6:**
   - Tổng hợp danh mục test case hoàn chỉnh cho quy trình GitOps.
   - Tổng kết tiến độ tuần 6.

---

### 📊 Kết quả Đạt được & Minh chứng
- **Worklog Gốc Tuần 6:** [Worklog Tuần 6](file:///d:/AWS%20FCAJ/fcj-workshop-template/content/1-Worklog/1.6-Week6/_index.vi.md)
