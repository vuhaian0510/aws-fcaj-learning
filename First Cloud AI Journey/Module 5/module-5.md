# AWS FCAJ - Module 5: Dịch vụ Bảo mật, Quản trị định danh và Mã hóa dữ liệu

### 1. Tổng quan Triết lý Bảo mật và Mô hình Chia sẻ Trách nhiệm

+ Triết lý "Security is Job Zero":
  - Bảo mật là ưu tiên hàng đầu và là yếu tố cốt lõi nhất được đặt lên trước hết khi xây dựng sản phẩm hoặc dịch vụ trên nền tảng AWS.
  => Mọi nỗ lực triển khai công nghệ sẽ trở nên vô nghĩa nếu hệ thống không đảm bảo được các yêu cầu an toàn thông tin cơ bản.

+ Mô hình chia sẻ trách nhiệm (Shared Responsibility Model):
  - Khái niệm: Bảo mật trên điện toán đám mây là trách nhiệm chung giữa AWS và khách hàng, được phân định rõ ràng thành hai phạm vi:
    * Security of the Cloud (Bảo mật của Đám mây) - Trách nhiệm của AWS: AWS chịu trách nhiệm bảo vệ hạ tầng vật lý toàn cầu, bao gồm các trung tâm dữ liệu, mạng lưới phần cứng, các phân lớp Region, Availability Zone (AZ), Edge Location và lớp ảo hóa hypervisor bên dưới.
    * Security in the Cloud (Bảo mật trong Đám mây) - Trách nhiệm của Khách hàng: Khách hàng chịu trách nhiệm cấu hình an toàn cho phần tài nguyên mình khởi tạo.
      => Phạm vi này bao gồm: cập nhật bản vá hệ điều hành, cấu hình tường lửa mạng (Security Group/NACL), mã hóa dữ liệu ở trạng thái nghỉ và trạng thái truyền tải, quản lý ứng dụng, và quản trị quyền truy cập/định danh (IAM).

+ Sự thay đổi trách nhiệm theo loại hình dịch vụ:
  - Khái niệm: Mức độ quản trị hạ tầng của khách hàng thay đổi linh hoạt tùy theo mô hình dịch vụ lựa chọn:
    * Dịch vụ mức hạ tầng (Infrastructure Services - như EC2): Khách hàng gánh vác nhiều trách nhiệm nhất, từ quản lý hệ điều hành, cấu hình mạng, tường lửa đến cập nhật ứng dụng và mã hóa.
    * Dịch vụ quản lý kết hợp (Containerized/Semi-managed Services - như RDS): AWS thay mặt khách hàng quản lý lớp hệ điều hành và nền tảng (platform), khách hàng chỉ tập trung tối ưu hóa tầng ứng dụng và câu lệnh cấu hình.
    * Dịch vụ quản lý hoàn toàn (Fully Managed Services - như S3): Khách hàng bớt tối đa công sức quản lý hạ tầng, chỉ cần cấu hình chính sách bảo mật dữ liệu, bật versioning hoặc mã hóa theo nhu cầu riêng.

---

### 2. AWS Identity and Access Management (AWS IAM)

+ Quản trị Tài khoản Gốc (Root Account Best Practices):
  - Đặc quyền tối cao: Root Account được tạo ra ban đầu bằng email đăng ký, nắm toàn quyền kiểm soát không giới hạn đối với mọi tài nguyên và thông tin tài chính, đồng thời có khả năng gỡ bỏ mọi chính sách chặn quyền.
  - Nguyên tắc thực hành tốt nhất:
    * Tuyệt đối không sử dụng Root Account cho các tác vụ quản trị hoặc thao tác vận hành hàng ngày.
    * Bắt buộc phải khóa thông tin xác thực của Root User và kích hoạt bảo mật hai lớp (MFA) bằng phần cứng hoặc ứng dụng ảo.
    * Trong môi trường doanh nghiệp: Cần chia đôi thông tin password và thiết bị nhận mã MFA cho hai nhân sự cấp cao khác nhau nắm giữ.
    * Đảm bảo duy trì và gia hạn liên tục quyền sở hữu tên miền (Domain) của email dùng làm Root User để tránh nguy cơ bị kẻ tấn công chiếm quyền domain và tranh chấp tài khoản.

+ Các thành phần định danh trong IAM:
  - IAM Principal (Chủ thể): Là bất kỳ thực thể nào có thể gửi yêu cầu API để thực hiện hành động trên tài nguyên AWS (bao gồm Root User, IAM User, Federated User, IAM Role).
  - IAM User: Tài khoản người dùng con được tạo ra bên trong AWS account, sở hữu thông tin xác thực riêng biệt gồm Mật khẩu (truy cập Management Console) hoặc cặp mã khóa Access Key / Secret Access Key (để gọi API qua CLI/SDK).
    => Mặc định khi vừa tạo mới, IAM User không có bất kỳ quyền hạn nào.
  - IAM Group: Tập hợp dùng để gom nhóm nhiều IAM User nhằm mục đích quản lý và áp dụng chung một chính sách quyền hạn đồng nhất, giúp tối ưu hóa công tác quản trị.
    => IAM Group không thể chứa các group con khác (không hỗ trợ lồng nhóm).

+ Cơ chế chính sách quyền hạn (IAM Policies):
  - Khái niệm: Chính sách quyền hạn được định nghĩa bằng cấu trúc file JSON và chia làm ba loại chính:
    * AWS Managed Policy: Các tập chính sách quyền hạn được AWS xây dựng sẵn cho các vai trò phổ biến (như AdministratorAccess). Các chính sách này thường có phạm vi quyền rất rộng và ít ràng buộc.
    * Customer Managed Policy: Các chính sách do khách hàng tự biên soạn nhằm thắt chặt quyền hạn chi tiết đến từng tài nguyên, thẻ định danh hoặc điều kiện môi trường cụ thể.
    * Inline Policy: Chính sách đặc thù được nhúng trực tiếp và duy nhất vào một IAM User, không có khả năng tái sử dụng cho các chủ thể khác.
  - Nguyên tắc kiểm tra quyền:
    * Hệ thống luôn áp dụng nguyên tắc Cấp quyền tối thiểu (Least Privilege).
    * Mặc định mọi yêu cầu bị từ chối.
    => Khi đánh giá quyền, nếu có một câu lệnh từ chối tường minh (Explicit Deny), nó sẽ phủ quyết và override hoàn toàn mọi câu lệnh cho phép (Allow) khác.

+ IAM Role và Dịch vụ AWS STS:
  - Khái niệm: IAM Role là một tập hợp các quyền truy cập tài nguyên nhưng không sở hữu thông tin chứng thực cố định (không có password hay access key lâu dài).
  - Cơ chế hoạt động: Khi một chủ thể (User hoặc dịch vụ AWS) muốn sử dụng IAM Role, họ phải thực hiện hành động đảm nhận vai trò (Assume Role) thông qua dịch vụ AWS Security Token Service (STS).
    * Hệ thống sẽ kiểm tra chính sách tin cậy (Trust Policy) của Role xem chủ thể đó có được phép hay không.
    * Nếu hợp lệ sẽ cấp một chuỗi thông tin chứng thực bảo mật tạm thời (Temporary Credentials).
    => Khi Assume Role thành công, quyền hạn mới của Role sẽ thay thế hoàn toàn quyền hạn cũ của User.
  - Ứng dụng cho Dịch vụ (EC2 Instance Profile):
    * Đối với các ứng dụng chạy bên trong máy chủ ảo EC2 cần gọi tài nguyên AWS (như ghi file lên S3), phương pháp thực hành tốt nhất là gắn một IAM Role (Instance Profile) cho EC2 thay vì nhúng mã Access Key cố định của IAM User vào source code ứng dụng.
    => Giúp triệt tiêu hoàn toàn rủi ro bị lộ thông tin bí mật khi đẩy code lên các nền tảng công khai như GitHub, tránh việc bị các bot tự động của hacker quét và lợi dụng tài nguyên để trục lợi.
    => AWS SDK sẽ tự động chịu trách nhiệm luân chuyển (rotate) và gia hạn mã khóa tạm thời này.

---

### 3. Amazon Cognito

+ Tổng quan Amazon Cognito:
  - Khái niệm: Là dịch vụ quản lý định danh, xác thực và phân quyền dành riêng cho các nhà phát triển ứng dụng di động và ứng dụng web theo định hướng đám mây (Cloud-native).
  - Thành phần: Bao gồm hai thành phần cốt lõi hoạt động độc lập hoặc kết hợp:
    * User Pools (Thư mục người dùng):
      - Vai trò: Đóng vai trò là một thư mục lưu trữ thông tin tài khoản người dùng trực tiếp.
      - Tính năng: Cung cấp sẵn các module tính năng như form đăng ký, đăng nhập, khôi phục mật khẩu, xác thực hai lớp (MFA) mà lập trình viên không cần tự xây dựng từ đầu.
      - Xác thực liên kết (Federated Authentication): Cho phép người dùng đăng nhập ứng dụng thông qua tài khoản của các bên thứ ba như mạng xã hội (Google, Facebook, Amazon, Apple) hoặc các hệ thống định danh doanh nghiệp chuẩn mã nguồn mở như SAML hay OIDC.
      => Đầu ra của quá trình xác thực thành công tại User Pools là một chuỗi mã xác thực (Token) chứng minh người dùng hợp lệ.
    * Identity Pools (Kho cấp quyền tài nguyên):
      - Vai trò: Đóng vai trò tiếp nhận chuỗi Token chứng thực (từ Cognito User Pools hoặc từ các nhà cung cấp định danh bên thứ ba) để tiến hành ánh xạ (mapping) người dùng đó với một IAM Role cụ thể trên AWS.
      - Ứng dụng thực tế: Cho phép phân quyền trực tiếp người dùng cuối của ứng dụng di động tiếp cận tài nguyên AWS một cách chặt chẽ.
        => Ví dụ: Ánh xạ tài khoản người dùng "Free" vào một IAM Role chỉ được đọc ảnh chất lượng thấp từ một S3 Bucket nhất định, và ánh xạ tài khoản "Premium/VIP" vào một IAM Role có quyền truy cập đĩa lưu trữ chứa ảnh chất lượng cao 4K.

---

### 4. AWS Organizations

+ Tổng quan AWS Organizations:
  - Khái niệm: Dịch vụ quản trị và điều hành tập trung cho các doanh nghiệp sở hữu môi trường điện toán quy mô lớn bao gồm nhiều tài khoản AWS khác nhau.
  => Nhằm thu hẹp phạm vi ảnh hưởng khi xảy ra sự cố kỹ thuật (Blast Radius).

+ Cấu trúc phân cấp:
  - Hệ thống bao gồm một tài khoản quản trị tối cao (Management Account / Master Account) điều hành toàn bộ cấu trúc cây thư mục.
  - Bên dưới phân chia thành các đơn vị tổ chức lồng nhau (Organizational Units - OUs), trong các OUs chứa các tài khoản thành viên (Member Accounts).

+ Consolidated Billing (Thanh toán tập trung):
  - Bản chất: Toàn bộ chi phí tiêu hao tài nguyên của tất cả các member account trong tổ chức sẽ được gom tụ lại.
  => Xuất hóa đơn tổng về một tài khoản Master duy nhất để quản lý dòng tiền.

+ Service Control Policies (SCP):
  - Khái niệm: Là các chính sách quản trị tối cao được gán từ tài khoản Master xuống các nhánh OU hoặc các member account chỉ định.
  - Bản chất của SCP: SCP không dùng để cấp quyền mà đóng vai trò là một Rào cản giới hạn quyền tối đa (Permission Boundary) cho tất cả các user và role nằm bên trong account chịu ảnh hưởng.
  => Lưu ý: Nếu một hành động bị chặn bởi SCP (Ví dụ: SCP cấu hình chỉ cho phép dùng dịch vụ S3), thì bất kỳ user nào trong member account đó – kể cả tài khoản Root User của member account đó – cũng không thể thực hiện hành động nào khác ngoài S3, và không có quyền override chính sách này.

---

### 5. AWS Identity Center

+ Tổng quan AWS Identity Center (trước đây gọi là AWS Single Sign-On / AWS SSO):
  - Khái niệm: Giải pháp quản lý truy cập một lần tập trung cho toàn bộ hệ thống AWS Organizations và các ứng dụng doanh nghiệp bên ngoài.
  - Nguồn quản lý định danh đa dạng: Identity Center có thể tự quản lý danh sách user nội bộ hoặc kết nối liên kết trực tiếp với các kho dữ liệu doanh nghiệp sẵn có thông qua dịch vụ Microsoft Active Directory (như On-premises AD, AWS Managed Microsoft AD hoặc Azure AD qua ID Connector).

+ Cơ chế thiết lập 3 bước:
  - Bước 1 - Khởi tạo định danh: Tạo lập danh sách tài khoản người dùng và phân chia thành các nhóm người dùng (User Groups).
  - Bước 2 - Cấu hình tập hợp quyền (Permission Sets): Xây dựng các bộ khung quyền hạn tập trung (gắn các IAM Policy tương ứng) lưu trữ tại Identity Center.
    => Bộ quyền này sẽ tự động được triển khai xuống các tài khoản AWS dưới dạng các IAM Role.
  - Bước 3 - Ánh xạ phân quyền: Thực hiện gán Nhóm người dùng vào các Tài khoản AWS cụ thể đi kèm với Bộ quyền chỉ định.
    => Ví dụ: Cho phép User John thuộc Group DBA được phép truy cập vào cả tài khoản Production lẫn Development, nhưng tại tài khoản Production chỉ được gán bộ quyền Read-only.

+ Trải nghiệm người dùng:
  - Người dùng chỉ cần ghi nhớ duy nhất một thông tin đăng nhập để truy cập vào một cổng thông tin web tập trung (User Portal).
  => Tại đây, họ có thể lựa chọn danh sách các AWS account được phân quyền để truy cập nhanh chóng mà không cần quản lý thủ công từng password riêng lẻ cho từng account.

---

### 6. AWS Key Management Service (AWS KMS)

+ Tổng quan AWS Key Management Service (AWS KMS):
  - Khái niệm: Dịch vụ quản lý khóa mã hóa cấu hình sẵn, tuân thủ tiêu chuẩn bảo mật nghiêm ngặt FIPS 140-2 Level 2.
  - Vai trò: Chuyên dùng để thực hiện mã hóa dữ liệu ở trạng thái nghỉ (Encryption at Rest) trên các dịch vụ lưu trữ AWS.
  - Cơ chế hoạt động: Dựa trên cơ chế mã hóa phong bì (Envelope Encryption), phân tách rõ hai tầng cấp khóa.

+ Phân loại khóa:
  - Customer Managed Key (CMK) / Master Key:
    * Là khóa gốc tối cao luôn được lưu trữ và bảo vệ nghiêm ngặt bên trong phân lớp bảo mật của AWS KMS, có kích thước tối đa 4 KB.
    => Khóa Master Key này không bao giờ được xuất ra ngoài và chỉ làm nhiệm vụ duy nhất: mã hóa và giải mã các Data Key.
  - Data Key (Khóa dữ liệu):
    * Là khóa trực tiếp dùng để mã hóa dữ liệu thô bên ngoài dịch vụ KMS.

+ Quy trình mã hóa dữ liệu (Envelope Encryption):
  - Bước 1: Ứng dụng gọi KMS để xin một Data Key dựa trên Master Key.
  - Bước 2: KMS trả về 2 bản: Một bản Plaintext Data Key (Khóa thô) và một bản Encrypted Data Key (Khóa đã bị mã hóa bởi Master Key).
  - Bước 3: Ứng dụng dùng bản khóa thô Plaintext Data Key kết hợp thuật toán để mã hóa dữ liệu gốc thành Encrypted Data, sau đó lập tức xóa bản khóa thô này khỏi bộ nhớ để đảm bảo an toàn.
  - Bước 4: Dữ liệu lưu trữ xuống ổ đĩa sẽ bao gồm: Khối dữ liệu đã mã hóa (Encrypted Data) đi kèm với Khóa dữ liệu đã mã hóa (Encrypted Data Key) nằm kế bên.

+ Quy trình giải mã dữ liệu:
  - Khi cần đọc file, hệ thống bốc khóa dữ liệu đã mã hóa gửi ngược lại cho KMS.
  - KMS dùng Master Key nội bộ để giải mã thành data key thô trả về cho ứng dụng giải mã file dữ liệu.
  => Do đó, nếu kẻ tấn công sao chép trái phép tệp tin sang một tài khoản AWS khác mà không được phân quyền sử dụng Master Key nằm tại KMS gốc, dữ liệu hoàn toàn không thể bị giải mã.

---

### 7. Giám sát tuân thủ bảo mật: AWS Security Hub

+ Tổng quan AWS Security Hub:
  - Khái niệm: Dịch vụ quản trị trung tâm, tự động thực hiện các tác vụ quét và đánh giá cấu hình bảo mật liên tục trên toàn bộ tài khoản AWS.

+ Tiêu chuẩn đánh giá:
  - Quét dựa trên các bộ khung tiêu chuẩn của AWS (AWS Foundational Security Best Practices) hoặc các tiêu chuẩn bảo mật khắt khe của ngành tài chính/quản lý dữ liệu như PCI DSS, ISO 27001.

+ Phạm vi hoạt động:
  - Chỉ có thẩm quyền quét và đánh giá các lỗ hổng cấu hình bảo mật ở phần hạ tầng do AWS quản lý.
    => Ví dụ: Phát hiện tường lửa Security Group đang mở toan port nguy hiểm ra Internet, phát hiện chưa bật tính năng mã hóa KMS, chưa kích hoạt MFA cho tài khoản hoặc chưa bật dịch vụ lưu vết nhật ký API CloudTrail.
  - Lưu ý: Dịch vụ không có khả năng can thiệp hay kiểm tra các lỗ hổng bảo mật nằm sâu bên trong hệ điều hành hoặc bên trong source code do người dùng tự viết.

+ Kết quả đầu ra:
  - Cung cấp một bảng điều khiển trung tâm (Dashboard) hiển thị điểm số tuân thủ hệ thống theo tỷ lệ phần trăm (%).
  => Phân loại rõ các phát hiện rủi ro (Findings) theo từng cấp độ nguy hiểm (Low, Medium, High) để kỹ sư nhanh chóng lên phương án khắc phục.