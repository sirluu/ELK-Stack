<h1 align="center">Elasticsearch Index Lifecycle Policies</h1>


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

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/53.png"></h3>

Bước 2: đặt tên cho Policy
<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/54.png"></h3>

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

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/55.png"></h3>

- `Force merge`: Gộp các index shard và dọn dẹp các documents đã xóa. 

- `Shrink`: Thu gọn các index và lưu lại thành các index mới

- `Searchable snapshot`: chuyển đổi  dữ liệu data thành snapshots. Tính năng này có thể thay thế reolicas  để đảm bảo tính phục hồi

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/56.png"></h3>

- `Read only`: chỉ cho phép đọc dữ liệu và metadata
- `Index priority`: thiết lập mức độ ưu tiên khôi phục dữ liệu cho các node thuộc Hot phase sau khi khởi động lại

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/57.png"></h3>

**`Warm phase`**
Di chuyển dữ liệu từ lớp hot sang lớp warm, tại đây vẫn cung cấp khả năng tìm kiếm dữ liệu nhưng hoạt động với múc độ ít thường xuyên hơn so với lớp hot. Ở lớp Warm dữ liệu được tối ưu cho tìm kiếm 

- Move data into phase when: Thời giản di chuyển từ Hot phase sang Warm phase tính từ thời điểm data được lưu trữ vào hệ thống
- `Keep data in this phase forever`: Tưởng tự với tầng hot lựa chọn thiết lập thùng rác
- Replicas: thiết lập lại số lượng relicas của các shard, cũng có thể giữ nguyên số lượng relicas như đã thiết lập trước đây

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/58.png"></h3>

- Các phần Shrink, Force merge, Read only được thiết lập sử dụng với mục đích tương tự với tầng hot

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/59.png"></h3>

- `Data allocation`: Vị trí lưu trữ sử dụng các node warms
- `Index priority`: Độ phục hồi dữ liệu ưu tiên 

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/60.png"></h3>

**`Frozen phase`**: Do các node ở tầng warms đã được sử dụng tích hợp thêm role data_cold nên không cần chuyển dữ liệu từ tầng warms đến tầng cold mà có thể chuyển thằng sang tầng frozen.
- Dữ liệu sẽ được chuyển vào tầng này sau 30 ngày để tiến hành lưu trữ 
- Ở đây có thể thực hiện chạy các bản snapshots để backup dữ liệu định kỳ dự trên `Snapshot repository`

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/61.png"></h3>

- `Delete phase`: Mục này thiết lập thời gian tiến hành xóa dữ liệu sau bao lâu kể từ ngày khởi tạo giá trị đó
  - trước khi tiến hành xóa sẽ thực hiện tạo 1 bản snapshots dữ liệu trước khi xóa để tăng tính an toàn hoặc có thể ỏ qua phần này nếu chắc chắc dữ liệu đó có thể xóa mà không dùng lại

  <h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/62.png"></h3>

- Bấm chọn `Save Policy`


  <h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/63.png"></h3>


Bước 3: Gán Policy vừa tạo vào Index Templates để sử dụng

  <h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/64.png"></h3>


- Sau khi gán các Indices được tạo bởi Index Template sẽ được áp dụng Policy này

  <h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/65.png"></h3>

- Kiểm tra