<h1 align="center">Configuring Logstash</h1>

- Tài liệu này sẽ tập trung các thức cấu hình cũng như cách sử dụng các Plugin Filter để xử lý dữ liệu
- Các khái niệm cùng mô hình hoạt động có thể tham khảo ở bài viết [sau đây](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/1-T%C3%A0i%20li%E1%BB%87u/1.1%20T%E1%BB%95ng%20Quan/02-C%C6%A1%20ch%E1%BA%BF%20ho%E1%BA%A1t%20%C4%91%E1%BB%99ng%20v%C3%A0%20th%C3%A0nh%20ph%E1%BA%A7n.md#21-logstash)

<h3 align="center">-----------------------------------------</h3>

- Để cấu hình xử lý dữ liêu Logstash, cần thực hiện tạo và cấu hình file sử dụng các plugin. Có thể sử dụng các event fields cấu hình và điều kiện để kiểm tra và xử lý các event theo điều kiện đặt ra.
- Thực hiện tạo 1 file config mẫu có tên là `logstash.conf`:
```sh
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```
- Để có thể kiểm tra file cấu hình đã chính xác hay chưa sử dụng câu lệnh `-f` đi kèm với tên file
```sh
bin/logstash -f logstash-simple.conf
```

Theo nội dung file cấu hình bên trên thì logstash sẽ đọc nội dung file được cấu hình chỉ định sau đó sẽ xuất nội dung ra elasticsearch sử dụng Port 9200 (default) và stdout



# Phần I. Cấu trúc file config

- Logstash file config có từng phần  cấu hình riêng biệt đối với mỗi kiểu plugin riêng biệt để thêm vào tiến trình xử lý sự kiện
```sh
# This is a comment. You should use comments to describe
# parts of your configuration.
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```

- Trong cùng 1 file config nếu sử dụng nhiều Filter thì các Filter sẽ được xử lý một cách tuần tự theo nội dung có bên trong file

```sh
input {
  file {
    path => "/var/log/messages"
    type => "syslog"
  }

  file {
    path => "/var/log/apache/access.log"
    type => "apache"
  }
}
```

