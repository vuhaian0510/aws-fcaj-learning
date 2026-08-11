# AWS FCAJ - Module 1: Điện toán đám mây & AWS Cloud Basics

### 1. Điện toán đám mây (Cloud Computing) là gì?

+ Khái niệm: Cung cấp tài nguyên IT theo nhu cầu (on-demand) qua Internet. Thanh toán theo lượng sử dụng thực tế (pay-as-you-go).

- So sánh mô hình hạ tầng:
  * Truyền thống (On-premises):
    - Tốn chi phí lớn ban đầu (CapEx) mua sắm, lắp đặt máy chủ, tủ rack, điều hòa...
    - Mất thời gian tự vận hành, bảo trì, sửa chữa phần cứng.
    - Phải dự báo trước nhu cầu hạ tầng từ 3-5 năm (dễ thừa hoặc thiếu).
  * Đám mây (Cloud):
    - Không cần quan tâm phần cứng vật lý.
    - Cần tài nguyên => Tạo yêu cầu => Hệ thống tự khởi tạo máy chủ ảo/dịch vụ trong vài phút.

+ 4 Lợi ích cốt lõi:
  - Tối ưu chi phí: Không tốn chi phí đầu tư ban đầu. Chỉ trả tiền cho tài nguyên thực tế chạy (tắt đi không tính tiền).
  - Tốc độ & Linh hoạt: Triển khai ứng dụng nhanh nhờ các dịch vụ được quản lý sẵn (Managed Services) và tự động hóa.
  - Khả năng co giãn (Elasticity): Tăng/giảm CPU, RAM, dung lượng lưu trữ nhanh chóng theo nhu cầu thời thực.
  - Quy mô toàn cầu: Đưa ứng dụng đến nhiều khu vực trên thế giới chỉ trong vài click để giảm độ trễ cho người dùng cuối.

---

### 2. Sự khác biệt của AWS (Amazon Web Services)

+ Dẫn đầu tuyệt đối về hạ tầng:
  - 13 năm liên tiếp dẫn đầu thị trường Cloud (theo Gartner).
  - Hạ tầng phân cấp tối ưu: Data Center (Phần cứng custom) -> Availability Zone (AZ - Cô lập lỗi) -> Region (Vùng địa lý chứa >= 3 AZ) -> Edge Location (Mạng lưới biên tối ưu CloudFront CDN, WAF, Route 53).

+ Triết lý giá độc đáo - Lợi thế quy mô (Economies of Scale):
  - Cam kết giảm giá và đã giảm hơn 120 lần kể từ khi thành lập.
  - Quy luật hoạt động:
    Khách hàng tăng => Quy mô AWS tăng => Chi phí vận hành trên mỗi đơn vị giảm => AWS giảm giá dịch vụ cho khách hàng.
  => AWS không dùng bẫy giá (thu hút ban đầu rồi tăng giá sau).

+ Văn hóa vận hành (Leadership Principles):
  - Customer Obsession (Cuồng khách hàng): Kỹ sư/tư vấn AWS tập trung tối ưu kiến trúc và giảm chi phí thừa cho khách hàng, không chạy theo doanh số.
  - Ownership (Tinh thần làm chủ): Tư duy dài hạn, xây dựng hệ thống bền vững thay vì lợi ích ngắn hạn.
  - Deliver Results (Đạt kết quả): Tập trung mang lại giá trị thực tế cuối cùng cho khách hàng.

---

### 3. Lộ trình & Kinh nghiệm bắt đầu với AWS

+ Hệ sinh thái AWS cực lớn: Tài liệu phong phú, nhiều khóa học chất lượng từ chính hãng và cộng đồng.
+ Sự chuyển dịch tư duy:
  - Truyền thống: Chuyên môn hóa sâu (chỉ làm database, chỉ làm network...).
  - Cloud: Đòi hỏi tư duy tổng thể (Full-stack infrastructure), kết hợp đan xen hàng nghìn dịch vụ.
+ Cách học hiệu quả:
  - Tham gia cộng đồng (như AWS Study Group) để cùng thảo luận, gỡ lỗi.
  - Thực hành thực tế: Tự tạo tài khoản cá nhân, tận dụng AWS Free Tier (1 năm đầu) để tự hands-on.
  => Không học lý thuyết suông hay làm lab giả lập.
+ Điểm khác biệt của First Cloud Journey:
  - Không đi theo lối mòn slide lý thuyết hoặc các bài step-by-step nhàm chán.
  - Tập trung hướng dẫn học viên tự xây dựng Side Projects và tham gia Workshop thực tế để tạo CV thực chiến.

---

### 4. Hạ tầng toàn cầu của AWS (Global Infrastructure)

+ Data Center (Trung tâm dữ liệu):
  - Chứa hàng chục nghìn máy chủ vật lý.
  - Sử dụng linh kiện phần cứng tùy biến (customized) riêng để đạt hiệu năng tối đa.

+ Availability Zone (AZ - Vùng sẵn sàng):
  - Gồm 1 hoặc nhiều Data Center kết nối với nhau qua mạng băng thông lớn, độ trễ cực thấp.
  - Cô lập lỗi (Fault Isolation): Mỗi AZ có nguồn điện, mạng độc lập. AZ này sập không ảnh hưởng AZ khác.
  => Khuyến nghị: Luôn chạy ứng dụng tối thiểu trên 2 AZ để đảm bảo tính sẵn sàng cao (High Availability).

+ Region (Vùng địa lý):
  - Là một khu vực địa lý cụ thể trên thế giới (VD: Singapore, Tokyo), chứa tối thiểu 3 AZ độc lập.
  - Dữ liệu nằm trong Region do người dùng chọn và không tự động chuyển đi nơi khác (đảm bảo bảo mật & luật pháp).
  - Yếu tố chọn Region: Vị trí gần người dùng cuối (giảm lag) + Tính sẵn có của dịch vụ + Chi phí.

+ Edge Location (Điểm biên):
  - Trạm trung chuyển dữ liệu trung gian, phân bố dày đặc toàn cầu.
  - Phục vụ các dịch vụ biên: Amazon CloudFront (CDN), WAF (Tường lửa), Route 53 (DNS) để tăng tốc và bảo mật.

---

### 5. Công cụ quản lý & Tương tác với AWS

+ AWS Management Console:
  - Giao diện Web trực quan, thích hợp làm quen và cấu hình thủ công.
  - Phân quyền sử dụng:
    * Root User: Tài khoản tối cao (tạo khi đăng ký), nắm mọi quyền hành. Cần bật MFA và cất đi, tránh dùng hàng ngày.
    * IAM User: Tài khoản con do Root tạo ra, phân quyền tối thiểu (Least Privilege) cho từng nhân sự/ứng dụng.

+ AWS CLI (Command Line Interface):
  - Công cụ dòng lệnh tương tác trực tiếp qua Terminal/cmd.
  - Dùng cặp Access Key & Secret Access Key để xác thực quyền truy cập.

+ AWS SDK (Software Development Kit):
  - Thư viện lập trình dành cho các ngôn ngữ (Python, JS, Java...).
  - Dùng để viết code tương tác, điều khiển các dịch vụ AWS tự động bên trong ứng dụng.

---

### 6. Tối ưu chi phí & Các gói AWS Support

+ Chiến lược tối ưu chi phí:
  - Right-sizing: Đo lường chính xác cấu hình cần thiết, tránh bê nguyên xi hạ tầng dư thừa từ On-prem lên Cloud.
  - Các mô hình mua tài nguyên (EC2):
    * On-Demand: Trả tiền theo giờ/giây, linh hoạt nhất nhưng đắt nhất.
    * Reserved Instances (RI) / Savings Plans: Cam kết dùng 1-3 năm để được giảm giá sâu (lên đến 72%).
    * Spot Instances: Đấu giá tài nguyên dư thừa của AWS, rẻ hơn đến 90%, nhưng AWS có thể thu hồi bất cứ lúc nào (hợp với tác vụ chạy ngắt quãng, xử lý dữ liệu hàng loạt).
  - Tự động hóa: Thiết lập lịch tự động tắt các môi trường không dùng (như dev/test) ngoài giờ làm việc.
  - Serverless: Chuyển sang kiến trúc không máy chủ (Lambda, S3...) để chỉ trả tiền khi code thực thi.
  - Giám sát ngân sách: Cấu hình AWS Budgets để cảnh báo khi chi phí vượt ngưỡng, kết hợp Cost Allocation Tags để theo dõi chi phí theo dự án/phòng ban.

+ Các gói AWS Support:
  - Basic: Miễn phí, mặc định cho mọi tài khoản. Chỉ được xem tài liệu & hỗ trợ về billing/account.
  - Developer: Trả phí thấp, phù hợp môi trường thử nghiệm/phát triển. Hỗ trợ qua email trong giờ hành chính.
  - Business: Khuyến nghị cho hệ thống chạy Production. Hỗ trợ kỹ thuật 24/7 qua chat/phone, cam kết phản hồi nhanh.
  - Enterprise: Cao cấp nhất, có Kỹ sư hỗ trợ riêng (TAM - Technical Account Manager), cam kết phản hồi sự cố khẩn cấp trong vòng 15 phút.