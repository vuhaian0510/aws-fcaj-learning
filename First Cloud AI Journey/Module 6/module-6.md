# AWS FCAJ - Module 6: Dịch vụ Cơ sở dữ liệu và Lưu trữ bộ nhớ đệm

### 1. Tổng quan Khái niệm Cơ sở dữ liệu cơ bản

+ Khái niệm chung về Database:
  - Hệ thống thông tin dữ liệu được lưu trữ dưới dạng có cấu trúc hoặc bán cấu trúc trên các thiết bị lưu trữ.
  - Mục tiêu: Thỏa mãn và đáp ứng yêu cầu khai thác thông tin đồng thời của nhiều người dùng, nhiều chương trình hoặc ứng dụng chạy cùng một lúc cho các mục đích khác nhau.
  => Ứng dụng kết nối tới cơ sở dữ liệu sẽ tạo ra một phiên làm việc (**Session**) có thời gian bắt đầu và kết thúc. Hệ thống chỉ hỗ trợ một số lượng phiên nhất định, nếu quá tải sẽ gây ra tình trạng tắc nghẽn.

+ Khóa chính (Primary Key) và Khóa ngoại (Foreign Key):
  - **Primary Key (Khóa chính):** Là một cột đặc biệt (hoặc sự kết hợp của nhiều cột) trong bảng của cơ sở dữ liệu quan hệ, dùng để định danh và xác định duy nhất một bản ghi (Record/Row) trong bảng đó. Giá trị khóa chính phải là duy nhất (Unique) và không được trùng lắp.
  - **Foreign Key (Khóa ngoại):** Là một cột (hoặc nhóm cột) trong một bảng thực hiện tham chiếu chéo đến khóa chính của một bảng khác. Khóa ngoại thiết lập mối liên kết dữ liệu và ràng buộc mối quan hệ giữa hai bảng khác nhau.

+ Chuẩn hóa dữ liệu (Normalization):
  - Khái niệm: Quá trình thiết kế cấu trúc dữ liệu bằng cách chia tách thành nhiều bảng nhỏ hơn liên kết qua khóa chính/khóa ngoại.
  => Mục tiêu: Triệt tiêu tình trạng trùng lặp thông tin, tiết kiệm tối đa dung lượng lưu trữ vật lý và tối ưu hóa tài nguyên tính toán khi vận hành hệ thống.

+ Chỉ mục cơ sở dữ liệu (Database Index):
  - Khái niệm: Một cấu trúc dữ liệu đặc thù được xây dựng nhằm cải thiện tốc độ tìm kiếm và định vị dữ liệu một cách nhanh chóng mà không cần phải quét qua toàn bộ bảng (Table Scan).
  - Bản chất đánh đổi: Index giúp tăng tốc độ đọc dữ liệu (Read), nhưng làm tăng chi phí và thời gian cho các tác vụ ghi dữ liệu (Write) do hệ thống vừa phải cập nhật bảng chính vừa phải duy trì chỉ mục. Đồng thời, Index tiêu tốn thêm không gian lưu trữ riêng.

+ Phân vùng dữ liệu (Partitioning):
  - Khái niệm: Kỹ thuật chia tách một bảng dữ liệu có quy mô cực lớn (hàng triệu/hàng tỷ bản ghi) thành các phần nhỏ hơn gọi là phân vùng (Partition) dựa trên một mã khóa phân vùng (Partition Key).
  => Mục tiêu: Khi thực hiện truy vấn, hệ thống chỉ cần khoanh vùng và quét dữ liệu trên một hoặc một tập con phân vùng cụ thể thay vì quét toàn bộ bảng lớn, giúp cải thiện đáng kể hiệu năng hệ thống.

+ Nhật ký cơ sở dữ liệu (Database Transaction Log):
  - Khái niệm: Hệ thống file ghi lại toàn bộ các biến động, thay đổi trạng thái và các giao dịch xảy ra bên trong cơ sở dữ liệu.
  - Vai trò: Khôi phục lại dữ liệu chính xác đến từng thời điểm sau khi xảy ra lỗi (Point-in-Time Recovery) và đồng bộ hóa dữ liệu giữa cơ sở dữ liệu chính và các hệ thống dự phòng (Standby).
  => Các giao dịch phải được xác nhận hoàn tất (**Commit**) thì mới chính thức ghi vào cơ sở dữ liệu, các bước trung gian trước đó sẽ nằm tại file log.

+ Bộ nhớ đệm dữ liệu (Database Buffer):
  - Khái niệm: Vùng lưu trữ tạm thời nằm trên bộ nhớ RAM chính của máy chủ, lưu trữ bản sao của các khối dữ liệu (Data Blocks) từ ổ đĩa.
  => Mục tiêu: Tốc độ truy xuất của bộ nhớ RAM nhanh hơn rất nhiều so với ổ đĩa cứng, do đó các dữ liệu thường xuyên được truy cập sẽ ưu tiên nằm trên Buffer để tăng tốc độ phản hồi.

---

### 2. Phân loại Hệ quản trị Cơ sở dữ liệu: RDBMS vs NoSQL

+ Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS):
  - Cấu trúc: Dữ liệu tổ chức chặt chẽ dưới dạng các hàng và cột trong các bảng, có sự liên kết và ràng buộc phụ thuộc lẫn nhau giữa các bảng.
  - Ngôn ngữ: Sử dụng ngôn ngữ truy vấn có cấu trúc **SQL** (Structured Query Language).

+ Cơ sở dữ liệu phi quan hệ (NoSQL / Not Only SQL):
  - Cấu trúc: Không sử dụng mô hình bảng cột truyền thống. Dữ liệu được tổ chức linh hoạt tùy loại hình: Document store, Key-value store, Wide-column store, hoặc Graph store.
  - Đặc tính: Tập trung phục vụ cho các bài toán/trường hợp sử dụng cụ thể (Use cases).

+ Bảng so sánh thuộc tính cốt lõi giữa RDBMS và NoSQL:

| Thuộc tính (Attribute) | RDBMS | NoSQL |
| :--- | :--- | :--- |
| **Cấu trúc dữ liệu** <br>(Data Structure) | Sử dụng Hàng và Cột cố định trong các bảng. | Đa dạng hình thức (Key-value, Document, Wide-column, Graph...). |
| **Lượt đồ thiết kế** <br>(Schema) | Cấu trúc lượt đồ tĩnh (**Fixed Schema**), nghiêm ngặt. | Cấu trúc lượt đồ động (**Dynamic Schema**), dễ dàng thêm bớt trường. |
| **Phương thức truy vấn** <br>(Queries) | Sử dụng ngôn ngữ chuẩn **SQL**. | Linh hoạt theo từng Engine cụ thể (Ví dụ: MongoDB, DynamoDB). |
| **Khả năng mở rộng** <br>(Scalability) | Ưu tiên mở rộng theo chiều dọc (**Vertical Scaling** - tăng cấu hình CPU/RAM). | Thiết kế gốc để dễ dàng mở rộng theo chiều ngang (**Horizontal Scaling** - phân mảnh qua Partition). |
| **Triết lý tối ưu** <br>(Optimization) | Tối ưu không gian lưu trữ thông qua chuẩn hóa dữ liệu (**Normalization**). | Chấp nhận trùng lặp dữ liệu và tốn không gian hơn (**Denormalization**) để lấy hiệu năng đọc/ghi. |
| **Mô hình giao dịch** <br>(Transaction Model) | Tuân thủ nghiêm ngặt tính chất **ACID** bảo toàn dữ liệu tuyệt đối. | Theo mô hình **BASE**, ưu tiên tối đa tính sẵn sàng cao (**Availability**) và hiệu năng. |

---

### 3. Mô hình Xử lý dữ liệu: OLTP vs OLAP

+ Hệ thống xử lý giao dịch trực tuyến (OLTP - Online Transaction Processing):
  - Bản chất: Cơ sở dữ liệu chuyên trách lưu trữ và xử lý các giao dịch phát sinh liên tục trong thời gian thực (Ví dụ: hệ thống đặt hàng, thanh toán ngân hàng, giao dịch bán lẻ).
  - Đặc điểm: Tần suất đọc, ghi và cập nhật dữ liệu diễn ra liên tục với khối lượng bản ghi nhỏ trên mỗi giao dịch. Đòi hỏi tốc độ xử lý nhanh và có cơ chế **Rollback** để hoàn trả trạng thái nếu giao dịch thất bại nhằm đảm bảo tính toàn vẹn dữ liệu.

+ Hệ thống xử lý phân tích trực tuyến (OLAP - Online Analytical Processing) / Data Warehouse:
  - Bản chất: Kho dữ liệu (**Data Warehouse**) lưu trữ một khối lượng khổng lồ dữ liệu lịch sử tích lũy qua nhiều năm, được thu gom và tổng hợp từ các nguồn OLTP và nhiều nguồn dữ liệu khác của doanh nghiệp.
  - Đặc điểm: Không phục vụ cho các tác vụ ghi dữ liệu nhỏ lẻ hàng ngày mà chuyên xử lý các câu truy vấn phân tích tổng hợp (**Analytical Queries**) vô cùng phức tạp trên diện rộng.
  => Mục tiêu: Biến dữ liệu thô thành thông tin hữu ích, cung cấp các góc nhìn chuyên sâu (**Business Insights**) phục vụ cho các quyết định chiến lược kinh doanh. Dữ liệu cũ từ OLTP thường được chuyển dịch định kỳ sang OLAP để giải phóng dung lượng cho hệ thống vận hành.

---

### 4. Amazon Relational Database Service (Amazon RDS)

+ Tổng quan dịch vụ Amazon RDS:
  - Khái niệm: Dịch vụ cơ sở dữ liệu quan hệ (RDBMS) được quản lý hoàn toàn (**Managed Service**) bởi AWS.
  - Các công cụ hỗ trợ: Hỗ trợ 6 cơ chế database engine phổ biến bao gồm **MySQL**, **PostgreSQL**, **MariaDB**, **Oracle**, **Microsoft SQL Server** và **Amazon Aurora**.
  - Phân định trách nhiệm: AWS chịu trách nhiệm toàn bộ phần hạ tầng vật lý, cài đặt hệ điều hành, cấu hình platform, tự động vá lỗi phần mềm (**Patching**) và cung cấp công cụ nâng cấp phiên bản. Người dùng (DBA) tập trung tối ưu hóa ứng dụng, thiết kế schema và tinh chỉnh câu lệnh truy vấn.

+ Bản chất kiến trúc hạ tầng:
  - Bản chất bên dưới của một **Amazon RDS** instance thực tế là một máy chủ ảo **Amazon EC2** đã được đội ngũ AWS cấu hình chuyên biệt và khóa lại cho tác vụ cơ sở dữ liệu.
  - Kế thừa bảo mật mạng của EC2: Nằm bên trong mạng ảo **Amazon VPC** (khuyến nghị đặt ở Private Subnet), sử dụng **Security Group** và **Network ACLs** để làm tường lửa kiểm soát truy cập port.
  - Hệ thống lưu trữ: Sử dụng **Amazon EBS** (Elastic Block Store), hỗ trợ tính năng tự động mở rộng dung lượng ổ đĩa (**Storage Auto Scaling**) khi dữ liệu tăng trưởng, giúp tối ưu hóa chi phí.

+ Cơ chế đảm bảo tính sẵn sàng cao (RDS Multi-AZ):
  - Cách thức vận hành: Khi kích hoạt tính năng **Multi-AZ**, AWS sẽ tự động khởi tạo một cơ sở dữ liệu chính (**Primary/Master Instance**) tại một Availability Zone (AZ) và một cơ sở dữ liệu dự phòng (**Standby Instance**) tại một AZ khác trong cùng một Region.
  - Cơ chế đồng bộ: Dữ liệu được đồng bộ liên tục giữa Master và Standby bằng cơ chế đồng bộ thời gian thực (**Synchronous Replication**). Thao tác ghi vào Master chỉ thành công khi Standby xác nhận đã ghi xong.
  - Cơ chế Failover tự động: Nếu Master Instance gặp sự cố, hệ thống RDS sẽ tự động thực hiện chuyển vùng (**Failover**) sang Standby Instance. Ứng dụng kết nối thông qua một **Endpoint** duy nhất nên quá trình Failover diễn ra tự động mà không cần can thiệp mã nguồn hay IP.
  => *Lưu ý:* Multi-AZ làm tăng gấp đôi chi phí tài nguyên, do đó thường chỉ áp dụng cho môi trường Production, không khuyến khích bật cho môi trường Dev/Test.

+ Cơ chế mở rộng hiệu năng đọc (RDS Read Replicas):
  - Cách thức vận hành: Cho phép tạo ra một hoặc nhiều bản sao chỉ đọc (**Read Replicas**) từ cơ sở dữ liệu Master để chia sẻ tải cho các ứng dụng chuyên đọc dữ liệu như hệ thống báo cáo (Reporting), phân tích dữ liệu.
  - Cơ chế đồng bộ: Dữ liệu được đẩy từ Master sang Read Replicas bằng cơ chế bất đồng bộ (**Asynchronous Replication**). Do đó sẽ tồn tại một khoảng trễ thời gian nhỏ (**Replication Lag**) giữa bản chính và bản sao.
  - Ưu điểm: Cơ chế bất đồng bộ giúp Master không phải chờ đợi các bản sao ghi xong, bảo toàn hiệu năng ghi của Master. Khi cần thiết hoặc xảy ra sự cố nghiêm trọng, một Read Replica có thể được tách ra và nâng cấp (**Promote**) thành một cơ sở dữ liệu độc lập (**Stand-alone Primary**) để khắc phục thảm họa (Disaster Recovery).

---

### 5. Amazon Aurora

+ Tổng quan Amazon Aurora:
  - Khái niệm: Hệ quản trị cơ sở dữ liệu quan hệ cao cấp chuẩn Enterprise được AWS thiết kế và tối ưu riêng cho nền tảng điện toán đám mây.
  - Tính tương thích: Hoàn toàn tương thích với hai mã nguồn mở phổ biến là **MySQL** và **PostgreSQL** (**MySQL/PostgreSQL-compatible**), cho phép sử dụng nguyên vẹn các công cụ quản trị truyền thống.

+ Sự khác biệt đột phá về kiến trúc lưu trữ (Aurora Storage Cluster):
  - Khác biệt so với RDS truyền thống: Tách rời hoàn toàn lớp tính toán (**Compute**) và lớp lưu trữ (**Storage**).
  - Cơ chế nhân bản vùng lưu trữ: Dữ liệu của một Aurora Cluster được tự động chia nhỏ và nhân bản thành 6 bản sao (**6 Data Copies**) phân tán đều trên 3 Availability Zones (AZs) khác nhau ở tầng lưu trữ chuyên dụng (**Cluster Volume**).
  => Các bản sao chỉ đọc (**Reader Instances**) và bản ghi (**Writer Instance**) trong cụm Cluster đều nhìn vào một vùng lưu trữ dùng chung này. Do đó, các Reader của Aurora không bị hiện tượng trễ đồng bộ (**Zero Replication Lag**), mang lại hiệu năng xử lý cực kỳ cao (nhanh gấp 3 đến 5 lần so với RDS thông thường).

+ Các tính năng cao cấp cho doanh nghiệp:
  - **Amazon Aurora Backtrack:** Cho phép người dùng "quay ngược thời gian" toàn bộ trạng thái cơ sở dữ liệu về một thời điểm chính xác trong quá khứ chỉ trong vài giây mà không cần phục hồi từ backup.
  - **Aurora Fast Cloning:** Khả năng tạo một bản sao (**Clone**) của cơ sở dữ liệu gốc cực nhanh mà không làm tăng chi phí lưu trữ ban đầu (chỉ tính tiền cho khối dữ liệu có sự thay đổi sau đó).
  - **Aurora Global Database:** Xây dựng một cụm cơ sở dữ liệu quy mô toàn cầu. Một vùng **Primary Region** giữ vai trò ghi dữ liệu (Writer), dữ liệu được replicate xuống tầng lưu trữ sang các **Secondary Regions** với độ trễ dưới 1 giây.
  - **Aurora Multi-Master:** Cho phép cấu hình nhiều máy chủ cùng đóng vai trò ghi dữ liệu (**Multiple Writer Instances**) hoạt động đồng thời trên nhiều AZ khác nhau trong cụm cluster.

---

### 6. Amazon Redshift

+ Tổng quan Amazon Redshift:
  - Khái niệm: Dịch vụ kho dữ liệu (**Data Warehouse**) quy mô lớn (**Massive Scale**) được quản lý hoàn toàn bởi AWS, có khả năng lưu trữ và xử lý phân tích dữ liệu lên tới mức hàng Petabyte (PB).
  - Bản chất công nghệ: Thiết kế ban đầu dựa trên PostgreSQL nhưng được AWS tái cấu trúc và tối ưu hóa triệt để cho các tác vụ xử lý phân tích trực tuyến (**OLAP workloads**).

+ Kiến trúc xử lý song song hàng loạt (MPP - Massively Parallel Processing):
  - Cấu trúc Node: Một cụm **Redshift Cluster** bao gồm:
    * **Leader Node:** Nằm ở trên tiếp nhận các câu lệnh truy vấn SQL từ client (như BI Tools, Amazon QuickSight), thực hiện dịch mã, lập kế hoạch thực thi và điều phối công việc xuống các Compute Nodes.
    * **Compute Nodes:** Đảm nhận việc lưu trữ dữ liệu đã được phân vùng (Partition) và thực hiện tính toán song song độc lập.
  - Phân tách Compute và Storage: Các thế hệ node hiện đại cho phép tách biệt hoàn toàn tài nguyên tính toán và lưu trữ để tối ưu hóa chi phí.

+ Cơ chế lưu trữ theo dạng cột (Columnar Storage):
  - Bản chất: Khác với cơ sở dữ liệu truyền thống lưu trữ dữ liệu theo từng hàng (Row-based), Redshift lưu trữ toàn bộ dữ liệu của cùng một cột nằm kề nhau trên ổ đĩa vật lý.
  => Ưu điểm: Phù hợp hoàn hảo cho các câu lệnh OLAP vốn chỉ cần tính toán trên một vài cột cụ thể của một bảng chứa hàng trăm cột (Ví dụ: tính tổng doanh thu). Hệ thống chỉ cần đọc đúng phân vùng đĩa của cột đó, giảm thiểu tối đa số lượng I/O ổ đĩa.

+ Tính năng tối ưu chi phí và mở rộng:
  - **Redshift Data Sharing:** Khả năng chia sẻ dữ liệu trực tiếp giữa các cụm Redshift độc lập mà không cần thực hiện sao chép hay di chuyển dữ liệu vật lý.
  - **Redshift Spectrum:** Cho phép chạy các câu truy vấn SQL trực tiếp trên khối dữ liệu lịch sử khổng lồ đang được lưu trữ giá rẻ tại **Amazon S3** mà không cần nạp (Load) dữ liệu vào Compute Nodes của Redshift.

---

### 7. Amazon ElastiCache

+ Tổng quan Amazon ElastiCache:
  - Khái niệm: Dịch vụ lưu trữ bộ nhớ đệm (**In-memory Caching**) được quản lý hoàn toàn bởi AWS, hoạt động như một lớp trung gian nằm phía trước hệ thống cơ sở dữ liệu chính.
  - Vai trò: Lưu trữ các dữ liệu tạm thời, kết quả truy vấn phức tạp hoặc dữ liệu thường xuyên truy cập vào bộ nhớ RAM để tăng hiệu năng phản hồi từ mili-giây xuống micro-giây, đồng thời giảm tải áp lực cho database bên dưới.

+ Các công cụ bộ nhớ đệm hỗ trợ (Caching Engines):
  - **Memcached:** Hệ thống lưu trữ key-value đơn giản, thuần túy, thích hợp cho các bài toán cache nhỏ gọn.
  - **Redis:** Hệ thống mạnh mẽ hơn, hỗ trợ nhiều cấu trúc dữ liệu đa dạng (Lists, Sets, Hashes...), hỗ trợ replication, sẵn sàng cao và được hầu hết các ứng dụng hiện đại ưu tiên lựa chọn.
  => **AWS ElastiCache** tự động giám sát, phát hiện và thay thế các Node bị lỗi trong Cluster để duy trì hệ thống ổn định.

+ Trách nhiệm quản lý Logic bộ nhớ đệm (Caching Logic):
  - Phân định phạm vi: AWS quản lý phần hạ tầng của ElastiCache, nhưng Logic của bộ nhớ đệm (**Caching Logic**) hoàn toàn thuộc trách nhiệm của người dùng/lập trình viên.
  - Công việc cụ thể: Người lập trình phải tự định nghĩa logic (ví dụ: kiểm tra cache trước, nếu gặp **Cache Miss** mới truy cập DB và ghi ngược lại vào cache) và tự quản lý cơ chế xóa bỏ cache (**Cache Invalidation**) khi dữ liệu gốc thay đổi.

---

### 8. Giải pháp Di trú Cơ sở dữ liệu (Database Migration)

+ Quy trình dịch chuyển Database lên AWS:
  - Đối với các hệ thống cơ sở dữ liệu đang chạy tại môi trường On-premises (môi trường truyền thống) cần di trú lên đám mây AWS, doanh nghiệp có thể thực hiện dịch chuyển đồng nhất hệ thống hoặc thay đổi hoàn toàn hệ quản trị cơ sở dữ liệu để tối ưu chi phí (Ví dụ: Chuyển đổi từ Oracle Database độc quyền sang Amazon Aurora PostgreSQL mã nguồn mở).

+ Các công cụ hỗ trợ di trú cốt lõi:
  - **AWS Schema Conversion Tool (AWS SCT):** Công cụ chuyên dụng dùng để quét và tự động biên dịch cấu trúc lượt đồ (Schema), các view, các hàm thủ tục (Stored Procedures) của cơ sở dữ liệu gốc sang cấu trúc tương thích với cơ sở dữ liệu đích mới trên AWS khi thực hiện di trú khác hệ sinh thái (**Heterogeneous Migrations**).
  - **AWS Database Migration Service (AWS DMS):** Dịch vụ kết nối và truyền tải trực tiếp dữ liệu vật lý từ nguồn sang đích an toàn. AWS DMS hỗ trợ cơ chế đồng bộ liên tục trong suốt quá trình di trú, giúp hệ thống nguồn vẫn hoạt động bình thường, giảm thiểu tối đa thời gian gián đoạn dịch vụ (**Downtime**).