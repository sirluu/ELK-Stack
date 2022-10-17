<h1 align="center">Định tuyến và phân bổ dữ liệu trong Elasticsearch</h1>

# Phần I. Path data
## 1. Data Path
- Elasticsearch lưu trữ dữ liệu data nhận về và thông tin log hoạt động thông qua những đường dẫn lưu trữ data cụ thể.
- Trên từng HDH cài đặt ứng dụng thì hệ thống sẽ có những đường dẫn lưu trữ riêng tùy theo hệ điều hành cụ thể
- Để có thể đảm bảo về dung lượng lưu trữ và tính an toàn dữ liệu có thể thực hiện thay đổi đường dẫn lưu trữ theo ý muốn.

- Định dạng cấu hình trong files `elasticsearch.yml`:
```sh
path.data: /var/data/elasticsearch
path.logs: /var/log/elasticsearch
```
### thực hiện cấu hình multi-path data trên 1 node Elasticsearch
- Các thực cấu hình:
```sh
path.data: 
  - /var/elasticsearch/data1
  - /var/elasticsearch/data2
```
- Tính năng multi-path data trước đây được sử dụng như một giải pháp để mở mở rộng khả năng lưu trữ liệu.
- Hiện nay tính năng này đã ngững hỗ trợ kể từ phiên bản 7.13 do nhận được phản ảnh trái chiều về các hoạt động khi sử dụng tính năng này, kể từ phiên bản 8.0 thực hiện xóa tính năng này
- Đường [link](https://github.com/elastic/elasticsearch/issues/71205) sẽ cho ta biết rõ hợn về vấn đề này
- Giải pháp mở rộng lưu trữ dữ liệu được các nhà sáng chế đưa ra là sử dụng `LVM`
## 2. Cơ chế lưu trữ data
- Trong một cụm Cluster khi dữ liệu được đẩy về từ Logstash sẽ được thông báo đến node master, từ đây node master thực hiên kiểm tra, phân công nhiệm vụ xử lý dữ liệu chuyển về và quyết định dữ liệu logs đó sẽ được lưu trữ tại đâu tùy theo cấu hình hiền thết lập về số lượng primary shard và replicas shard được áp dụng cho index 
- Hệ thống đảm bảo các primary shard và replicas shard của một index sẽ được lưu trữ trên nhiều server khác nhau và cùng ghi dữ liệu đồng thời trên các server đã được master chỉ định
- Khi 1 node trong cụm cluster đạt một múc sử dụng dung lượng disk nhất định không thể ghi thêm dữ liệu vào node, hệ thống sẽ thực hiện dừng ghi dữ liệu trên node đó và đẩy dữ liệu vào 1 node khác. 

<h3 align="center"><img src="../../../../../ELK-Stack/03-Images/dosc/51.png"></h3>

# Phần II. Định tuyến và phân bổ dữ liệu trong cụm
- Shard allocation là tiến trình phân bổ các shard bên trong các node data, được diễn ra trong quá trình recovery, phân bổ replicas, tái cân bằng dữ liệu hoặc trong tường hợp thêm hoặc xóa node trong hệ thống cluster
- Roles Master là role chính trong vấn đề quyết định phân bổ các shard đến các node như thế nào, ở thời điểm di chuyển shard giữa các node cần thực hiện tại cân bằng cụm cluster 
- Một số Phương pháp cấu hình cài đặt kiểm soát phân bổ các shard
  - `Cluster-level shard allocation settings`: Kiểm soát phân bổ và tái cân bằng trong trong cụm
  - `Disk-based shard allocation settings`: Giải thích và thiết lập về không gian lưu trữ dữ liệu
  - `Shard allocation awareness and Forced awareness`: Kiểm soát các phân đoạn có thể phân phối thông qua khu vục khác nhau hay giống nhau
  - `Cluster-level shard allocation filtering`: cho phép các node thuộc group thực hiện phân bổ có chọn lọc để có thể loại bỏ một số node

## 1. Cluster-level shard allocation settings

- **`cluster.routing.allocation.enable`**: Bật hoặc tăt phân bổ  các loại shard riêng biệt
  - `all (dèault)`: cho phép thực hiện phân bổ tất cả các loại shard
  - `primaries`: Chỉ cho phép phân bổ primaries mà không thực hiện phân bổ replicas shrads
  - `new_primaries`: Chỉ cho phép phân bổ các  primary shards của các indices mới
  - `none`: không thực hiện phân bổ shards của các indices dưới bất kỳ hình thức nào

Việc thực hiện cấu hình seting không ảnh hưởng đến sự không phục của Primary shards tại node đó khi thực hiện khởi động lại node. Khi khởi đông lại thành công node thực hiện coppy các primary shard chưa được lưu chỉ định vị trí lưu trữ và thực hiện phục hồi lại các primary shard đó ngay lập tức 

Ví dụ: 
```sh
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable ": all
  }
}
```

- **`cluster.routing.allocation.node_concurrent_incoming_recoveries`**: Cấu hình cho phép thực hiện số lượng phục hổi số lượng shard đầu vào ( thường sẽ là các relicas) đồng thời trên một node. `Default 2`

- **`cluster.routing.allocation.node_concurrent_outgoing_recoveries`**: Cấu hình cho phép thực hiện số lượng phục hổi số lượng shard đi ra bên ngoài đồng thời trên một node. `Default 2`

- **`cluster.routing.allocation.node_concurrent_recoveries`**: cho phép thực hiện cấu hình kết hợp giữa incoming và outgoing trong một dòng config

- **`cluster.routing.allocation.node_initial_primaries_recoveries`**: Cấu hình thực hiện cho phép khôi phục một primary node bằng hính dữ liệu nằm trên local disk. Vì thực hiện trên local nên có thể thục hiện đồng thời một số lượng lớn tại một thời điểm. Default 4

- **`cluster.routing.allocation.same_shard.host`: Nếu `true` cho phép thực hiện sao chép các shard phân bổ  trên cùng 1 node . Default `false`

## 2. Shard rebalancing settings
### Thông tin cơ bản
- Cụm cân bằng khi mà số lượng shard mỗi node bình cân bằng mà không có sự tập trung của shard trên bất node riêng biệt nào trong cụm. Elasticsearch tự động thực hiện gọi đến các tiến trình thực hiện việc tái cân bằng khi di chuyển các shards giữa các node trong cụm cluster để đảm bảo tính cân bằng của cụm
- Thực hiện tái cân bằng được thực hiện dựa trên rule allocation filtering và forced awareness. Việc áp dụng các rule có thể ảnh hưởng đến tính cân bằng dữ liệu giữa các node trong cụm. Đối với trường hợp này cụm sẽ cố gắng tối ứu việc cân bằng nhất có thể
- Đối với trường hợp có các tầng dữ liệu cụ thể giống như data_hot, data_warm,.. Việc thực hiện cân bằng sẽ diễn ra trong các tầng riêng biệt của cụm

### Cấu hình kiểm soát tái cân bằng shards trong Cluster

- **`cluster.routing.rebalance.enable`**: Cho phép thực hiện cần bằng các kiểu shards:
  - `all` (default): Cho phép thực hiện cân bằng cho tất cả các kiểu shards
  - `primaries`: Chỉ cho phép cân bằng các primary shards.
  - `replicas`: Chỉ cho phép cân bằng các replica shards.
  - `none`: Không thực hiện bằng tất cả các kiểu shards của tất cả indices.

- **`cluster.routing.allocation.allow_rebalance`**: Thực hiện chỉ định việc tái cân bằng shard trong cụm
  - always: Luôn luôn
  - indices_primaries_active: Tại thời điểm mà các primaries bên trong Cluster đã được phân bổ thành công
  - indices_all_active ( Default ): Thực hiện khi mà primaries shards and replicas shards đã hoàn thành việc phân bổ

- **`cluster.routing.allocation.cluster_concurrent_rebalance`**: Cho phép thực hiện vận tái cân bằng cùng một thời điểm diễn ra trong cụm. `Defaults 2`

## 3. Disk-based shard allocation settings
- Tính năng phân bổ các shards trên các node đảm bảo các node có đủ dung lượng mà không thực hiện sử dụng mức dung lượng trên mức cần thiết. Khả năng phân bổ dựa trên 1 cặp ngưỡng gọi là `low watermark` và `high watermark`. 
- `low watermark` và `high watermark` có vai trò đảm bảo không cho node nào sử dụng vượt quá ngưỡng ngày hoặc là sự quá ấy chỉ diễn ra một cách tạm thời. Trong trường hợp nếu 1 node có quá mức ngưỡng đã thiết lập thì Elasticsearch sẽ thực hiện di chuyển dữ liệu hoặc shard sang node khác trong cụm đảm bảo sử không vượt ngưỡng đã đặt ra

- Trình phân bổ dữ liệu sẽ cố gắng giữa cho các note không vượt ngưỡng low watermark. Đặc biệt khi tất cả các node vượt qua mức cấu hình low watermark thì sẽ không có bất kỳ shard nào được ghi vào hệ thống và Elasticsearch cũng không thể di chuyển bất kỳ shards nào giữa các node có mức sử dụng đạt high watermark

> Lưu ý: Luôn phải đảm bảo Tổng thể cụm có đủ khả năng lưu trữ và ghi nhận thêm các shards và luôn có một số node đạt mức sử dụng thấp hơn cấu hình low watermark


- Việc di chuyển các shard được kích hoạt bằng trình phân bổ shard để thiết bị lữu trữ cần đáp ứng các quy tác phân bổ  allocation filtering và forced awareness đã được đặt ra trước đó. Trong trường hợp các quy tắc đặt ra quá nghiêm ngặt sẽ ảnh hưởng đến việc di chuyển các shard cần thiết để thực hiện kiểm soát sử dụng ổ đĩa trên các node trong cụm cluster

- Trong trường hợp sử dụng các lớp dữ liệu thì Elasticsearch sẽ chủ động thực hiện cấu hình các quy tắc chọn lọc để các shard trong lớp dữ liệu tích hợp. Đối với mỗi tầng dữ liệu trình phân bổ các shard hoạt động độc lập và không có tác động đến dữ khả năng phân bổ shard trên các tầng dữ liệu khác

- Nếu 1 node mà khả năng nhập dữ liệu nhanh hơn khả năng di chuyển dữ liệu đó sang node khác dẫn đến full data có thể xử lý bằng cách thực hiện cấu hình flood-stage watermark khi đó dữ liệu sẽ không ghi vào node đang gặp vấn đề. Nó sẽ tiếp tục ghi dữ liệu vao cụm khi mức dữ liệu sử dụng trên node bị ảnh hưởng giảm xuống dưới mức high watermark, Elasticsearch sẽ tự động xóa khỏi khối ghi

