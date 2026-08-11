# AWS FCAJ - Module 4: Dịch vụ Lưu trữ & Giải pháp Phục hồi sự cố (Storage & DR)

### 1. Amazon Simple Storage Service (Amazon S3)

+ Bản chất lưu trữ hướng đối tượng (Object Storage):
  - Đơn vị lưu trữ cơ bản: Dữ liệu được lưu trữ dưới dạng các đối tượng (Object) độc lập chứa trong các thùng chứa (Bucket).
  - Cơ chế ghi đè (Immutable/Override): S3 khác biệt hoàn toàn so với lưu trữ dạng khối (Block Storage - như EBS hay ổ cứng laptop) ở chỗ không thể chỉnh sửa một phần nhỏ bên trong file (ví dụ: sửa 1 dòng trong file 1000 dòng). Khi có bất kỳ thay đổi nào, người dùng bắt buộc phải tải lên toàn bộ file mới để ghi đè (override) đối tượng cũ.
  - Use Case tối ưu: Cực kỳ phù hợp cho mô hình dữ liệu WORM (Write Once, Read Many) – Ghi một lần, đọc nhiều lần (như file hình ảnh, video, source code tĩnh, tài liệu).

+ Các đặc tính kỹ thuật vượt trội của S3:
  - Dung lượng không giới hạn: S3 không giới hạn tổng dung lượng lưu trữ của một Bucket. Tốc độ AWS bổ sung phần cứng cho S3 luôn nhanh hơn tốc độ upload của người dùng. Tuy nhiên, kích thước tối đa của một đối tượng đơn lẻ (Single Object) giới hạn ở mức 5 TB.
  - Độ bền và Độ sẵn sàng tối đa:
    * Durability (Độ bền dữ liệu): Thiết kế đạt 11 số 9 (99.999999999%), bảo đảm xác suất mất mát dữ liệu gần như bằng 0.
    * High Availability (Độ sẵn sàng): Đạt 4 số 9 (99.99%) về thời gian dịch vụ online (Up-time).
  - Cơ chế nhân bản: Khi upload lên S3, dữ liệu tự động được nhân bản đồng thời trên tối thiểu 3 Availability Zones (AZ) trong một Region (Khác với EBS chỉ nhân bản 3 bản trong cùng 1 AZ).
  - Tính năng S3 Event Trigger: S3 có thể bắt các sự kiện như Put (Upload) hoặc Delete (Xóa) đối tượng để tự động kích hoạt các hàm xử lý không máy chủ (Serverless Function như AWS Lambda).
    => Ví dụ: Tự động resize một ảnh gốc thành nhiều kích thước thumbnail cho mobile và desktop ngay khi ảnh gốc được upload.
  - Cấu trúc dàn hàng ngang (Flat Structure): Trong S3, khái niệm thư mục (Folder/Directory) thực tế chỉ là ảo. Tất cả các đối tượng được xếp ngang hàng trong bucket thông qua chuỗi định danh Object Key. Hệ thống sẽ băm (hash) ký tự tiền tố (prefix) của Object Key để tự động phân vùng (partition) nhằm mở rộng quy mô (Scale out) hệ thống một cách tối ưu.

+ Amazon S3 Access Points:
  - Giải quyết bài toán kiểm soát phức tạp: Trước đây, khi nhiều ứng dụng cùng truy cập một Bucket, việc cấu hình một Bucket Policy duy nhất rất dễ dẫn đến lỗi cấu hình bảo mật.
  - Giải pháp: S3 Access Points cho phép tạo ra nhiều điểm kết nối với các Hostname riêng biệt dành cho từng ứng dụng/phòng ban khác nhau, giúp việc phân chia các chính sách tài nguyên (Resource Policy) trở nên tập trung, an toàn và dễ quản lý.

+ Các lớp lưu trữ (S3 Storage Classes) & Tối ưu chi phí:
  - Phân loại các lớp lưu trữ: AWS phân chia các lớp lưu trữ dựa trên tần suất truy cập dữ liệu để tối ưu hóa giữa dung lượng lưu trữ và chi phí request (Put/Get).
  - Các lớp lưu trữ cụ thể:
    * S3 Standard: Dành cho dữ liệu truy cập thường xuyên. Phí lưu trữ cao, phí request thấp.
    * S3 Standard-Infrequent Access (S3 Standard-IA): Dành cho dữ liệu ít truy cập (nhưng khi cần là phải có ngay). Phí lưu trữ rẻ hơn nhưng phí request rất cao.
      => Nếu đưa dữ liệu nóng vào đây, chi phí tổng sẽ đắt hơn cả lớp Standard.
    * S3 One Zone-IA: Tương tự Standard-IA nhưng dữ liệu chỉ lưu tại 1 AZ duy nhất thay vì 3 AZ. Tiết kiệm chi phí hơn 20%, phù hợp cho dữ liệu ít truy cập và có thể tái tạo lại được.
    * S3 Intelligent-Tiering: Tự động giám sát và luân chuyển đối tượng giữa các lớp lưu trữ (Standard <-> IA) dựa trên hành vi sử dụng (ví dụ: sau 30 ngày không có lượt truy cập, tự động chuyển về lớp IA để giảm phí).
    * S3 Glacier Flexible Archive & Glacier Deep Archive: Các lớp lưu trữ lạnh phục vụ lưu trữ dữ liệu lưu trữ dài hạn (5 - 10 năm) nhằm tuân thủ pháp lý. Chi phí rẻ hơn lên tới 20 lần so với S3 Standard, nhưng không thể truy xuất trực tiếp (xem chi tiết ở phần 3).

+ Quản lý vòng đời đối tượng (S3 Object Lifecycle Management):
  - Tính năng: Cho phép lập cấu hình tự động luân chuyển các lớp lưu trữ của đối tượng theo thời gian.
  - Quy tắc tính thời gian: Số ngày chuyển lớp lưu trữ (ví dụ: sau 30 ngày chuyển sang IA, sau 90 ngày chuyển sang Glacier) bắt buộc phải được tính từ ngày đối tượng được tạo ra ban đầu.
    => Không tính từ ngày đối tượng được di chuyển sang lớp lưu trữ liền trước.

---

### 2. Các tính năng nâng cao và Bảo mật trên S3

+ S3 Static Website Hosting & CORS:
  - Static Website Hosting: S3 có thể hoạt động như một máy chủ web tĩnh, hỗ trợ tối ưu để triển khai các ứng dụng một trang Single Page Application (SPA) sử dụng framework JavaScript hiện đại (React, Vue, Angular).
    => Thay vì tải lại toàn bộ trang từ Web Server truyền thống gây tốn tài nguyên, SPA trên S3 chỉ cập nhật vùng nội dung cần thiết.
  - CORS (Cross-Origin Resource Sharing): Khi host tài nguyên (hình ảnh, file CSS/JS) trên một S3 Bucket mà muốn cho phép các website thuộc Domain khác truy cập và sử dụng, bắt buộc phải bật cấu hình CORS trên Bucket đó.
    => Khai báo rõ Domain nguồn, các phương thức HTTP (Get, Put, Post) và các Header được phép.

+ Cơ chế kiểm soát quyền truy cập (Access Control):
  - Access Control List (ACL): Cơ chế đời đầu (có trước IAM), cho phép gán quyền ở cả cấp Bucket và cấp từng Object con độc lập.
    => Cơ chế này gây phức tạp lớn khi quản trị hệ thống.
  - IAM Policy & Resource Policy: Phương pháp quản trị hiện đại.
    => Người dùng kết hợp gán quyền cho thực thể định danh (Identity Policy - quy định User này được làm gì) và gán quyền trực tiếp tại thùng chứa (Resource Policy / Bucket Policy - quy định ai được phép vào Bucket này) để tạo ra cơ chế phân quyền chặt chẽ.

+ S3 Gateway Endpoints (Kết nối mạng nội bộ):
  - Mặc định S3 nằm ngoài VPC: Nếu một máy chủ EC2 muốn upload dữ liệu lên S3, traffic thông thường sẽ phải đi vòng qua Internet Gateway để ra ngoài Internet công cộng.
  - S3 Gateway Endpoint: Cho phép tạo một điểm kết nối nội bộ ngay trong VPC.
    => Máy chủ EC2 có thể upload/download dữ liệu trực tiếp với S3 thông qua mạng xương sống bảo mật (Backbone Network) của AWS bằng địa chỉ IP Private.
    => Không cần cấp IP Public cho EC2 và không cần đi qua Internet công cộng.

+ S3 Versioning (Quản lý phiên bản) & Chống Ransomware:
  - Hoạt động: Khi bật Versioning, mọi thao tác ghi đè (Put) sẽ tạo ra một Version ID mới, giữ nguyên file gốc.
  - Cơ chế Xóa (Delete): Khi thực hiện lệnh xóa, S3 không xóa thực sự file dữ liệu mà tạo ra một version mới gán nhãn Delete Marker.
    => Nếu người dùng Get thông thường sẽ báo lỗi 404 Not Found.
    => Nếu Get kèm theo Version ID cụ thể của bản ghi cũ, dữ liệu vẫn được khôi phục nguyên vẹn.
  - Phòng chống Ransomware: Khi mã độc tấn công mã hóa ghi đè dữ liệu, hệ thống chỉ ghi đè lên version mới nhất.
    => Doanh nghiệp có thể dễ dàng khôi phục lại phiên bản gốc trước thời điểm bị tấn công.
  - Lưu ý: Quyền Versioning khi đã bật thì không thể disable hoàn toàn mà chỉ có thể tạm dừng (Suspend).
    => Tổng dung lượng của tất cả các phiên bản cộng dồn đều sẽ bị tính phí.

---

### 3. Amazon S3 Glacier (Lưu trữ lạnh và Tuân thủ)

+ Đặc điểm: Quản lý dữ liệu theo các Glacier Vaults và các đối tượng lưu trữ được gọi là các Archives.
+ Hạn chế giao diện: Không hỗ trợ người dùng upload dữ liệu trực tiếp qua giao diện đồ họa AWS Management Console.
  => Bắt buộc phải đẩy dữ liệu vào qua AWS CLI, AWS SDK hoặc thông qua chính sách S3 Lifecycle.
+ Cơ chế khôi phục dữ liệu (Retrieve):
  - Khái niệm: Dữ liệu trong Glacier không thể đọc/tải trực tiếp (Get). Muốn đọc dữ liệu, bắt buộc phải thực hiện lệnh Retrieve để sao chép dữ liệu từ Archive trong Glacier ngược trở lại thành các Object thông thường nằm trên một lớp lưu trữ S3 khác.
  - 3 tùy chọn tốc độ rã đông dữ liệu:
    * Expedited (Tốc độ nhanh): Hoàn tất trong 1 đến 5 phút (phí cao nhất).
    * Standard (Tiêu chuẩn): Hoàn tất trong 3 đến 5 giờ.
    * Bulk (Hàng loạt): Hoàn tất trong 5 đến 12 giờ (chi phí rẻ nhất).
+ Tính năng Vault Lock (Khóa đối tượng phục vụ tuân thủ):
  - Khái niệm: Cho phép áp dụng các chính sách nghiêm ngặt (như cấm xóa dữ liệu trong vòng 5 năm).
  - Vận hành: Sau khi cấu hình, người dùng có 24 giờ thử nghiệm để chỉnh sửa. Qua thời hạn 24 giờ, chính sách sẽ bị khóa vĩnh viễn và áp dụng bắt buộc.
  => Kể cả tài khoản tối cao Root User cũng không có quyền can thiệp hay xóa file cho đến khi chính sách hết hạn (Expire).

---

### 4. AWS Snow Family (Chuyển tải dữ liệu ngoại tuyến - Offline)

+ Khái niệm & Mục đích:
  - Dành cho các dự án chuyển tải dữ liệu quy mô lớn (từ hàng chục Terabyte đến Petabyte) từ môi trường On-premises lên Đám mây.
  - Phù hợp trong điều kiện băng thông Internet tốc độ thấp và không có đường truyền AWS Direct Connect chuyên dụng.
  => Đây gọi là phương thức Cold Data Transfer (Vận chuyển vật lý phần cứng thay vì truyền trực tuyến).

+ Các dòng thiết bị trong gia đình Snow:
  - Snowball: Thiết bị phần cứng lưu trữ vật lý dạng vali chuyên dụng, dung lượng sử dụng thực tế đạt 80 TB.
    => Người dùng đặt hàng thiết bị từ Console -> Thiết bị được ship tới trung tâm dữ liệu.
    => Kỹ sư cài đặt ứng dụng Snowball Client tại môi trường local để nén, mã hóa dữ liệu và copy vào thiết bị, sau đó ship trả lại AWS để import vào S3.
  - Snowball Edge (Snowball x): Bản cải tiến với dung lượng lên tới 100 TB.
    => Điểm khác biệt cốt lõi: Thiết bị này được tích hợp sẵn tài nguyên tính toán (Compute) – tương đương một máy chủ EC2 mini nằm bên trong thiết bị.
    => Cho phép doanh nghiệp thực hiện tiền xử lý dữ liệu, chạy phân tích cục bộ ngay tại hiện trường trước khi gửi dữ liệu về AWS Region.
  - Snowmobile: Giải pháp vận chuyển cấp độ siêu lớn.
    => Là một chiếc xe container chuyên dụng kéo theo hệ thống nguồn và ổ đĩa khổng lồ, hỗ trợ dịch chuyển dữ liệu lên tới hàng trăm Petabyte (PB) hoặc Exabyte (EB).

---

### 5. AWS Storage Gateway (Kiến trúc lưu trữ hỗn hợp - Hybrid Cloud)

+ Khái niệm & Vai trò:
  - Là giải pháp cầu nối, cho phép các ứng dụng tại môi trường On-premises có thể sử dụng dung lượng lưu trữ không giới hạn và độ bền cao của AWS Cloud.
  - Thực hiện thông qua việc cài đặt một thiết bị ảo hóa (Storage Gateway Virtual Appliance) hoặc thiết bị phần cứng tại chỗ.

+ 3 loại cổng kết nối cung cấp:
  - File Gateway:
    * Giao thức hỗ trợ: NFS (cho Linux) và SMB (cho Windows).
    * Cơ chế hoạt động: Chuyển đổi các thư mục chia sẻ cục bộ tại On-premises thành các Object lưu trữ trực tiếp trên Amazon S3.
      => Người dùng có thể xem và truy cập các file này dưới dạng các đối tượng thô ngay trên S3 Console.
  - Volume Gateway:
    * Giao thức hỗ trợ: iSCSI (Lưu trữ dạng khối - Block Storage).
    * Cơ chế hoạt động: Chuyển dữ liệu từ các ổ đĩa của ứng dụng On-premises lên lưu trữ trong S3 dưới dạng các khối block. Dữ liệu này không thể đọc trực tiếp trên S3 mà phải dùng dịch vụ AWS Backup tạo thành các bản EBS Snapshot, từ đó phân tách thành các ổ đĩa EBS gán vào máy chủ EC2 phục vụ cho việc dựng môi trường Disaster Recovery (DR).
    * 2 tùy chọn cấu hình:
      - Stored Volumes: Lưu toàn bộ dữ liệu thô đầy đủ tại On-premises, đồng thời replicate một bản sao dự phòng lên AWS.
        => Phù hợp cho các hệ thống DR cần truy cập dữ liệu cục bộ với độ trễ siêu thấp.
      - Cached Volumes: Chỉ lưu các dữ liệu thường xuyên truy cập (Hot Data) ở bộ nhớ đệm cục bộ On-premises, toàn bộ dữ liệu còn lại được đẩy lên AWS.
        => Nhằm tối ưu hóa dung lượng đĩa cứng tại chỗ.
  - Tape Gateway:
    * Giao thức hỗ trợ: iSCSI thông qua kiến trúc thư viện băng từ ảo VTL (Virtual Tape Library).
    * Mục đích: Thay thế hoàn toàn cho hệ thống đầu đọc băng từ và các tủ băng từ vật lý truyền thống vốn có chi phí vận hành, bảo trì cực kỳ tốn kém.
      => Dữ liệu backup từ các ứng dụng sao lưu (Backup Server) sẽ đổ vào Tape Gateway và được lưu trữ an toàn, tự động tối ưu chi phí trong S3 hoặc S3 Glacier.

---

### 6. Chiến lược khắc phục sự cố sau thảm họa (Disaster Recovery on AWS)

+ Đo lường chỉ số SLA khi thiết kế DR:
  - Nhằm cân đối giữa chi phí hạ tầng đám mây và mức độ chấp nhận rủi ro.
  - RTO (Recovery Time Objective - Thời gian phục hồi mục tiêu): Khoảng thời gian tối đa cho phép hệ thống ngưng hoạt động kể từ khi thảm họa xảy ra cho đến khi phục hồi bình thường.
    => Ví dụ: Sập hệ thống lúc 14:00, RTO = 4 giờ -> Hệ thống phải online trở lại trước 18:00.
  - RPO (Recovery Point Objective - Điểm phục hồi mục tiêu): Khoảng thời gian tối đa chấp nhận mất mát dữ liệu.
    => Ví dụ: Hệ thống backup định kỳ một ngày một lần vào lúc 0h, thảm họa xảy ra lúc 23:00 -> Doanh nghiệp chấp nhận mất tối đa 23 giờ dữ liệu thô chưa kịp backup, RPO = 24 giờ.

+ 4 chiến lược DR cốt lõi trên AWS (Xếp theo thứ tự Chi phí tăng dần / Thời gian phục hồi giảm dần):
  - Backup and Restore (Sao lưu và Khôi phục):
    * Khái niệm: Chiến lược cơ bản và bắt buộc phải có cho mọi mô hình.
    * Vận hành: Dữ liệu được cấu hình chụp snapshot định kỳ lên Cloud. Khi thảm họa xảy ra, kỹ sư tiến hành khởi tạo mới hoàn toàn lại máy chủ và nạp đĩa từ snapshot.
    * Đặc điểm: Chi phí rẻ nhất nhưng RTO và RPO cao nhất (mất nhiều thời gian dựng lại hệ thống).
  - Pilot Light (Mô hình lò sưởi - Active / Standby):
    * Khái niệm: Dữ liệu cốt lõi (như Database) liên tục được đồng bộ thời gian thực từ On-premises lên Cloud.
    * Vận hành: Các máy chủ ứng dụng (EC2) trên Cloud luôn ở trạng thái tắt hoặc chưa được khởi tạo để tiết kiệm tiền. Khi xảy ra thảm họa, hệ thống mới bật các máy chủ EC2 lên và trỏ đĩa vào DB.
    * Đặc điểm: Chi phí rất thấp, RTO ở mức trung bình.
  - Warm Standby (Mô hình chạy tải nhỏ - Low Capacity Active / Active):
    * Khái niệm: Hệ thống trên AWS được xây dựng đầy đủ cấu hình giống hệt On-premises nhưng thu nhỏ quy mô tài nguyên ở mức tối thiểu (ví dụ: On-premises chạy 80% tải, AWS duy trì 20% tải).
    * Vận hành: Hai môi trường hoạt động song song. Khi On-premises sập, Auto Scaling trên Cloud lập tức tự nâng scale out tài nguyên lên tối đa để gánh toàn bộ 100% tải của người dùng.
    * Đặc điểm: RTO và RPO cực thấp.
  - Multi-site Active-Active (Chạy đa điểm toàn tải - Full Capacity):
    * Khái niệm: Hệ thống trên Cloud và On-premises được dựng song song với quy mô tài nguyên full 100% như nhau, chia tải đều thời gian thực.
    * Vận hành: Hệ thống xử lý thảm họa tự động hoàn toàn.
    * Đặc điểm: RTO và RPO tiệm cận mức 0 (hệ thống chuyển đổi không gián đoạn, phù hợp cho Core Banking). Chi phí vận hành cao nhất.

---

### 7. Dịch vụ quản lý sao lưu tập trung: AWS Backup

+ Chức năng:
  - Là dịch vụ quản lý sao lưu tự động và tập trung toàn diện cho các tài nguyên trên AWS (ổ đĩa EBS, máy chủ EC2, hệ thống file EFS, FSx, các cơ sở dữ liệu).
  => Thay thế hoàn toàn cho mô hình máy chủ Backup Server tập trung phức tạp truyền thống.

+ Cơ chế thiết lập:
  - Người dùng xây dựng các Backup Plans để lên lịch trình (Schedule) sao lưu tự động và quản lý chu kỳ sống của bản backup qua cơ chế Retention (Thời gian lưu trữ).

+ Tối ưu chi phí:
  - Cần thiết lập thuộc tính Retention rõ ràng (ví dụ: chỉ giữ lại 30 bản ghi theo cơ chế Incremental).
  => Nếu quên cấu hình vòng đời xóa bản ghi cũ, số lượng snapshot tích lũy vô hạn sẽ dẫn đến việc hóa đơn chi phí lưu trữ tăng cao không kiểm soát.

+ Lưu ý về công cụ bổ sung:
  - Hệ thống cung cấp thêm tính năng AWS Import/Export (VM Import/Export) hỗ trợ đắc lực trong việc di chuyển (import) trực tiếp các file máy ảo từ môi trường ảo hóa nội bộ như VMware, Hyper-V lên thành các AMI trên AWS và ngược lại.