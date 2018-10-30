# Tổng quan về các services thành phần trong OpenStack

| Service          | Project name | Description                              |
| ---------------- | ------------ | ---------------------------------------- |
| Dashboard        | Horizon      | Cung cấp giao diện trên nền tảng web để người dùng tương tác với các dịch vụ khác của OpenStack ví dụ như tạo VM, gán địa chỉ IP, cấu hình kiểm soát truy cập... |
| Compute service  | Nova         | Quản lí các máy ảo trong môi trường OpenStack. Nó có trách nhiệm khởi tạo, lên lịch và gỡ bỏ các VM theo yêu cầu |
| Networking       | Neutron      | Cung cấp khả năng kết nối mạng cho các dịch vụ khác. Cung cấp API cho người dùng tạo ra network và quản lí chúng. Nó có kiến trúc linh hoạt hỗ trợ hầu hết các công nghệ về networking. |
| Object Storage   | Swift        | Lưu trữ và truy xuất các dữ liệu phi cấu trúc thông qua RESTful, HTTP based API. Nó có khả năng chịu lỗi cao nhờ cơ chế sao chép và mở rộng. Quy trình thực hiện của Swift không giống với file server. |
| Block Storage    | Cinder       | Cung cấp các storage để chạy máy ảo. Kiến trúc linh hoạt của nó cho phép người dùng khởi tạo và quản lí các thiết bị lưu trữ. |
| Identity service | Keystone     | Cung cấp dịch vụ xác thực và ủy quyền cho các dịch vụ khác. Cung cấp một danh mục các thiết bị đầu cuối cho tất cả các dịch vụ OpenStack. |
| Image Service    | Glance       | Lưu trữ và truy xuất các ổ đĩa của máy ảo. Compute sẽ sử dụng dịch vụ này trong suốt quá trình khởi tạo và chạy máy ảo |

## 1. Nova - Compute service
### 1.1) Nova là gì ?
- Là service chịu trách nhiệm chứa và quản lí các máy ảo trong môi trường OpenStack. Nó có trách nhiệm khởi tạo,  xóa, bật, tắt, ...

### 1.2) Các thành phần
<img src="http://i.imgur.com/SWgKmDp.png"> 

- **nova-api** : Là service tiếp nhận và phản hồi các compute API calls từ user. Service hỗ trợ OpenStack Compute API, Amazon EC2 API và Admin API đặc biệt được dùng để user thực hiện các thao tác quản trị. Nó cũng có một loạt các policies và thực hiện hầu hết các orchestration activities ví dụ như chạy máy ảo.
- **nova-api-metadata** : Là service tiếp nhận các metadata request từ máy ảo. Service này thường được dùng khi chạy multi-host kết hợp với nova-network.
- **nova-compute** : Là service chịu trách nhiệm tạo và hủy các máy ảo qua hypervisors APIs. Ví dụ:
  - XenAPI for XenServer/XCP
  - libvirt for KVM or QEMU
  - VMwareAPI for VMware

Quá trình xử lí khá phức tạp, về cơ bản, daemon tiếp nhận các actions từ queue và thực hiện một loạt các câu lệnh hệ thống như chạy máy ảo kvm và upsate trạng thái trong database.
- **nova-placement-api** : Lần đầu xuất hiện tại bản Newton, placement api được dùng để theo dõi thống kê và muức độ sử dụng của mỗi một resource provider. Provider ở đây có thể là compute node, shared storage pool hoặc  IP allocation pool. Ví dụ, một máy ảo có thể được khởi tạo và lấy RAM, CPU từ compute node, lấy disk từ storage bên ngoài và lấy địa chỉ IP từ pool resource bên ngoài.
- **nova-scheduler** : Service này sẽ lấy các yêu cầu máy ảo đặt vào queue và xác định xem chúng được chạy trên compute server host nào.
- **nova-conductor** : Là module chịu trách nhiệm về các tương tác giữa `nova-compute` và database. Nó sẽ loại bỏ tất cả các kết nối trực tiếp từ `nova-compute` tới database.
- **nova-consoleauth** : Xác thực token cho user mà console proxies cung cấp. Dịch vụ này buộc phải chạy cùng với console proxies. Bạn có thể chạy proxies trên 1 nova-consoleauth service hoặc ở trong 1 cluster configuration.
- **nova-novncproxy** : Cung cấp proxy cho việc truy cập các máy ảo đang chạy thông qua VNC connection. Nó hỗ trợ các trình duyệt based novnc clients.
- **nova-spicehtml5proxy** : Cung cấp proxy để truy cập các máy ảo đang chạy thông qua SPICE connection. Nó hỗ trợ các trình duyệt based HTML5 client.
- **nova-xvpvncproxy** : Cung cấp proxy cho việc truy cập các máy ảo đang chạy thông qua VNC connection. Nó hỗ trợ OpenStack-specific Java client.
- **The queue**: Trung tâm giao tiếp giữa các daemons. Thường dùng RabbitMQ hoặc các AMQP message queue khác như ZeroMQ.
- **SQL database** : Dùng để lưu các trạng thái của hạ tâng caloud bảo gồm:
  - Các loại máy ảo có thể chạy
  - Các máy ảo đang được dùng
  - Các network khả dụng
  - projects
## 2. Keystone - Identity service
### 2.1) Keystone là gì ?
`Keystone is an OpenStack service that provides API client authentication, service discovery, and distributed multi-tenant authorization by implementing OpenStack’s Identity API. It supports LDAP, OAuth, OpenID Connect, SAML and SQL.`

Keystone chịu trách nhiệm cung cấp các kết nối mang tính bảo mật cao tới các nguồn tài nguyên cloud.
### 2.2) Các khái niệm
-  Projects
    - Trong Keystone, Project được dùng bởi các services của OpenStack để nhóm và cô lập các nguồn tài nguyên. Nó có thể được hiểu là 1 nhóm các tài nguyên mà chỉ có một số các user mới có thể truy cập và hoàn toàn tách biệt với các nhóm khác.
    - Ban đầu nó được gọi là tenants sau đó được đổi tên thành projects.
    - Mục đích cơ bản nhất của keystone chính là nơi để đăng kí cho các projects và xác định ai được phép truy cập projects.
    - Bản thân projects không sở hữu users hay groups mà users và groups được cấp quyền truy cập tới project sử dụng cơ chế gán role.
    - Trong một vài tài liệu của OpenStack thì việc gán role cho user còn được gọi là "grant".
- Domains
    - Trong thời kì đầu, không có bất cứ cơ chế nào để hạn chế sự xuất hiện của các project tới những nhóm user khác nhau. Điều này có thể gây ra những sự nhầm lẫn hay xung đột không đáng có giữa các tên của project của các tổ chức khác nhau.
    - Tên user cũng vậy và nó hoàn toàn cũng có thể dẫn tới sự nhầm lẫn nếu hai tổ chức có user có tên giống nhau.
    - Vì vậy mà khái niệm Domain ra đời, nó được dùng để cô lập danh sách các Projects và Users.
    - Domain được định nghĩa là một tập hợp các users, groups, và projects. Nó cho phép người dùng chia nguồn tài nguyên cho từng tổ chức sử dụng mà không phải lo xung đột hay nhầm lẫn.
- Users and User Groups (Actors)
    - Trong keystone, Users và User Groups là những đối tượng được cấp phép truy cập tới các nguồn tài nguyên được cô lập trong domains và projects.
    - Groups là một tập hợp các users. Users và User Groups được gọi là Actors.
    - Mối quan hệ giữa  domains, projects, users, và groups:
    <img src="http://i.imgur.com/iYkqE5O.png">
- Roles 
    - Roles được dùng để hiện thực hóa việc cấp phép trong keystone. Một actor có thể có nhiều roles đối với từng project khác nhau.
- Assignment
    - Role assignment là sự kết hợp của actor, target và role.
    - Role assignment được cấp phát, thu hồi, và có thể được kế thừa giữa các users, groups, project và domains.
- Targets
    - Projects và Domains đều giống nhau ở chỗ cả hai đều là nơi mà role được "gán" lên.
    - Vì thế chúng được gọi là targets.
- Token
    - Để user truy cập bất cứ OpenStack API nào thì user cần chúng minh họ là ai và họ được phép gọi tới API. Để làm được điều này, họ cần có token và "dán" chúng vào "API call". Keystone chính là service chịu trách nhiệm tạo ra tokens.
    - Sau khi được xác thực thành công bởi keystone thì user sẽ nhận được token. Token cũng chứa các thông tin ủy quyền của user trên cloud.
    - Token có cả phần ID và payload. ID được dùng để đảm bảo rằng nó là duy nhất trên mỗi cloud và payload chứa thông tin của user.
- Catalog
    - Chứa URLs và endpoints của các services khác nhau.
    - Nếu không có Catalog, users và các ứng dụng sẽ không thể biết được nơi cần chuyển yêu cầu để tạo máy ảo hoặc lưu dữ liệu.
    - service này được chia nhỏ thành danh sách các endpoints và mỗi một endpoint sẽ chứa admin URL, internal URL, and public URL.
### 2.3) Chức năng
- Identity:
    - Nhận diện những người đang cố truy cập vào các tài nguyên cloud
    - Trong keystone, identity thường được hiểu là User
    - Tại những mô hình OpenStack nhỏ, identity của user thường được lưu trữ trong database của keystone. Đối với những mô hình lớn cho doanh nghiệp thì 1 external Identity Provider thường được sử dụng.
- Authentication:
    - Là quá trình xác thực những thông tin dùng để nhận định user (user's identity) keystone có tính pluggable tức là nó có thể liên kết với những dịch vụ xác thực người dùng khác như LDAP hoặc Active Directory.
    - Thường thì keystone sử dụng Password cho việc xác thực người dùng. Đối với những phần còn lại, keystone sử dụng tokens.
    - OpenStack dựa rất nhiều vào tokens để xác thực và keystone chính là dịch vụ duy nhất có thể tạo ra tokens
    - Token có giới hạn về thời gian được phép sử dụng. Khi token hết hạn thì user sẽ được cấp một token mới. Cơ chế này làm giảm nguy cơ user bị đánh cắp token.
    - Hiện tại, keystone đang sử dụng cơ chế bearer token. Có nghĩa là bất cứ ai có token thì sẽ có khả năng truy cập vào tài nguyên của cloud. Vì vậy việc giữ bí mật token rất quan trọng.
- Access management(Authorization):
    - Access Management hay còn được gọi là Authorization là quá trình xác định những tài nguyên mà user được phép truy cập tới.
    - Trong OpenStack, keystone kết nối users với những Projects hoặc Domains bằng cách gán role cho user vào những project hoặc domain ấy.
    - Các projects trong OpenStack như Nova, Cinder...sẽ kiểm tra mối quan hệ giữa role và các user's project và xác định giá trị của những thông tin này theo cơ chế các quy định (policy engine). Policy engine sẽ tự động kiểm tra các thông tin (thường là role) và xác định xem user được phép thực hiện những gì.

## 3. Neutron  - Networking
### 3.1) Neutron là gì ?
`OpenStack Neutron is an SDN networking project focused on delivering networking-as-a-service (NaaS) in virtual compute environments.`

OpenStack Networking cho phép bạn tạo và quản lí các network objects ví dụ như networks, subnets, và ports cho các services khác của OpenStack sử dụng. Với kiến trúc plugable, các plug-in có thể được sử dụng để triển khai các thiết bị và phần mềm khác nhau, nó khiến OpenStack có tính linh hoạt trong kiến trúc và triển khai.

Dịch vụ Networking trong OpenStack (neutron) cũng cấp API cho phép bạn định nghĩa các kết nối mạng và gán địa chỉ ở trong môi trường cloud. Nó cũng cho phép các nhà khai thác vận hành các công nghệ networking khác nhau cho phù hợp với mô hình điện toán đám mây của riêng họ. Neutron cũng cung cấp một API cho việc cấu hình cũng như quản lí các dịch vụ networking khác nhau từ L3 forwarding, NAT cho tới load balancing, perimeter firewalls, và virtual private networks.

### 3.2) Các định nghĩa
- Với neutron, bạn có thể tạo và cấu hình các networks, subnets và thông báo tới Compute để gán các thiết bị ảo vào các ports của mạng vừa tạo. OpenStack Compute chính là "khách hàng" của neutron, chúng liên kết với nhau để cung cấp kết nối mạng cho các máy ảo. Cụ thể hơn, OpenStack Networking hỗ trợ cho phép các projects có nhiều private networks và các projects có thể tự chọn danh sách IP cho riêng mình, kể cả những IP đã được sử dụng bởi một project khác.
- **Provider network** ánh xạ tới các mạng vật lý hoặc VLAN thực tế trong trung tâm dữ liệu và không thể được sử dụng để tạo cấu trúc liên kết của user. Chúng chỉ có thể được thiết lập bởi admin.
- **Self-service network** cho phép user tạo topology của riêng họ: Bao nhiêu mạng ảo như họ muốn, kết nối với các bộ định tuyến ảo và kết nối với mạng bên ngoài.

### 3.3) Thành phần
- **API server**: OpenStack Networking API hỗ trợ Layer2 networking và IP address management (IPAM - quản lý địa chỉ IP), cũng như một extension để xây dựng router Layer 3 cho phép định tuyến giữa các networks Layer 2 và các gateway để ra mạng bên ngoài. OpenStack Networking cung cấp một danh sách các plug-ins (đang ngày càng tăng lên) cho phép tương tác với nhiều công nghệ mạng mã nguồn mở và cả thương mại, bao gồm các routers, switches, switch ảo và SDN controller.
- **OpenStack Networking plug-in and agents**: Các plugin và các agent này cho phép gắn và gỡ các ports, tạo ra network hay subnet, và đánh địa chỉ IP. Lựa chọn plugin và agents nào là tùy thuộc vào nhà cung cấp và công nghệ sử dụng trong hệ thống cloud nhất định. Điều quan trọng là tại một thời điểm chỉ sử dụng được một plug-in.
- **Messaging queue**: Tiếp nhận và định tuyến các RPC requests giữa các agents để hoàn thành quá trình vận hành API. Các Message queue được sử dụng trong ML2 plugin để thực hiện truyền thông RPC giữa neutron server và các neutron agents chạy trên mỗi hypervisor, cụ thể là các ML2 driver cho Open vSwitch và Linux bridge.
## 4. Glance - Image service 
### 4.1) Glance là gì ?
```Galance image service bao gồm khám phá, đăng ký và truy xuất các VM image. Glance có một API RESTful cho phép truy vấn VM metadata image cũng như truy xuất actual image. VM image có sẵn thông qua Glance có thể được lưu trữ ở nhiều vị trí khác nhau từ các hệ thống tập tin đơn giản đến các hệ thống lưu trữ Object như OpenStack Swift.```
### 4.2) Các thành phần
Glance có các thành phần sau:
- glance-api : tiếp nhận lời gọi API để tìm kiếm, thu thập và lưu trữ image
- glance-registry: thực hiện tác vụ lưu trữ, xử lý và thu thập metadata của images. Metadata bao gồm các thông tin như kích thước và loại image.
- database: cơ sở dữ liệu lưu trữ metadata của image
<img src="http://i.imgur.com/qiYM1n9.png">

- Storage repository for image files: Có rất nhiều các loại repository được hỗ trợ để lưu image. Một vài backend phổ biến:
  - File system: Glance lưu trữ images của các máy ảo trong hệ thống tệp tin thông thường theo mặc định, hỗ trợ đọc ghi các image files dễ dàng vào hệ thống tệp tin.
  - Object Storage: Là hệ thống lưu trữ do OpenStack Swift cung cấp - dịch vụ lưu trữ có tính sẵn sàng cao , lưu trữ các image dưới dạng các object.
  - BlockStorage : Hệ thống lưu trữ có tính sẵn sàng cao do OpenStack Cinder cung cấp, lưu trữ các image dưới dạng khối
  - VMWare
  - HTTP : Glance có thể đọc các images của các máy ảo sẵn sàng trên Internet thông qua HTTP. Hệ thống lưu trữ này chỉ cho phép đọc.
  - RADOS Block Device(RBD): Lưu trữ các images trong cụm lưu trữ Ceph sử dụng giao diện RBD của Ceph




**Nguồn:**

[https://github.com/meditechopen/meditech-thuctap/tree/master/ThaoNV](https://github.com/meditechopen/meditech-thuctap/tree/master/ThaoNV)
[https://www.openstack.org/software/project-navigator/openstack-components#openstack-services](https://www.openstack.org/software/project-navigator/openstack-components#openstack-services)
