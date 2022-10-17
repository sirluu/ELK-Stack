<h1 align="center">Tổng quan về ELK</h1>

## Mục lục
I. [Logging](#Logging)

II. [Technical Stack là gì ?](#technicalstack)

III. [Khái niệm về ELK Stack](#ELKstack)

IV. [Một số thành phần cơ bản](#thanhphan)


<h3 align="center">-----------------------------------------</h3>

## Phần I. <a name="Logging"></a>Logging

- Logging là một phần không thể thiếu trong bất kì hệ thống nào, logging hỗ trợ lưu lại dấu vết và các hoạt động của ứng dụng, giúp phân tích, điều tra bugs,...
- Đối với các doanh nghiệp việc có thể tập trung logs để quản lý và theo dõi hệ thống rất phúc tạp và khó khăn. 
- Việc quản lý log một các hiệu quả luôn là một vấn đề cần phải xử lý khi mà một hệ thống lớn thì file logs có dung lượng lên đến hàng chục GB, hoặc là đối với mô hình microservice có rất nhiều server tương ứng với có rất nhiều files log được sinh ra
- Ở mọt thời điểm nào đó khi mà hệ thống có bugs, để kiểm tra nguyên nhân logs thì k thể truy cập từng Server để có thể truy vấn đến từng dòng logs. Điều đó tiêu tốn rất nhiều thời gian và công sức mà hiệu quả không cao, Để khắc phục điều đó thì các công cụ quản lý log tập trung đã ra đời nhằm hỗ trợ những người quản trị hệ thống trong vấn đề quản lý log, trong đó nổi trội và mạnh mẽ nhất là ELK Stack hay còn gọi là Elastic Stak


## Phần II. <a name="technicalstack"></a>Technical Stack là gì ?
- Technical Stack hay còn gọi là solution Stack, là tập hợp nhưng công nghệ/phần mềm phối hợp với nhau để có thể tạo ra một nên tàng có thể phối hợp với nhau để tạo ra một ứng dụng có thể hoạt động được.
- Đối với một Stack thường sẽ được cấu tạo bới các thành phần:
  - Hệ điều hành
  - Web Server
  - Database
  - back-end Programming language
- Mỗi thành phần trong stack đảm nhận các vai trò và nhiệm vụ riêng biết và không có sự trùng lặp.

Đọc thêm [tại đây](https://toidicodedao.com/2017/05/23/giai-thich-technical-stack-la-gi/)
## Phần III. <a name="ELKstack"></a>Khái niệm về ELK Stack
- ELK Stack là tập hợp 3 phần mềm kết hợp làm việc chung với nhau, phục vụ cho công việc thu thập logging. 3 Phần mềm lần lượt là
  - Elasticsearch: Cơ sở dữ liệu lưu trữ và query log
  - Logtash: Tiếp nhận log từ nhiều nguồn sau đó xử lý và ghi log vào Elasticsearch
  - Kibana: Giao diện hỗ trợ quản lý, thống kê log. Hỗ trợ đọc thông tin từ Elasticsearch
  - Beats: Tập hợp các công cụ dùng để thu thập thông tin chuyên dụng. Có nhiệm vụ thu thập và vận chuyển dữ liệu từ client đến máy chủ ELK
- Ngoài ra để ELK Stack có thể thu thập logs để xử lý và hiển thị đến người dùng chúng ta cần thêm 1 thành phần rất quan trọng có nhiệm vụ thu thập logs trên các thiết bị client rồi thực hiện vận chuyển những logs đó đến ELK Stack, đó chính là **`Beats`**
-  ELK có điểm mạnh là khả năng thu thập và hiển thị log theo thời gian thực. Có thể đáp ứng truy vấn đến một lượng dữ liệu cực lớn


<h3 align="center"><img src="../../../../../ELK-Stack/03-Images/dosc/1.png"></h3>


## Phần IV. <a name="thanhphan"></a>Một số thành phần cơ bản
### 3.1 Logstash
- Có chức năng phân tích cú pháp của các dòng dữ liệu, việc phân tích là cho dữ liệu ban đầu ở dạng khó đọc, không có dãn nhãn thành dữ liệu có cấu trúc và được dán nhãn.
- Khi cấu hình logstash luôn có 3 phần : Input, filter và Output
- Khi làm việc với logstash thường sẽ phải làm việc với filter nhiều nhất. Filter sử dụng Grok để thực hiện phân tích dữ liệu 

### 3.2 Elasticsearch
- ELK là một RESTful distributed search engine. Có thể hiểu là cung cấp khả năng tìm kiếm phân tán qua API. Sủ dụng kiểu lưu trữ dữ liệu noSQL( lưu trữ dữ liệu không cấu trúc)
- Elasticsearch cho phép tìm kiếm theo nhiều loại hình thức khác nhau: theo cấu trúc, phi cấu trúc, gieo metric theo cách mong muốn
- Elasticsearch rất nhanh, cho phép truy vấn lượng lớn dữ liệu tức thì với các dữ liệu đã thay đổi 
- Có thể thiết lập cài đặt vận hành trên hàng petabyte dữ liệu.
- Vận hành dễ dàng:
  - Có khả năng co giãn và tính sẵn sàng cao
  - Dự đoán trước, đáng tin cậy
  - Đơn giản, trong suôt
  - Elasticsearch sử dụng chuẩn RESTful APIs và JSON.B

### 3.3 Kibana

- kibana được phát triển để sử dụng riêng cho ứng dụng ELK, sử dụng để thực hiện chuyển đổi truy vấn của người dùng thành các câu lệnh truy vấn mà khi đó Elassticsearch có thể thực hiện được. Sau khi truy vấn kết quả có thể hiển thị theo nhiều các và nhiều dạng biểu đồ khác nhau

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/2.png"></h3>




## Phần V. <a name="visao"></a>Tài liệu tham khảo
- https://itnavi.com.vn/blog/elk-la-gi
- https://blog.cloud365.vn/logging/ELK-part1-tong-quan-ve-elk-stack/