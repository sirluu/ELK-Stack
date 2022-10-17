<h1 align="center">Elasticsearch Multi-Tier Architecture</h1>

# Phần I. Giới thiệu về kiến trúc hot-warm-cold-frozen
- Kiến trúc nhiều tầng cho phép Elasticsearch thực hiện tối ưu trong các trường hợp tìm kiếm. Các node thuộc cluster cần được tổ chức ở hình thưc pools đối với các node cần có cấu hình nhanh để đáp ứng nhu cầu thường xuyên tìm kiếm data, tiếp sau đó sẽ là nhiều cấu hình khác nhau kém mạnh mẽ hơn tùy thuộc yêu cầu sử dụng tìm kiếm hoặc ghi dữ liệu trên các node
- Với một số cấu hình thì số lượng node đối với mỗi tầng có thể không giống nhau
- Một data thường xuyên được truy vấn có thể được lưu trữ tại tầng có tỉ lệ sô node nhiều nhất và tốc độ nhanh nhất
- Không bắt buộc yêu cầu ở các tầng lưu trữ phải có cấu hình khác nhau, nhưng thường đối với tầng lưu trữ dữ liệu sử dụng thường xuyên sẽ có cấu hình manh và nhanh nhất.

- Elastisearh triển khai nhiều tầng sẽ có 3 tầng : hot, warm, cold, frozen và trong vòng đời lưu trữ dữ liệu sẽ lần lượt đi qua  3 tầng đó.
- Không yêu cầu cần triển một hệ thống Elasticsearc với đầy đủ các tầng trên. Đối với tầng hot là tầng ghi dữ liệu đầu vào của elasticsearch nên sẽ xuất hiện ở tất cả các trường hợp


<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/52.png"></h3>

# Phần II. Multi-tier architecture

Tầng dữ liệu là tập hợp các node có chung đặc điểm về lưu trữ và sử dụng dữ liệu. Sự khác nhau ở đây chủ yếu đến từ tính khả dụng và phần cứng tài nguyên
Hiện nay đag có 5 node role đại diện cho các tầng dữ liệu:
- `Content tier`: Tầng hỗ trợ lập chỉ mục  và tìm kiếm nội dung các dữ liệu nhập vào
  - Tầng này lưu trữ nội dung dữ liệu không có sự thay đổi theo thời gian. Có thể thực hiện chuyển đến một tầng khác sau một thời gian nhất định. Thường xuyên thực hiện tối ưu hóa trong các trường hợp không lập chỉ mục chuyên sâu như là dữ liệu về chuỗi thời gian. Đây là tầng bắt buộc vì dữ lieuj cần lập chỉ mục thì phải đi qua nó
- `Hot tier`: đây là tầng thực hiện xử lý các index theo dòng thời gian của log hay metrics và thục hiện giữ lại, lưu trữ các dữ liệu sử dụng thường xuyên
  - Đây là tâng lưu trữ dữ liệu hoạt động với tần suất cao và liên tục nên cần sử dụng hệ thống phần cứng đáp ứng yêu cầu xử lý dữ liệu nhanh chóng, đặc biệt là khả năng đọc ghi dữ liệu
  - VÌ hot tier là lớp lưu trữ dữ liệu đầu tiên tiếp xúc với dữ liệu nên các thao tác bóc tách , thực hiện lập các index lưu trữ dữ liệu cũng diễn ra tại đây
> Trong 1 hệ thông ELK gồm nhiều lớp lưu trữ data thì Content tier và Hot tier là 2 roles hoạt động cao và liên tực nên thường được đặt chung role với nhau trong một lớp lưu trữ có cấu hình hình phần cứng đáp ứng được yêu cầu tốc độ đọc ghi dữ liệu cao hơn các lớp lưu trữ còn lại.

- `Warm tier`: So với `hot tier` là lớp có mức hoạt động cao của dữ liệu thì đến Warm tier các dữ liệu được lưu trữ tại đây có mức sử dụng ít hơn
  - Đối với các index đã được khởi tạo ở lớp hot thì sau một khoảng thời gian mà hệ thống cũng như người dùng không còn truy vấn đến nhiều như ở thời điểm vừa được khởi tạo sẽ được được chuyển đến lớp warm để lưu trữ để nhường chỗ cho những index mới có thế được khởi tạo
  - Do nhu cầu hoạt động của các index không còn lớn như trước nữa nên cấu hình phần cứng sử dụng để lưu trữ 

- `Cold tier`: Tầng lưu trữ dữ liệu gần như không được sử dụng và cập nhật nữa 
  - Tại đây dữ liệu vẫn có thể được thực hiện tìm kiếm trong trường hợp cần thiêt với mức độ thường xuyên là thấp hơn tầng hot và warm.
  - Đối với dữ liệu tầng này thường sẽ bị thu nhỏ và nén lại nằm thực hiện giải phóng và tăng không gian lưc trữ 

- `Frozen tier`: Tại đây dữ liệu gần như đã không còn được truy vấn mà chuyển vào trạng thái chuẩn bị xóa khỏi hệ thống
  - Tại đây dữ liệu được thực hiện tạo các bản snapshot để lưu trữ vào một repo snapshot đã được cầu hình. Sau khi Snapshot hoàn thành thường dữ liệu tại đây sẽ bị xóa với mục đích giảm thiểu chi phí vận hành và tăng không gian lưu trữ dữ liệu cho hệ thống
  - đây là lớp mà thường sẽ không được cấu hình relicas

- Với các role như trên thì 1 hệ thống thường được chia làm 3 lớp trong đó gồm các role:
  - Lớp 1: data_content, data_hot sử dụng cho các dữ liệu index mới có yêu hệ thống có I/O cao
  - Lớp 2: data_Warm, data_cold có mức hoạt động ít hơn và yêu cầu hệ thống có tốc độ xử lý thấp hơp
  - Lớp 3: data_Frozen Lưu trữ data đang trong giai đoạn tạo snapshots và chờ xóa , đ
> Lưu ý: 
> - Khi thiết kế hệ thống nhiều lớp  lưu trữ data cần tính toán số lượng node cho mỗi lớp. Các Primary shards và  Primary Replicas chỉ lưu trữ các lớp với nhau.


# Phần III. Thiết lập lưu trữ Multi-tier
## 1. Phần cứng
- Để thực hiện khởi tạo nhiều tầng lưu trữ dữ liệu , bước đầu cần thự hiện cấu hình khởi tạo một cụm trong đó mỗi node sẽ có 1 số role đặc thù về mục đích sử dụng. Việc chia role thực hiện trong file elasticsearch.yml
- Ở trong một cụm ELK thông thường để lưu trữ data thường chỉ dùng role `data` cho tất cả các lớp , khi chia role theo cấu trúc Multi-tier thường sẽ chia thành 3 nhóm role như đã kể trên bao gồm:
  - Node_data1
  ```sh
  node.roles: [ data_content, data_hot ]
  ```
  - Node_data2
  ```sh
  node.roles: [ data_content, data_hot ]
  ```
  - Node_data3
  ```sh
  node.roles: [ data_warm, data_cold ]
  ```
  - Node_data4
  ```sh
  node.roles: [ data_warm, data_cold ]
  ```
  - Node_data5
  ```sh
  node.roles: [ data_frozen ]
  ```
- Trên đây các chia role để đảm bảo mỗi lớp có mà dữ liệu có thể được thực hiện truy vấn  sẽ có 1 bản replicas để đảm bảo tính an toàn dữ liệu và hỗ trợ truy vấn dữ liệu một cách nhanh chóng, 
- Như đã nói ở trên thì node data_frozen thường dùng cho những dữ liệu trong thòi gian thực hiện backup và chờ xóa nên sẽ không cần dùng đến các bản repicas
- Đối với mỗi node kể trên có thể add thêm một số rule như master, ingest, coordinating để hoàn thành 1 cụm Elasticsearch cluster hoàn chỉnh 

## Index Lifecycle Policies
- Để thiết lập 1 vòng đồi lưu trữ từ khởi tạo , di chuyển giữa lớp lưu trữ dữ host, warm, cold, frozen và sau đó là thực hiện backup và xóa dữ liệu đó khỏi hệ thống sẽ sử dụng đến `Index Lifecycle Policies ` để thực hiện việc đó

- Để tạo nên 1 Index Lifecycle Policies hoàn chỉnh cần khởi tạo theo các bước sau:
  - Tạo Indices thông qua Index Templates
  ```sh
  Data -> Index Management -> Component Templates -> Index Templates -> Indices
  ```
  - Khởi tạo các Snapshots thông qua Policies
  ```sh
  Data -> Snapshot and Restore -> Repositories -> Policies -> Snapshots
  ```

### Khởi tạo **`Index Lifecycle Policies`**

Bước 1: truy cập theo đường dẫn `Management`-> `Data` -> `Index Lifecycle Policies` -> `Create policy`

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/53.png"></h3>

Bước 2: đặt tên cho Policy
<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/54.png"></h3>

Bước 3: Thiết lập

**`Hot phase`** (thiết lập mặc định): Sử dụng lưu trữ dữ liệu lớp hot data
- `Keep data in this phase forever`: Chọn biểu tượng thùng rác để xóa dữ liệu sau khi hoàn thành dữ liệu từ `Hot phase` sang `Warm phase`
- `Rollover`: Cấu hình cho phép tạo ra index mới khi index cũ đã đạt một trong ác giới hạn đã đặt ra, bao gồm các thông số: 
  - Enable rollover: Kích hoạt tính năng thiết lập thủ công
  - Maximum primary shard size: Tổng dung lượng của một primary shard
  - Maximum age: thời gian lưu trữ của 1 index
  - Maximum documents: Số lượng documents có thể tạo ra trong index
  - Maximum index size: dung lượng tối đa của index
  - Use recommended defaults: Sử dụng cấu hình mặc định là khởi tạo index mới khi index cũ đạt 30 ngày hoặc primary shard đạt dung lượng 50G

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/55.png"></h3>

- `Force merge`: Gộp các index shard và dọn dẹp các documents đã xóa. 

- `Shrink`: Thu gọn các index và lưu lại thành các index mới

- `Searchable snapshot`: chuyển đổi  dữ liệu data thành snapshots. Tính năng này có thể thay thế reolicas  để đảm bảo tính phục hồi

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/56.png"></h3>

- `Read only`: chỉ cho phép đọc dữ liệu và metadata
- `Index priority`: thiết lập mức độ ưu tiên khôi phục dữ liệu cho các node thuộc Hot phase sau khi khởi động lại

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/57.png"></h3>

**`Warm phase`**
Di chuyển dữ liệu từ lớp hot sang lớp warm, tại đây vẫn cung cấp khả năng tìm kiếm dữ liệu nhưng hoạt động với múc độ ít thường xuyên hơn so với lớp hot. Ở lớp Warm dữ liệu được tối ưu cho tìm kiếm 

- Move data into phase when: Thời giản di chuyển từ Hot phase sang Warm phase tính từ thời điểm data được lưu trữ vào hệ thống
- `Keep data in this phase forever`: Tưởng tự với tầng hot lựa chọn thiết lập thùng rác
- Replicas: thiết lập lại số lượng relicas của các shard, cũng có thể giữ nguyên số lượng relicas như đã thiết lập trước đây

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/58.png"></h3>

- Các phần Shrink, Force merge, Read only được thiết lập sử dụng với mục đích tương tự với tầng hot

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/59.png"></h3>

- `Data allocation`: Vị trí lưu trữ sử dụng các node warms
- `Index priority`: Độ phục hồi dữ liệu ưu tiên 

<h3 align="center"><img src="../../../../../ELK-Stack/03-Images/dosc/60.png"></h3>

**`Frozen phase`**: Do các node ở tầng warms đã được sử dụng tích hợp thêm role data_cold nên không cần chuyển dữ liệu từ tầng warms đến tầng cold mà có thể chuyển thằng sang tầng frozen.
- Dữ liệu sẽ được chuyển vào tầng này sau 30 ngày để tiến hành lưu trữ 
- Ở đây có thể thực hiện chạy các bản snapshots để backup dữ liệu định kỳ dự trên `Snapshot repository`

<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/61.png"></h3>

- `Delete phase`: Mục này thiết lập thời gian tiến hành xóa dữ liệu sau bao lâu kể từ ngày khởi tạo giá trị đó
  - trước khi tiến hành xóa sẽ thực hiện tạo 1 bản snapshots dữ liệu trước khi xóa để tăng tính an toàn hoặc có thể ỏ qua phần này nếu chắc chắc dữ liệu đó có thể xóa mà không dùng lại

  <h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/62.png"></h3>

- Bấm chọn `Save Policy`


  <h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/63.png"></h3>


Bước 3: Gán Policy vừa tạo vào Index Templates để sử dụng

  <h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/64.png"></h3>


- Sau khi gán các Indices được tạo bởi Index Template sẽ được áp dụng Policy này

  <h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/65.png"></h3>