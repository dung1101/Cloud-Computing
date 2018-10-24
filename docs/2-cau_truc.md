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



**Nguồn:**
[https://github.com/meditechopen/meditech-thuctap/tree/master/ThaoNV](https://github.com/meditechopen/meditech-thuctap/tree/master/ThaoNV)
