<h1 align="center">Tổng quan về Beats</h1>

# Phần I. Tổng quan
- Beats là một nền tảng công nghệ miễn phí được phát triển sử dụng cho mục đích vận chuyển dữ liệu trên các thiết bị, hệ thống đến ELK Stack
- Beats hoạt động được hỗ trợ dựa trên thư viện libeat framework giúp hỗ trợ và phát triển 
- Để có thể thu thập dữ liệu gửi về hệ thống ELK Stack, trên các hệ thống server hay thiết bị cần cài đặt các agent tương ứng với loại dữ liệu cần thực hiện thu thâp:
  - `Auditbeat`: Xác thực, kiểm tra các hoạt động cũng như tiến trình trên hệ thống
  - `filebeat`: Thu thâp log và journals trên server linux
  - `Functionbeat`: thu thập dữ liệu trên hệ thống cloud
  - `Heartbeat`: kiểm tra tính khả dụng và năng hoạt động của ứng dụng và hệ thống
  - `Metricbeat`: Kiểm tra và cung cấp số liệu về hiệu năng hoạt động trên thiết bị
  - `Packetbeat`: thu thập dữ liệu về Netwirk Traffic
  - `Winlogbeat`: thu thập dữ liệu event log trên hệ thống thiết bị chạy windows

- Dữ liệu khi đẩy về hệ thống ELK Stack trước khi được kibana hiển thị lên các website sẽ có thể được gửi thẳng trực tiêp đến Elasticsearch để lưu trữ, hoặc gửi đến Logstash để có thể tiến hành xử lý và sẽ được gửi đến Elasticsearch để lưu trữ trước khi hiển thị thông qua kibana


<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/35.png"></h3>

# Phần II. Thành phần
## 1. Auditbeat
### Mô tả
- Để thực hiện theo dõi hoạt động của người dùng, tiến trình và phân tích dữ liệu của hệ thống thông qua ELK Stack cần sử dụng đến auditbeat
- Auditbeat làm việc trực tiếp với linux audit framework, thu thập dữ liệu giống như Audit và gửi các sự kiện thu thập đó về ELK Stack theo thời gian thực
- Trong trường hợp dữ liệu chuyển về bị châm trễ so với thực tế, có thể thực hiện chạy Audit cùng với Audtbeat

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/36.jpg"></h3>

### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/auditbeat/current/auditbeat-installation-configuration.html)


### Config Auditbeat

- Sau đây là một số ví dụ về cấu hình file  auditbeat.yml: 
  - Module
  ```sh
  // Cấu hình thu thập dữ liệu từ auditd và file_integrity modules
  auditbeat.modules:
  
  - module: auditd
    audit_rules: |
      -w /etc/passwd -p wa -k identity
      -a always,exit -F arch=b32 -S open,creat,truncate,ftruncate,openat,open_by_handle_at -F exit=-EPERM -k access
  
  - module: file_integrity
    paths:
    - /bin
    - /usr/bin
    - /sbin
    - /usr/sbin
    - /etc
  ```
  - reload module:
  ```sh
    auditbeat.config.modules:
    path: ${path.config}/conf.d/*.yml
    reload.enabled: true
    reload.period: 10s
  ```
  - Output:
  ```sh
    output.logstash:
    hosts: ["127.0.0.1:5044"]
  ```

> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/auditbeat/current/configuring-howto-auditbeat.html)

## 2. Filebeat

### Mô tả

- Filebeat thu thập dữ liệu từ các thiết bị bảo mật, cloud,container, máy chủ hay các thiết bị lưu trữ. Filebeat hỗ trợ vận chuyển dữ liệu một các đơn giản đến các thiết bị lưu trữ log và file tập trung
- Hỗ trợ tông hợp, xem log theo thời gian thực và hỗ trợ tìm kiếm
- Hoạt động mạnh mẽ, khi ứng dụng downtime filebeat sẽ lưu trữ vào queue và gửi lại nó sau khi hệ thống hoạt động trở lại
- Filebeat làm co những thứ vốn dĩ đã đơn giản trở nên đơn giản hơn cung cấp module cung cấp khả năng quan sát và bảo mật nguồn dữ liệu dẫn đến đơn giản hóa thu thập,phân tích và hình dung

- Hoạt đông tốt trên các nền tảng cloud và container
- Filebeat sử dung giao thức `backpressure-sensitive` khi gửi dữ liệu đến Logstash hoặc elaasticsearch. Trong trường hợp Logstash đang xử lý dữ liệu, khi đó filebeat sẽ chủ động làm chậm quá trình đọc

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/36.png"></h3>


### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)


> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html)


## 3. Functionbeat

### Mô tả
- Được triển khai dưới dạng chức năng trong nên tảng do nhà cung cấp đem tới dựa trên  nền tảng dịch vụ FaaS, thu thập, vận chuyển và giám sáng dữ liệu dịch vụ cloud
- Khả năng triển khai đơn giản khi sử dụng một câu lệnh duy nhất. 

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/37.png"></h3>

- Triển khai Functionbeat với tư cách là shipper không có máy chủ để chuyển trực tiếp các luồng dữ liệu từ AWS CloudWatch Logs đến Elastic Stack để giám sát thống nhất tốt hơn. Theo dõi chặt chẽ nhật ký cho tất cả các ứng dụng và hệ thống trong quá trình triển khai của bạn.
- Sử dụng Functionbeat để khai thác vào hàng đợi AWS Kinesis hoặc SQS để tiến hành vô số quy trình xử lý theo hướng sự kiện, tất cả đều theo kiểu không máy chủ. Cho dù là nhật ký, chỉ số hay bất kỳ dữ liệu nào khác, hãy truyền trực tuyến các sự kiện của bạn vào Elastic Stack để tìm kiếm và phân tích trong thời gian thực.

### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/functionbeat/current/functionbeat-installation-configuration.html)


> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/functionbeat/current/configuring-howto-functionbeat.html)

### 4. Heartbeat

- Heartbeat liên tục kiểm tra tính khả dụng của các dịch vụ một các liên tục và tích cực
- Heartbeat dễ dàng tạo ra thời gian hoạt động và thời gian phản hồi data. Hỗ trợ kibana trong vấn đề khởi tạo các bảng biểu để có 1 cái nhìn trực quan
- Để kiểm tra tính khả dụng của dịch vụ Heartbeat sử dụng ping thông qua các giao thức ICMP,TCP,HTTP và hỗ trợ TLS, xác thực hay proxies. Có thể thực hiện giám sát tất các hosts sử dụng load-balanced thông qua phân giải DNS
<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/38.png"></h3>


### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/heartbeat/current/heartbeat-installation-configuration.html)


> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/heartbeat/current/configuring-howto-heartbeat.html)

### 5. Metricbeat

- Metricbeat hỗ trợ gọi đến các thông số hệ thống hoặc các services: CPU,Ram,...

- Triển khai Metricbeat trên các thiết bị Linux, Windows, Mac kết nối đến hệ thống ELK Stack sẽ hiển thị các thông tin như: System , CPU,RAM, Memory, files system, disk IO và network IO giống với các thông kê hiệu năng đang hoạt động trên các thiết bị
- Metricbeat đi kèm với các module thu thập thông số như Apache, Jolokia, NGINX, MongoDB, MySQL, PostgreSQL, Prometheus,... thực hiện cài đặt và không có bất kỳ sự phụ nào mà chỉ cần kích hoạt các module cần thiết trong file config


- Việc khởi tạo module khá đơn giản, khi cần thiết có thể chủ động khởi tạo các module được viết bằng Go


<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/39.jpg"></h3>


### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation-configuration.html)


> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/metricbeat/current/configuring-howto-metricbeat.html)


### 6. Packetbeat

- Theo dõi network traffic và điều quan trọng trong quan sát và đảm bảo môi trường hệ thống, hiệu năng và bảo mật đối với hệ thống

- Theo dõi đối với dịch vụ và ứng dụng 

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/39.png"></h3>


### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/packetbeat/current/packetbeat-installation-configuration.html)

> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/packetbeat/current/configuring-howto-packetbeat.html)


### 7. Winlogbeat

- thực hiện theo dõi các vấn đề đang xảy ra trên thiết bị windows. Winlogbeat truyền trực tiếp các event log tới Logstash

- Có khả năng đọc bất kỳ kệnh nhật ký nào trên windows

### Tài liệu hướng dẫn cài đặt [tại đây](https://www.elastic.co/guide/en/beats/winlogbeat/current/winlogbeat-installation-configuration.html)

> Để có thể xem kỹ hơn về các Option và cấu hình `config` chọn [xem thêm](https://www.elastic.co/guide/en/beats/winlogbeat/current/configuring-howto-winlogbeat.html)
