<h1 align="center">Tổng quan về ELK</h1>

## Mục lục

I.  [Vì sao nên sử dụng ELK Stack](#visao)

II. [Các trường hợp sử dụng ELK Stack ?](#sudung)


<h3 align="center">-----------------------------------------</h3>

## Phần IV. <a name="visao"></a>Vì sao nên sử dụng ELK Stack ?
- Đối với các hệ thống hoặc ứng dụng nhỏ không cần đến ELK mà có thể truy xuất log định lỳ ra các files và thực hiện theo dõi khi cần thiết là được.

- Ở trường hợp đối với công ty có các hệ thống lớn với nhiều người dùng, có nhiều dịch vụ hoạt động cùng lúc dẫn đến việc ghi logs ra files không còn hiệu quả nữa. Vì số lượng thiết bị lớn nên không thể truy cập vào từng thiết bị để đọc từng files log mà cập quản lý tập trung để quản lý logs.ELK sẽ hỗ trợ xử lý vấn để đó.
- Ưu điểm mà ELK Stack đem lại:
  - `Quản lý log tập trung`: Khi sử dụng ELK người dùng sẽ không phải truy cập vào từng serves để xem log mà người dùng chỉ cần truy câp `Kibana` để có thể xem được log ở tất cả những server mà người dùng mong muốn
  - `Đọc được nhiều loại log khác nhau`: Logstash có thể đọc được log từ rất nhiều nguồn, từ log file cho đến log database cho đến UDP hay REST request.
  - `Dễ dàng tích hợp`: Dù bạn có dùng Nginx hay Apache, dùng MSSQL, MongoDB hay Redis, Logstash đều có thể đọc hiểu và xử lý log của bạn nên việc tích hợp rất dễ dàng.
  - `Triển khai dễ dàng và miễn phí`: Chỉ cần tải về, setup và dùng, không tốn một đồng nào cả. Công ty tạo ra ELK Stack kiếm tiền bằng các dịch vụ cloud hoặc các sản phẩm premium phụ thêm.
  - `Khả năng scale tốt`: Logstash và Elasticsearch chạy trên nhiều node nên hệ thống ELK cực kì dễ scale. Khi có thêm service, thêm người dùng, muốn log nhiều hơn, bạn chỉ việc thêm node cho Logstash và Elasticsearch là xong.
  - `Search và filter mạnh mẽ`: Elasticsearch cho phép lưu trữ thông tin kiểu NoSQL, hỗ trợ luôn Full-Text Search nên việc query rất dễ dàng và mạnh mẽ.
  - Cộng đồng mạnh => tutorial nhiều => dễ dàng tiếp cận.


## Phần V. <a name="sudung"></a>Các trường hợp sử dụng ELK Stack

- Đối với các hệ thống ứng dụng nhỏ, không cần dùng đến ELK mà sẽ sử dụng các thư viện ghi logs đi kèm ngôn ngữ rồi thực hiện ghi logs ra files và tiến hành đọc bình thường.
- Ở trường hợp đối với công ty có các hệ thống lớn với nhiều người dùng, có nhiều dịch vụ hoạt động cùng lúc dẫn đến việc ghi logs ra files không còn hiệu quả nữa. Vì số lượng thiết bị lớn nên không thể truy cập vào từng thiết bị để đọc từng files log mà cập quản lý tập trung để quản lý logs.ELK sẽ hỗ trợ xử lý vấn để đó.
