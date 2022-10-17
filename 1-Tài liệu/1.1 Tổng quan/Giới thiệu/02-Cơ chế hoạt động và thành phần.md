<h1 align="center">Cơ chế hoạt động và các thành phần ELK Stack</h1>
## Mục lục
[Lời nói đầu](#loinoidau)

I.  [Cơ chế hoạt động](#coche)

II. [Thành Phần](#thanhphan)

- 2.1 [Logstash](#Logstash)
- 2.2 [Elasticsearch](#Elasticsearch)
- 2.3 [Kibana](#Kibana)
  
<h3 align="center">-----------------------------------------</h3>

## Phần I. <a name="coche"></a>Cơ chế hoạt động

ELK Stack là tập hợp 3 phần mềm đi chung với nhau, phục vụ cho công việc logging. Ba phần mềm này lần lượt là:
  - Logstash
  - Elasticsearch
  - Kibana

- Cơ chế hoạt động của ELK khá là đơn giản, mô tả ở hình sau:
<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/1.png"></h3>
- Phân tích cơ chế hoạt động:

  - Bước 1: logstash tiếp nhận dữ liệu log được đưa đến. Logstash có thể tiếp nhận logs dưới nhiều hình thức khác nhau, chẳng hạn như server gửi 1 UDP repuest chứa log đến URL logstash. Hoặc beats đọc files logs sau đó gửi đến cho logstash
  - Bước 2: Sau khi Logstash đọc logs được gửi về sẽ thêm một số thông tin về IP, thời gian, parse từ dữ liệu log (server, level cảnh báo,nội dung) rồi sẽ thực hiện ghi log xuống database là Elasticsearch
  - Bước 3: Ở bước tiếp theo sau khi log được lưu xuống Elasticsearch thì kibana sẽ tiến hành đọc những logs này và hiển trị quan giao diện để người dùng có thể query và xử lý 

## Phần II. <a name="thanhphan"></a>Thành phần

### 2.1 <a name="Logstash"></a>Logstash

Logstash là một chương trình mã nguồn mở, nằm trong hệ sinh thái của bộ sản phẩm ELK Stack, có nhiệm vụ rất quan trọng bao gồm 3 giai đoạn xử lý sự kiện log (pipeline) tương ứng với 3 module:
- `INPUT`: Tiếp nhận/thu thập dữ liệu sự kiện log ở dạng thô từ các nguồn dữ liệu khác nhau như file,redis,rabbitmq,beats,syslog,...
- `FILTER`: Sau khi tiếp nhận sẽ thự hiện xử lý dữ liệu log như thêm,xóa,thay thế ,...theo cấu hình của quản trị viên để xây dựng lại cấu trúc dữ liệu log event theo như ý muốn
- `OUTOUT`: Cuối cùng sau khi xử lý log sẽ được gửi đến Elasticsearch để tiếp nhận lưu trữ và hiển thị log

***`Luông xử lý dữ liệu logstash`***
<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/3.png"></h3>

- ở bước INPUT, Logstash được cấu hình để tiếp log evnet hoặc đi lấy dữ liệu log ở dịch remote theo nhu cầu. Sau khi lấy được log event thì bước INPUT sẽ ghi dữ liệu event xuống hàng đợi tập trung ở bộ nhớ ram hoặc trên ổ cứng
- Mỗi pipeline worker thread sẽ tiếp tục lấy một loạt các sự kiện đang nằm trong hàng đợi để xử lý FILTER giúp tái cấu trúc dữ liệu log sẽ được gửi đi ở phần OUTPUT. Số lượng sự kiện được xử lý 1 loạt số lượng pipeline worker thread có thể được cấu hình tinh chỉnh tối ưu hơn
- Mặc định Logstash sử dụng hàng đợi nằm trong bộ nhớ ram giữa các giai đoạn (input -> filer và filter -> output) để làm bộ đệm lưu trữ dữ liệu event trước khi xử lý
- Trong một số trường hợp nếu mà dịch vụ logstash vì 1 lý do nào đó bị mà ngừng hoạt động sẽ dẫn đến dữ liệu về event đang nằm trong buffer sẽ biến mất


> Lưu ý: Trong thư mục chứa nhiều file config thì logstash sẽ thực hiện đọc và xử lý một các tuần tự, cần chú ý cách đặt tên file để có vị trí xử lý một cách hơp. Nêu đặt ký tự đầu tiên là số để đánh dấu vị trí
**`INPUT`**

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/4.png"></h3>

Thực hiện sử dụng cấu hình block `INPUT` để có thể quy định cơ chế nhận/lấy log vào bên trong chương trình Logstash. Một số plugin input thường được sử dụng để nhận hoặc lấy log như sau:
  - file: đọc dữ liệu từ file trên hệ thống, tượng tự với lện `tail -f` trên UNIX
  - syslog: chương trình Logstash lắng nghe và tiếp nhận dữ liệu từ syslog thông qua port 514
  - Redis: Đọc dữ liệu từ redis Server , sử dụng cả 2 cơ chế là redis channel và redis list
  - beats: xử lý các dữ liêu thông tin gửi từ chương trình Beats(một sản phẩm trong hệ thống ELK)

Logstassh hỗ trợ nhiều loại Plugin input khác nhau giúp người dùng linh động tring việc nhận nguồn dữ liệu log

**`Filter`**

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/5.png"></h3>

Có thể kết hợp filter cùng với các điều kiện so sánh nhằm thực hiện 1 tác vụ khi một sự kiện thỏa mãn các tiêu chí do người quản trị đặt ra trước đó. Một số Plugin hữu ích thường được dùng như:
  - Grok: là plugin filter tốt nhất để phân tích cú pháp dữ liệu log không có cấu trúc thành dữ liệu có cấu trúc có thể thực hiện truy vấn được
  - mutate: hỗ trợ thực hiện thay đổi thông tin về sự kiện log: đổi tên, xóa, thay thế, tinh chỉnh các trường thông tin có trong log
  - drop: Dừng xử lý các sự kiện tức thì, ví dụ đối với các debug event
  - Clone: tạo ra một bản coppy evnet
  - geoip: thêm thông tin về vị trí địa lý của địa chỉ IP


**`OUTPUTS`**


<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/6.png"></h3>


Outputs là bước cuối trong chuỗi xử lý dữ liệu log của logstsash. Một evnet log có thể được đưa ra nhiều output khác nhau. Một số Output plugin thường được sử dụng
 - elasticsearch: Gửi các dữ liệu log đã qua xử lý đến hệ thống Elasticsearch.
 - file: khi không cần phải lưu trữ dữ liệu logs phục vụ cho tìm kiếm và hiển thị thì có thể thực hiện lưu log ra file trên hệ thống
 - Graphite: là một trong những tool mã nguồn mở hỗ trợ việc tạo biểu đồ metric
 - statsd: gửi dữ liệu tới dịch vụ ‘statsd’.

### 2.2 <a name="Elasticsearch"></a>Elasticsearch
**`Elasticsearch là gì ?`**
- Elasticsearch là một công cụ tìm kiếm và phân tích mã nguồn mở phân tán, phụ hợp với nhiều loại dữ liệu bao gồm văn bản , số , không gian địa lý , dữ liệu có cấu trúc và phi cấu trúc.
- Elasticsearch được biết đến với API kiểu REST đơn giản, các tính năng phân tán, tốc độ và khả năng mở rộng và là thành phần cốt lõi của Elastic Stack. Elastic Stack là một bộ các công cụ nguồn mở để thu thập dữ liệu, làm giàu, lưu trữ, phân tích và trực quan hóa.
- Vì đây là một công cụ mã nguồn mở và miễn phí nên có thể thực hiện cài đặt và sử dụng trên 1 máy chủ cài linux

**`Mô hình hoạt động Elasticsearch`**
- Nguyên Tắc triển khai của Elasticsearch là khi người dùng gửi dữ liệu đên trung tâm dữ liệu Elasticsearch, sau đó sử dụng bộ điều kiển phân doạn từ để thực hiện phân đoạn dữ liệu tương ứng và lưu trữ kết quả phân đoạn vào dữ liệu. Ở thời điểm đó kết quả sẽ được đánh dầu bằng các trọng số và trả về người dùng khi có yêu cầu truy vấn đến những kết quả đó


**`Elasticsearch được sử dụng trong những trường hợp`**
- Tìm kiếm ứng dụng tìm kiếm
- Tìm kiếm trang web
- Tìm kiếm doanh nghiệp
- Xử lý và phân tích dữ liệu
- Giám sát hiệu suất ứng dụng
- Phân tích dữ liệu
- Phân tích bảo mật dữ liệu
- Phân tích kinh doanh

**`Elasticsearch tương thích với nhiều loại ngôn ngữ khác nhau`**
Thư viện của Elasticsearch hỗ trợ nhiều loại ngôn ngũ lập trình khác nhau:
- Java
- javaScripts
- PHP
- C#
- Ruby
- PyThon
- Go
- Perl

**`Một số khái niệm cơ bản có trong Elasticsearch`**
- Document:
  - Là đơn vị nhỏ nhất để lưu trữ dữ liệu trong Elasticsearch. Đây là một đơn vị lưu trữ thông tin cơ bản trong Elasticsearch, là một JSON object đối với một số dữ liệu.

- Index:
  - Trong Elasticsearch có một cấu trúc tìm kiếm gọi là inverted index, nó được thiết kế để cho phép tìm kiếm full-text một cách nhanh chóng. Cách thức khá đơn giản, các văn bản được tách ra thành từng từ có nghĩa sau đó sẽ được map xem thuộc văn bản nào và khi search sẽ ra kết quả cụ thể.
  - Có 2 kiểu đánh index là forward index và inverted index. Bản chất của inverted index là đánh theo keyword: words -> pages còn forward đánh theo nội dung page -> words.
  - Việc đánh theo keyword giúp tìm kiếm sẽ nhanh hơn việc chúng ta phải tìm kiếm theo từng page. Elasticsearch sử dụng Apache lucence để quản lý và tạo inverted index.

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/7.png"></h3>

- Shard:
  - Shard là tập hợp con của một Index. Một index có thể được lưu trên nhiều shard.

  - Một node bao gồm nhiều Shard, shard chính là đối tượng nhỏ nhất hoạt động ở mức thấp nhất, đóng vai trò lưu trữ dữ liệu.

  - Người dùng không làm việc trực tiếp với shard vì Elasticsearch sẽ hỗ trợ chúng ta toàn bộ việc giao tiếp cũng như tự động thay đổi các Shard khi cần thiết.

  - Elasticsearch cung cấp 2 cơ chế của shard đó là primary shard và replica shard.

  - Primary shard sẽ lưu trữ dữ liệu và đánh Index, sau khi đánh dữ liệu xong sẽ được vận chuyển đến các replica shard, mặc định của Elasticsearch mỗi index sẽ có 5 Primary shard thì sẽ đi kèm với 1 Replica shard.

  - Replica shard là nơi lưu trữ dữ liệu nhân bản của Elasticsearch, đóng vai trò đảm bảo tính toàn vẹn dữ liệu khi Primary shard xảy ra vấn đề, ngoài ra nó còn giúp tăng tốc độ tìm kiếm vì chúng ta có thể cấu hình lượng Replica shard nhiều hơn cấu hình mặc định của Elasticsearch.


- Node:
  - Là trung tâm hoạt động của Elasticsearch. Là nơi lưu trữ dữ liễu ,tham gia thực hiện đánh index cúa cluster cũng như thực hiện các thao tác tìm kiếm
  - Mỗi node được định danh bằng 1 `unique name`

- Cluster:
  - Là tập hợp nhiều server hoạt động chung một dịch vụ giống nhau, đảm bảo tính sẵn sàng của một hệ thống. Khi 1 server bị tắt thì các server còn lại vẫn hoạt động nhằm đảm bảo tính ổn định của dịch vụ. Mỗi server trong một cụm Cluster được gọi là 1 node. Trong một cụm Cluster sẽ có 1 node hoạt động chính và node đó được gọi là node master
  - Chức năng chính của Cluster là quyết định xem shard nào được phân bổ cho node nào và khi nào thì di chuyển các Cluster để cần bằng lại Cluster.

### 2.3 <a name="Kibana"></a>Kibana
**`Kibana ?`**

- Kibana là công cụ trực quan hóa dữ liệu ELK Stack. Công cụ này được sử dụng để trực quan hóa các tài liệu Elasticsearch và giúp các nhà phát triển có cái nhìn sâu sắc về nó. 
- Bảng điều khiển Kibana cung cấp các sơ đồ tương tác, dữ liệu không gian địa lý và đồ thị khác nhau để hình dung các quy tắc phức tạp.
- Kibana có thể được sử dụng để tìm kiếm va tương tác với dữ liệu được lưu trữ bên trong Elasticsearch. Kibana hỗ trợ người dùng thực hiện phân tích dữ liệu nâng cao và hỗ trợ biểu thị dữ liệu thông qua bảng, biểu đồ hay là bản đồ

- Kibana có nhiều phương pháp khác nhau để hỗ trợ tìm kiếm dữ liệu

- Một số dạng tìm kiếm phổ biến:

|Search Type| <h3 align="center"><b>Usage</h3>|
|-----------|------|
|Free text searches|Sử dụng tìm kiếm theo chuỗi ký tự|
|Field-level searches| Có thể tìm kiếm chuỗi trong một trường dữ liệu cụ thể|
|Logical statements|Được sử dụng để kết hợp các giá trị tìm kiếm nhằm tạo ra một câu lệnh tìm kiếm logic|
|Proximity searches| Được sử dụng để tìm kiếm cụm từ trong các phạm vi cụ thể|

**`Một số tính năng của Kibana`**
- Bảng điều khiển mạnh mẽ, có khả năng hiển thị thông tin được thiết lập từ cụm elasticsearch cluter
- Hỗ trợ tính năng theo dõi và tìm kiếm theo thời gian thực
- Có thể thực hiện tìm kiếm, xem, tương tác với dữ liệu được lưu trữ bên trong Elasticsearch
- Thực thi truy vấn dữ liệu và biểu thị trực quan hóa ở các dạng bảng, biểu đồ cũng như là bản đồ
- Bảng điều khiển có thể cấu hình để cắt và chia các bản ghi logstash trong Elasticsearch
- Có khả năng cung cấp dữ liệu lịch sử ở dạng đồ họa, biểu đồ,...
- Dễ dàng cấu hình bảng điểu khiển hiển thị dữ liệu theo thời gian thực
- Kibana ElasticSearch cho phép tìm kiếm dữ liệu theo thời gian thực

**`Ưu và nhược điểm của Kibana`**

- Dễ dàng hình dung
- Tích hợp hoàn toàn với Elasticsearch
- Công cụ trực quan hóa
- Cung cấp khả năng phân tích, lập biểu đồ, tóm tắt và gỡ lỗi trong thời gian thực
- Cung cấp giao diện bản năng và thân thiện với người dùng
- Cho phép chia sẻ ảnh chụp nhanh của nhật ký được tìm kiếm qua
- Cho phép lưu trang tổng quan và quản lý nhiều trang tổng quan