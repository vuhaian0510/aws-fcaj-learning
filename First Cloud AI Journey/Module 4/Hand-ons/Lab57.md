# Bài Lab 57: Khởi đầu với Amazon S3 và các giải pháp tối ưu hóa Web tĩnh

> **Bối cảnh thực tế:** Amazon S3 là dịch vụ lưu trữ đối tượng cốt lõi của AWS với độ bền thiết kế lên tới 11 số 9 (99.999999999%). Ngoài lưu trữ thô, S3 được ứng dụng rộng rãi để host các website tĩnh hoặc ứng dụng một trang (SPA) nhờ chi phí tối ưu và khả năng tự động mở rộng quy mô.
>
> **Lựa chọn hiện đại:** AWS khuyến nghị sử dụng **AWS Amplify Hosting** để host website tĩnh vì bảo mật tốt hơn, tích hợp sẵn HTTPS mặc định và hệ thống CDN toàn cầu mà không cần cấu hình CloudFront thủ công. Bài lab này tập trung vào các bước thiết lập nền tảng S3, CloudFront và mã hóa/phiên bản để hiểu sâu về cơ chế lưu trữ đám mây.

---

## I. Chuẩn bị tài nguyên (Khởi tạo Bucket và tải dữ liệu)

**Mục tiêu:** Tạo một S3 Bucket với cấu hình Ownership chuẩn hóa và tải các tệp tin mã nguồn của trang web lên hệ thống.

### Các bước thực hiện:
1. Trên **AWS Management Console**, tìm kiếm và truy cập dịch vụ **S3**. Nhấn nút **Create bucket**.
2. Thiết lập thông tin cơ bản:
   - **Bucket name:** Nhập tên duy nhất trên toàn cầu (Ví dụ: `aws-first-cloud-journey-24102004`).
   - **Region:** Lựa chọn Region phù hợp (Ví dụ: `us-east-1`).
3. Tại mục **Object Ownership**, chọn **ACLs disabled (khuyến nghị)** để chủ sở hữu bucket tự động nắm toàn quyền kiểm soát mọi đối tượng, đơn giản hóa công tác quản trị.
4. Tại mục **Block Public Access settings for this bucket**, giữ mặc định tích chọn **Block all public access** để bảo vệ an toàn cho bucket khi vừa khởi tạo. Nhấn **Create bucket**.
5. Click vào tên bucket vừa tạo, chọn **Upload**. Tiến hành kéo thả các file giao diện website tĩnh bao gồm `index.html`, `error.html`, thư mục `css/` và `images/` từ máy tính cục bộ lên S3 rồi nhấn **Upload**.

---

## II. Bật tính năng Static Website Hosting

**Mục tiêu:** Kích hoạt phân hệ biến S3 Bucket thành một điểm cấu hình phân phối nội dung web tĩnh trực tiếp tới trình duyệt của người dùng cuối.

### Các bước thực hiện:
1. Tại giao diện S3 Bucket, chuyển sang tab **Properties**.
2. Kéo xuống cuối trang tìm phân hệ **Static website hosting**, nhấn nút **Edit**.
3. Cấu hình các thông số chi tiết:
   - **Static website hosting:** Chọn **Enable**.
   - **Hosting type:** Chọn **Host a static website**.
   - **Index document:** Điền tên file trang chủ mặc định: `index.html`.
   - **Error document (optional):** Điền tên file hiển thị lỗi: `error.html`.
4. Nhấn **Save changes** để lưu lại cấu hình.

> Hệ thống sẽ ngay lập tức cung cấp một đường dẫn Endpoint dạng DNS tại mục **Static website hosting** (Ví dụ: `http://aws-first-cloud-journey-24102004.s3-website-us-east-1.amazonaws.com`). 
> 
> *Lưu ý: Lúc này nếu click vào link sẽ bị báo lỗi **403 Forbidden** do dữ liệu chưa được mở quyền công khai.*

---

## III. Cấu hình cấp quyền truy cập công khai cho Object (Cơ chế ACL)

**Mục tiêu:** Gỡ bỏ hàng rào chặn public và cấu hình mở quyền công khai cho các object cần thiết để người dùng internet có thể đọc được website.

### Các bước thực hiện:
1. **Gỡ Block Public Access:** Chuyển sang tab **Permissions** tại S3 Bucket. Tại mục *Block public access (bucket settings)*, nhấn **Edit**, bỏ tích chọn mục *Block all public access*, nhấn **Save changes** và gõ chữ `confirm` để xác nhận gỡ bỏ.
2. **Bật cơ chế ACL cấp đối tượng:** Kéo xuống mục **Object Ownership**, nhấn **Edit**, chuyển trạng thái từ *ACLs disabled* sang chọn **ACLs enabled**. Tích chọn ô *I acknowledge that ACLs will be restored* và chọn **Bucket owner preferred**, sau đó nhấn **Save changes**.
3. **Public từng Object:** Quay trở lại tab **Objects**, tích chọn các file và thư mục cần công khai (`index.html`, `error.html`,... ). Nhấn vào menu **Actions** ở góc trên, kéo xuống cuối danh sách và chọn lệnh **Make public using ACL**. Xác nhận hành động.
4. **Kiểm tra (Test Website):** Click lại vào đường dẫn Endpoint DNS nhận được ở mục II để xác nhận website tĩnh đã tải nội dung và hiển thị thành công trên trình duyệt.

---

## IV. Tăng tốc Static Website với Amazon CloudFront (Best Practice)

**Mục tiêu:** Khóa toàn bộ quyền public trực tiếp của S3 để đảm bảo an toàn thông tin, chuyển sang dùng Amazon CloudFront (CDN) làm lớp bảo mật mặt tiền để tăng tốc độ phản hồi và phân phối nội dung toàn cầu.

### Các bước thực hiện:
1. **Khóa lại Public S3:** Vào tab **Permissions** của S3 Bucket, bật lại tính năng **Block all public access** sang trạng thái **On** để cô lập hoàn toàn S3 khỏi internet công cộng.
2. **Khởi tạo CloudFront Distribution:** Trên thanh tìm kiếm AWS Console, truy cập dịch vụ **CloudFront** và nhấn **Create distribution**.
3. **Cấu hình Nguồn cấp (Origin):**
   - **Origin domain:** Click chọn đúng tên S3 Bucket (ví dụ: `aws-first-cloud-journey-24102004.s3.amazonaws.com`) trong danh sách gợi ý.
   - **Origin access:** Tích chọn **Legacy access identities**. Tại ô *Origin Access Identity (OAI)*, nhấn **Create new OAI** để sinh một định danh bảo mật cho lớp kết nối ngầm giữa CloudFront và S3.
   - **Bucket policy:** Tích chọn **Yes, update the bucket policy** để CloudFront tự động nạp cấu hình phân quyền vào S3 Bucket cho bạn.
4. **Cấu hình Cài đặt (Settings):**
   - Tại phân hệ **Web Application Firewall (WAF):** Tích chọn **Do not enable security protections** trong phạm vi bài Lab.
   - Tại mục **Price class:** Lựa chọn **Use North America, Europe, Asia, Middle East, and Africa** để tối ưu vùng phủ sóng.
   - Tại mục **Default root object:** Điền chính xác file trang chủ: `index.html`.
5. Giữ nguyên các thông số mặc định khác và chọn **Create distribution**. 

> Hệ thống sẽ tiến hành deploy mạng lưới CDN toàn cầu và cung cấp cho bạn một tên miền CloudFront Domain Name mới (Ví dụ: `https://d123456abcdef.cloudfront.net`). Sử dụng tên miền này để truy cập website an toàn với tốc độ cao.

---

## V. Kích hoạt tính năng Bucket Versioning

**Mục tiêu:** Kích hoạt tính năng lưu vết đa phiên bản đối tượng để bảo vệ dữ liệu chống lại các hành vi vô tình xóa hoặc sửa đổi ngoài ý muốn.

### Các bước thực hiện:
1. Điều hướng quay trở lại dịch vụ **S3** và click vào bucket của bạn.
2. Chuyển sang tab **Properties**, tìm phân hệ **Bucket Versioning** nằm ngay trên cùng.
3. Nhấn nút **Edit**, chuyển trạng thái từ *Suspend* sang chọn **Enable**.
4. Nhấn **Save changes** để hoàn tất cài đặt lớp bảo vệ dữ liệu bổ sung.

---

## VI. Di chuyển Object và luân chuyển đa vùng (Move & Replication)

**Mục tiêu:** Thực hiện thao tác tổ chức lại cấu trúc lưu trữ và thiết lập nhân bản dữ liệu tự động sang một vùng địa lý khác (Cross-Region Replication) để phục vụ cho các phương án sao lưu thảm họa.

### Các bước thực hiện:
1. **Di chuyển Object (Move Object):** Tại tab **Objects**, tích chọn một file bất kỳ (Ví dụ: `banner.jpg`), nhấn menu **Actions** $\rightarrow$ chọn **Move**. Tiến hành chỉ định một đường dẫn thư mục mới hoặc một bucket đích khác hệ thống và chọn **Move** để dịch chuyển tệp tin.
2. **Nhân bản đa vùng (Multi-Region Replication):** Chuyển sang tab **Management** tại S3 Bucket, kéo xuống tìm phân hệ **Replication rules** và nhấn **Create replication rule**.
3. **Thiết lập quy tắc nhân bản:** Đặt tên cho rule, chọn phạm vi nhân bản (toàn bộ bucket hoặc theo filter prefix), chọn Destination là một S3 Bucket trống nằm ở một Region hoàn toàn khác (Ví dụ: `eu-west-1` - Ireland) và tạo/gán một IAM Role hợp lệ để S3 tự động thực hiện tiến trình đồng bộ ngầm Cross-Region theo thời gian thực.

---

## VII. Dọn dẹp tài nguyên Lab (Clean up)

Để tránh phát sinh chi phí không mong muốn, hãy thực hiện dọn dẹp theo các bước sau:

1. Truy cập vào **CloudFront Console**, tích chọn Distribution đã tạo, nhấn **Disable** và chờ hệ thống chuyển trạng thái, sau đó nhấn **Delete** để xóa bỏ phân phối CDN.
2. Vào **S3 Console**, chọn bucket làm Lab, nhấn nút **Empty** để xóa sạch toàn bộ các object cùng tất cả các phiên bản lịch sử (Version ID) bên trong.
3. Quay trở ra trang danh sách bucket, tích chọn bucket trống đó và thực hiện lệnh **Delete**, gõ chính xác tên bucket để hủy bỏ hoàn toàn tài nguyên, tránh phát sinh chi phí.