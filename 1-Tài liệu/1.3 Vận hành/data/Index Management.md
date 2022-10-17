<h1 align="center">Data: Index Management</h1>

- Các thành phần có trong Index Management
  - Component Templates: thành phần sử dụng cho các index template với mục đích thiết lập cấu hình, với 1 `Component Templates` có thể sử dụng cho nhiều `index Templates`
  - Index Templates: Tempalte khởi tạo các index tự động theo cấu trúc chung sử dụng các Component Templates cho phần thiết lập
  - Data Stream: hỗ trợ lưu trữ dữ lệu theo chuỗi thời gian dựa trên nhiều chỉ số index , có thể khởi tạo từ các index template
  - Indices: danh sách chỉ số file index được khởi tạo thông qua các index template hoặc Data Stream

- Sử dụng định dạng ngôn ngữ  Json để nhập nội dung thiết lập

- Để có một vòng tuần hoàn lưu trữ dữ liệu khép kín tự động cần thực hiền các bước sau:
  - component template: khởi tạo cá thành phần cấu hình lưu trữ sử dụng trong index template
  - Khởi tạo các index lưu trữ dữ liệu dưới dạng data Streams
  - Tạo repo và policy thực hiện chạy Snapshots dữ liệu cho dữ liệu định kỳ
  - Tạo ILM lưu trữ và dặt lịch thực hiện di chuyển và xóa dữ liệu tại các thời điểm sao cho thích hợp
  - Thiết lập logstash output tương ứng với các kiểu dữ liệu được thiết lập tại mục Index template
## 1. component template
- Component Templates được sinh ra để có thể sử dụng chung 1 cấu hình thành phần sử dụng trong 1 index template cho nhiều index tempate khác nhau
- Với mối Component Templates có thể tích hợp hoặc tách biệt các thiết lập về setting, mapping, alias


### 1.1 Setting

- Sử dụng với mục đích thiết lập cấu hình như shards, relicas và các thiết lập liên quan đến lưu trữ và policy cho dữ liệu

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/70.png"></h3>

 - Sau đây là một số thông tin cấu hình phổ biết thường được cấu hình cho các `component template`
   - Shards và Replicas: Cấu hình số lượng các bản lưu chính và sao chép
   ```sh
    {
    "index": {
      "number_of_shards": "2",
      "number_of_replicas": "1"
     }
    }
   ```
   - Index lifecycle Policy: Gán Policy lưu trữ thực hiện di chuyển dữ liệu giữa các node trong cụm theo thiết lập Policy
   ```sh
   {
    "lifecycle": {
     "name": "data-streams-ilm",
     "rollover_alias": "data"
        }
    }
    ```
### 1.2 Mapping template
- Cũng tương tự đối với phần setting Mapping thực hiện ánh xạ đến một thư mục hay một trường dữ liệu

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/71.png"></h3>

- Một số trường dữ liệu thường được sử dụng tri tạo template dùng chung là : @timestamp, host, data_stream 

```sh
 "mappings": {
  "dynamic_templates": [
    {
      "match_ip": {
        "mapping": {
          "type": "ip"
        },
        "match_mapping_type": "string",
        "match": "ip"
      }
    },
    {
      "match_message": {
        "mapping": {
          "type": "match_only_text"
        },
        "match_mapping_type": "string",
        "match": "message"
      }
    },
    {
      "strings_as_keyword": {
        "mapping": {
          "ignore_above": 1024,
          "type": "keyword"
        },
        "match_mapping_type": "string"
      }
    }
  ],
  "date_detection": false,
  "properties": {
    "@timestamp": {
      "type": "date"
    },
    "ecs": {
      "properties": {
        "version": {
          "ignore_above": 1024,
          "type": "keyword"
        }
      }
    },
    "data_stream": {
      "properties": {
        "namespace": {
          "type": "constant_keyword"
        },
        "dataset": {
          "type": "constant_keyword"
        }
      }
    },
    "host": {
      "type": "object"
    }
  }
}
```

### 1.3 Aliases
- Aliases được khởi tạo có nhiệm vụ thực hiện các hành động hỗ trợ việc thêm và xóa các tham số theo ngày tháng. Đối với các hanh động xóa sẽ hỗ trợ các tham số (*)
```sh
{
 "data":{}
}
```
## 2. Index Templates
- Index Templates sử dụng với mục đích tạo nên các form mẫu có chưa các Component Templates hoặc cũng có thể chủ động cấu hình thiết lập thử công các hạn mục setting, mapping và alias áp dụng cho các event gửi đến elasticseach có chưa các Index patterns đã được thiết lập trong các Index Templates
- Trong một Index Templates có thể thực hiện thiết lập cho nhiều Index patterns khác nhau 
- Đối với Index Template thì các index sau khi được xử lý có thể được chia thành 2 kiểu liểu lưu trữ khác nhau đó là `data streams` và `Indices`
  - data streams: thì đây là kiểu dữ liệu lưu trữ theo dòng thời gian nhằm mục dích hỗ trợ người dùng dễ dàng quản lý và tìm kiếm dữ liệu
  - Indices: Đối với kiểu lưu trữ dữ liệu này thì các event được gửi đến có thể được lưu trữ theo các khoảng thời gian cụ thể tuy theo sự thay đổi của tên file Indices có thể theo ngày , tháng hoặc năm

- Tại mục khởi tạo index template gồm có 6 phần chính như hình sau:
<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/72.png"></h3>
  - Mục 1: tại đầy điền các thông tin cơ bản về tên cũng như index patterns sẽ được áp dụng. Ngoài ra có 2 phần khá quan trong đó là Data Streams giuớ lựa chọn để có thể lưu trữ dữ liệu theo luồng thời gian và tiếp là Priority hỗ trợ xác định độ ưu tiên của index template có vị trí bao nhiêu trong tất cả các index template đang quản lý

  - Mục 2: lựa chọn các Component template đã thiết lập trước đó
  - Mục 3,4,5: cho phép thực hiện cấu hình hay custom lại hoặc tạo mới các  thiết lập mà Component template đã tạo trước đó

  - Mục 6: xác nhận và khởi tạo


## 3. Data Streams
- Data Streams cho phép thực hiện lưu trữ dữ liệu theo chuỗi thời gian, có khả năng cập nhật liên tục với phương tăng dần indices ở các file lưu trữ data có chung 1 định dạng.
- Data stream hỗ trợ tương tác trực tiếp đến luồng dữ liệu và tự động định tuyến yêu cầu đến các indices lưu trữ có trong data streams

### indices

<h3 align="center"><img src="../../../../ELK-Stack/03-Images/dosc/73.png"></h3>