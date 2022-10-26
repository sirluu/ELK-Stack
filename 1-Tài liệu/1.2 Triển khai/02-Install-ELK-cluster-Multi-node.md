<h1 align="center">Tài liệu xây dựng hệ thống ELK-Stack Cluster cơ bản </h1>

# Phần I. Tổng quan
## 1. Mô hình hệ thống
### -  Mô hình luồng xử lý dữ liệu


<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/46.png"></h3>

- Mội thành phần xử lý dữ liệu hay điều hướng các request đều được hoạt động dưới hình thức HA nhằm mục đích đảm bảo tính ổn định, tăng hiệu năng xử lý của hệ thống.

- Để có thể kết nối đến các thành phần trong một cụm sẽ sử dụng IP VIP để thực hiện kết nối và chia tải hệ thống


### -  Mô hình triển khai thiết bị vật lý
<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/47.png"></h3>

- Hệ thống hoạt động dựa trên mô hình phân lớp, các lớp giao tiếp với nhau thông qua 1 địa chỉ IP VIP thể thực hiện điều chỉnh HA các request gửi đến hệ thống
- Quý tắc đăt roles cho cụm `Elastcisearch`:
  - Tối thiểu 1 cụm cần có các thành phần: Master, Data, ingest, coordinating
  - Cần thực hiện chạy Cluster đối với các role
  - Trên một server vật lý cài đặt Elasticsearch có thể thực hiện đặt nhiều role khác nhau dẫn điến một cụm ELK-cluster có thể có số server chẵn nhưng số lượng server sử dụng cho 1 role luôn lẻ nhằm thỏa mãn công thức về số node hoạt động là (N/2)+1

- VLan:
  - Pubic 192.168.70..x/24: Sử dụng để management các server, tiếp nhận dữ liệu đẩy vào và đua dữ liệu ra bên ngoài thông qua kibana đến với người dùng
  - Prvate 10.10.10.x/24: Sủ dụng để các node trong hệ thống thưc hiện giao tiếp và xử lý dữ liệu 

## 2. Thành phần chức năng các node có trong mô hình
- Hộ thống được xậy dựng dựa trên 8 node trong đó 
  - 5 node sử dụng cho Elasticsearch bao gồm 3 node dùng cho master và lưu trữ data_hot và 2 node còn lại sử dụng cho role data_warm, data_cold.
  - 3 node chạy các thành phần kibana,logstash và elasticsearch với role: Coordinating để phân chia và điều hướng các request gửi đến cụm Elasticsearch
### `Node ELK-master01`
- Hosname: elk-master01
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.50/24
- IP Private: 10.10.10.10/24
- Container: elk-master01
- Role: master, data_hot, data_content, ingest

### `Node ELK-master02`
- Hosname: elk-master02
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.51/24
- IP Private: 10.10.10.11/24
- Container: elk-master02
- Role: master, data_hot, data_content, ingest


### `Node ELK-master03`
- Hosname: elk-master03
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.52/24
- IP Private: 10.10.10.12/24
- Container: elk-master03
- Role: master, data_hot, data_content, ingest, voting_only

### `Node ELK-service01`
- Hosname: elk-service01
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.53/24
- IP Private: 10.10.10.13/24
- Container1: elk-coordinating01
- Container2: elk-logstash01
- Container3: elk-kibana01
- Container4: keepalived-haproxy01

### `Node ELK-service02`
- Hosname: elk-service02
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.54/24
- IP Private: 10.10.10.14/24
- Container1: elk-coordinating02
- Container2: elk-logstash02
- Container3: elk-kibana02
- Container4: keepalived-haproxy02


### `Node ELK-service03`
- Hosname: elk-service03
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.55/24
- IP Private: 10.10.10.15/24
- Container1: elk-coordinating03
- Container2: elk-logstash03
- Container3: elk-kibana03
- Container4: keepalived-haproxy03

### `Node ELK-data01`
- Hosname: elk-data01
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.56/24
- IP Private: 10.10.10.16/24
- Container1: elk-coordinating03
- Container2: elk-logstash03
- Container3: elk-kibana03
- Container4: keepalived-haproxy03

### `Node ELK-data02`
- Hosname: elk-data02
- Hardware: 4CPU-6GB Ram
- IP Public: 192.168.70.56/24
- IP Private: 10.10.10.15/24
- Container1: elk-coordinating03
- Container2: elk-logstash03
- Container3: elk-kibana03
- Container4: keepalived-haproxy03

# Phần II. Triển khai cài đặt
## 1. Thiết lập cấu hình cơ bản
>### **`1. Trên tất cả các node`**
### 1.1 Cài đặt Docker-Compose
- Thực hiện update OS:
```sh
sudo apt-get update -y
```
- Cài đặt các gói ràng buộc
```sh
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```
- Adding Docker’s GPG Key

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

- Cài đặt Docker
```sh
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
sudo apt update
sudo apt-get install docker-ce -y
```
- Kiểm tra version docker cài đặt:
```sh
docker --version
kết quả:
Docker version 20.10.16, build aa7e414
```
- Cài đặt Docker Compose
```sh
apt-get install docker-compose -y
```
- kiểm tra version docker compose sẽ thực hiện cài đặt
```sh
$ apt-cache madison docker-compose-plugin
docker-compose-plugin | 2.5.0~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
docker-compose-plugin | 2.3.3~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```
- cài đặt docker compose version 2.5.0
```sh
sudo apt-get install docker-compose-plugin=2.5.0~ubuntu-focal
```
- Kiểm tra phiên bản docker compose sau khi cài đặt
```sh
$  docker compose version
Docker Compose version v2.5.0
```

### 1.2 Điểu chỉnh bộ nhớ ảo
- Mmapfs lưu trữ các shard index trên file system (ánh xạ đến Lucene MMapDirectory) bằng cách ánh xạ các file đó vào ram ảo (mmap). Việc ánh xạ bộ nhớ thường sẽ được thiết lập có size bằng với size file được ảnh xạ, trong trường hợp ở đây là JVM heap được cấu hình sử dụng cho Elastich và được gán giá trị trong file setup

- Theo quy tắc chung đặt ra số ram đặt cho JVM heap là 50% trên tổng ram vật lý trên 1 server, ở thời điểm hiện tại số ram tối đa mà JVM heap có thể hoạt động ổn định và hiệu quả là 32GB

- - Elasticsarch được viết bằng ngôn ngữ lập trình java, theo mặc định thì Elasticsarch sẽ gọi đến JVM để hoạt động và JVM cần sử dụng ram cho caching

Điều chỉnh số lượng mmap để tránh trường hợp hết bộ nhớ ảo
- Công thực tính: 
```sh
# Cấu hình thiết lập
vm.max_map_count=map_count
# mỗi 1 đơn vị map_count thường có giá trị 128kb bộ nhớ hệ thống
# Hiện tại mức ram hỗ trợ tối có thể sử dụng đối với JVM là 32GB nên có Công thực tính như sau:
map_count= (1024 * 1024 * 32)/128 = 262144
- trong đó: 
 - (1024 * 1024 * 32) là giá trị quy đổi từ GB sang kb
 - 128 là giá trị tương ứng với 1 đơn vị map_count có đơn vị tính kb
```

- cập nhật thông số trong file: `/etc/sysctl.conf`
```sh
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
```
- kiểm tra lại:
```sh
sysctl -p
```
- Với thông số thiết lập như trên thì mức cấu hình max là 262144 và nhỏ nhất dựa vào cấu hình hệ thống

### Tăng giới hạn mô tả file đang mở

- Elasticsearch sử dụng rất nhiều file descriptors/file handles (connections [open ports], read/write files…) nên cần nofile đủ lớn. Con số 1024 (2^10) mà Linux system set default cho mỗi process không đủ dùng nên cần con số lớn hơn. 
- Do source/destination ports trong TCP header chỉ dùng tới 16 bits nên con số maximum 65536 = 2^16 từ đây mà ra, port chỉ có thể nằm trong range [0, 2^16 - 1]

- Thay đổi giá trị trong file `/etc/security/limits.conf`
```sh
echo "- nofile 65536" >> /etc/security/limits.conf
```
### Cập nhật thông tin các node trong file host
```sh
echo "# List ip node elk Cluster
10.10.10.10 elk-master01
10.10.10.11 elk-master02
10.10.10.12 elk-master03
10.10.10.13 elk-coordinating01 elk-kibana01
10.10.10.14 elk-coordinating02 elk-kibana02
10.10.10.15 elk-coordinating03 elk-kibana03
10.10.10.16 elk-data01
10.10.10.17 elk-data02
10.10.10.20 IPVIP-local
192.168.70.63 IPVIP-public
" >> /etc/hosts
```

### 1.3 Tạo và phân quyền thư mục lưu trữ data và logs
```sh
mkdir -p /elasticsearch/ /elasticsearch/data/ /elasticsearch/logs/ /elasticsearch/certs/ /elasticsearch/snapshots/
chown -R 1000:1000 /elasticsearch/ /elasticsearch/data/ /elasticsearch/logs/ /elasticsearch/certs/ /elasticsearch/snapshots/
mkdir -p /elk-setup && cd /elk-setup

```

### 1.4 Allow Port firewalld

```sh
sudo ufw allow 9200
sudo ufw allow 9300
```
### 1.5 Tạo file `.env` chứa các thông tin thiết lập hệ thống
- Nội dung:
```sh
echo "# Phiên bản ELK Stack sử dụng để cài đặt
ELK_VERSION=7.17.5
# Set the cluster name
CLUSTER_NAME=elk-cluster
# Password user elastic sử dụng đăng nhập elasticsearch và kibana
PASS_ELASTIC=Password2022
PASS_KIBANA=Password2022
# Port to expose Elasticsearch HTTP API to the host
ES_PORT=9200
# Port to expose Elasticsearch transpost API to the host
ES_TRANSPORT_PORT=9300
# Port to expose Kibana to the host
KIBANA_PORT=5601
# Port to expose LOGSTASH to the host
LOGSTASH_PORT=5000
# Path
ES_DATA=/elasticsearch/data/
ES_LOGS=/elasticsearch/logs/
CERTS_DIR=/elasticsearch/certs/
CERTS_DIR_CONTAINER=/usr/share/elasticsearch/config/certs
KIBANA_DIR=/elasticsearch/kibana/
# Set to 'basic' or 'trial' to automatically start the 30-day trial
LICENSE=basic
# repo snapshot
PATH_REPO=/elasticsearch/snapshots/
" >> .env
```
### 1.6 Cài đặt một số gói packet cần thiết
```sh
apt-get install -y zip unzip
```

>### Lưu ý: Thực hiện cấu Hình sử dụng định dạng `LVM` và mount vào các thư mục lưu trữ data

>### **`2. Trên các node service`**

### 2.1 Allow thêm các port
```sh
sudo ufw allow 9200
sudo ufw allow 5601
sudo ufw allow 8080
sudo ufw allow 5000
```
### 2.2 Tạo thư mục lư trữ data cho kibana
```sh
mkdir -p /kibana/ && chown -R 1000:1000 /kibana
```

### 2.3 thiết lập cho phép sử dụng IP HA-proxy

```sh
echo 'net.ipv4.ip_nonlocal_bind = 1' >> /etc/sysctl.conf
```

## 2. Cài đặt hệ thống ELK Cluster 
>### **`1. Trên các node master`**

### 2.1: ELk-Master01
- Truy cập thư mục **`/elk-setup`** để thực hiện các bước tiếp theo
```sh
cd /elk-setup
```
**Chuẩn bị các file setup**
- Tạo file generation SSL có tên `create-certs.yml` có [nội dung](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/01-elk-master01/create-certs.yml)

- Tạo file config `elasticsearch-elk-master01.yml` sử dụng cho elasticsearch nội dung [tại đây](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/01-elk-master01/elasticsearch-elk-master01.yml)

- Tạo file setup `docker-compose.yml` nội dung [tại đây](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/01-elk-master01/docker-compose.yml)


- Kiểm tra danh sách file: file `.env` được khởi tạo ở bước cấu hình chung
```sh
root@elk-master:~/ELK-Stack/master/02-Multi-node/01-elk-master01# ls -alh
total 24K
drwxr-xr-x  2 root root 4.0K Jul  5 10:18 .
drwxr-xr-x 10 root root 4.0K Jul  5 09:54 ..
-rw-r--r--  1 root root  824 Jul  5 09:54 .env
-rw-r--r--  1 root root 2.0K Jul  5 09:56 create-certs.yml
-rw-r--r--  1 root root  820 Jul  5 09:54 docker-compose.yml
-rw-r--r--  1 root root 1.4K Jul  5 09:54 elasticsearch-elk-master01.yml
root@elk-master:~/ELK-Stack/master/02-Multi-node/01-elk-master01#
```

**Tạo chứng chỉ SSL Self-sign**



- Thực hiện lệnh :
```sh
root@elk-master:~/elk-setup# docker-compose -f create-certs.yml run --rm create-certs
Creating network "01-elk-master01_default" with the default driver
Creating CA
Archive:  config/certs/ca.zip
   creating: config/certs/ca/
  inflating: config/certs/ca/ca.crt  
  inflating: config/certs/ca/ca.key  
Creating certs
Archive:  config/certs/certs.zip
   creating: config/certs/elasticsearch/
  inflating: config/certs/elasticsearch/elasticsearch.crt  
  inflating: config/certs/elasticsearch/elasticsearch.key  
Setting file permissions
root@elk-master:~/elk-setup#
```

- Truy cập thư mục `/elasticsearch/certs/` kiểm tra kết quả:
```sh
root@elk-master:/elasticsearch/certs# ll
total 28
drwxr-x--- 4 1000 root 4096 Jul  3 19:52 ./
drwxr-xr-x 6 1000 1000 4096 Jul  1 10:08 ../
drwxrwxr-x 2 1000 root 4096 Jul  3 19:52 ca/
-rw------- 1 1000 root 2521 Jul  3 19:52 ca.zip
-rw------- 1 1000 root 2834 Jul  3 19:52 certs.zip
drwxrwxr-x 2 1000 root 4096 Jul  3 21:15 elasticsearch/
-rw-rw-r-- 1 1000 root  534 Jul  3 19:52 instances.yml
root@elk-master:/elasticsearch/certs#
```
- Thực hiện gửi các file .zip đến các node trong hệ thống
```sh
scp /elasticsearch/certs/* root@10.10.10.11:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.12:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.13:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.14:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.15:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.16:/elasticsearch/certs/
scp /elasticsearch/certs/* root@10.10.10.17:/elasticsearch/certs/
```

> Lưu ý: chứng chỉ SSL chỉ khởi tạo trên node master và chuyển đi các node khác

**Cài đặt Elasticsearch**

- chạy lệnh setup
```sh
docker compose up -d
```
- kiểm tra kết quả:
```sh
root@elk-master:~/elk-setup# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master01:9200'
{
  "name" : "elk-master01",
  "cluster_name" : "elk-cluster",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.17.5",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d61b4f7ddf931f219e3745f295ed2bbc50c8e84",
    "build_date" : "2022-06-23T21:57:28.736740635Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@elk-master:~/elk-setup#
```


### 2.2: ELk-Master02
**Giả nén SSL**

```sh
root@elk-master02:~# cd /elasticsearch/certs/ 
root@elk-master02:/elasticsearch/certs# ll
total 20
drwxr-xr-x 2 1000 1000 4096 Jul  5 10:23 ./
drwxr-xr-x 6 1000 1000 4096 Jul  2 10:53 ../
-rw------- 1 root root 2511 Jul  5 10:23 ca.zip
-rw------- 1 root root 2804 Jul  5 10:23 certs.zip
-rw-r--r-- 1 root root  523 Jul  5 10:23 instances.yml
root@elk-master02:/elasticsearch/certs# unzip ca.zip 
Archive:  ca.zip
   creating: ca/
  inflating: ca/ca.crt               
  inflating: ca/ca.key               
root@elk-master02:/elasticsearch/certs# unzip certs.zip
Archive:  certs.zip
   creating: elasticsearch/
  inflating: elasticsearch/elasticsearch.crt  
  inflating: elasticsearch/elasticsearch.key  
root@elk-master02:/elasticsearch/certs#
```
**Chuẩn bị các file setup**
- Truy cập thư mục **`/elk-setup`** để thực hiện các bước tiếp theo
```sh
cd /elk-setup
```
- Thực hiện Cài đặt Elasticsearch tương tự đối với node `ELk-Master01` với các file sau:
  - Nội dung file [elasticsearch-elk-master02.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/02-elk-master02/elasticsearch-elk-master02.yml)

  - Nội dung file [docker-compose.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/02-elk-master02/docker-compose.yml)

 - Kiểm tra danh sách file:
 ```sh
 root@elk-master02:~/elk-setup# ll -alh
total 56K
drwxr-xr-x  2 root root 4.0K Jul  5 10:41 ./
drwxr-xr-x 10 root root 4.0K Jul  5 10:37 ../
-rw-r--r--  1 root root  824 Jul  5 10:37 .env
-rw-r--r--  1 root root  819 Jul  5 10:37 docker-compose.yml
-rw-r--r--  1 root root 1.4K Jul  5 10:37 elasticsearch-elk-master02.yml
root@elk-master02:~/elk-setup#
```
**Cài đặt Elasticsearch**

- chạy lệnh setup
```sh
docker compose up -d
```
- kiểm tra kết quả:
```sh
root@elk-master02:~/elk-setup# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master02:9200'
{
  "name" : "elk-master02",
  "cluster_name" : "elk-cluster",
  "cluster_uuid" : "G_kv-XzfTMOUNMLZ42jUKw",
  "version" : {
    "number" : "7.17.5",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d61b4f7ddf931f219e3745f295ed2bbc50c8e84",
    "build_date" : "2022-06-23T21:57:28.736740635Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@elk-master02:~/elk-setup#
```


### 2.3: ELk-Master03
**Giả nén SSL**
```sh
cd /elasticsearch/certs/ 
unzip ca.zip
unzip certs.zip
```

**Tạo file setup và config**
- Truy cập thư mục **`/elk-setup`** để thực hiện các bước tiếp theo
```sh
cd /elk-setup
```
- Thực hiện Cài đặt Elasticsearch tương tự đối với node `ELk-Master01` và `ELk-Master02`
- Nội dung file [elasticsearch-elk-master03.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/03-elk-master03/elasticsearch-elk-master03.yml)

- Nội dung file [docker-compose.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/03-elk-master03/docker-compose.yml)

- Kiểm tra danh sách file:
```sh
root@elk-master03:~/elk-setup# ll -alh
total 20
drwxr-xr-x  2 root root 4096 Jul  5 11:24 ./
drwxr-xr-x 10 root root 4096 Jul  5 11:24 ../
-rw-r--r--  1 root root  824 Jul  5 11:24 .env
-rw-r--r--  1 root root  819 Jul  5 11:24 docker-compose.yml
-rw-r--r--  1 root root 1436 Jul  5 11:24 elasticsearch-elk-master03.yml
root@elk-master03:~/elk-setup#
```

**Cài đặt Elasticsearch**
```sh
docker compose up -d
```
- Kiểm tra
```sh
root@elk-master03:~/elk-setup# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master02:9200'
{
  "name" : "elk-master03",
  "cluster_name" : "elk-cluster",
  "cluster_uuid" : "G_kv-XzfTMOUNMLZ42jUKw",
  "version" : {
    "number" : "7.17.5",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d61b4f7ddf931f219e3745f295ed2bbc50c8e84",
    "build_date" : "2022-06-23T21:57:28.736740635Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@elk-master03:~/elk-setup#
```

>### **`1. Trên các node Service`**
### 2.4: ELk-Service01
**Giả nén SSL và tạo file .pem cho HAproxy**
```sh
cd /elasticsearch/certs/ 
unzip ca.zip
unzip certs.zip
cd elasticsearch
cat elasticsearch.key elasticsearch.crt > elasticsearch.pem
```
**Tạo file setup và config**
- Truy cập thư mục **`/elk-setup`** để thực hiện các bước tiếp theo
```sh
cd /elk-setup
```

**`elk-coordinating01`**
- Tạo file config `elasticsearch-elk-coordinating01.yml` sử dụng cho elasticsearch có [nội dung](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/04-elk-service01/elasticsearch-elk-coordinating01.yml)

**`elk-kibana01`**

- Tạo file config `elk-kibana01.yml`. Kibana sẽ trỏ vào Cụm Elasticsearch để đọc dữ liệu thông qua IP VIP được cấu hình để chạy HA cho 3 node `coordinating` có [nội dung](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/04-elk-service01/elk-kibana01.yml)

**`elk-logstash01`**
- Tạo thư mục lưu trữ file config và file pipeline xử lý dữ liệu logs đẩy về
```sh
mkdir -p logstash 
mkdir -p logstash/config 
mkdir -p logstash/pipeline 
```
- Tạo file config `logstash.yml`: 
```sh
# vi logstash/config/logstash.yml
- Nội dung:
---
## Default Logstash configuration from Logstash base image.
## https://github.com/elastic/logstash/blob/master/docker/data/logstash/config/logstash-full.yml
#
http.host: "0.0.0.0"
path.config: /usr/share/logstash/pipeline
```

- Tạo file pipeline nhận dự liệu được gửi đến thông qua port 5000 và sau khi xử lý dữ liệu sẽ trả dữ liệu về cụm `Elasticsearch` 

```sh
# vi logstash/pipeline/message.yml
input {
  beats {
    port => 5000
  }
}
filter {
  grok {
    match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    add_field => [ "received_at", "%{@timestamp}" ]
    add_field => [ "received_from", "%{host}" ]
    add_tag => ["message"]
    remove_tag => [ "beats_input_codec_plain_applied" ]
  }
  date {
  match => [ "syslog_timestamp", "MMM d HH:mm:ss", "MMM dd HH:mm:ss" ]
  }
}
output {
  elasticsearch {
    hosts => [ "https://IPVIP-local:9201" ]
      user => "elastic"
      password => "Password2022"
      ssl => true
      ssl_certificate_verification => false
      cacert => "/usr/share/logstash/certs/ca/ca.crt"
      index => "elk-cluster-%{+YYYY.MM.dd}"
    }
  stdout { codec => rubydebug }
}
```

> Lưu ý: Pipeline sử dụng `IPVIP-local` để trỏ đến cụm elasticsearch


**`keepalived-haproxy01`**

- Container này sử dụng để khởi tạo IP VIP và xử lý HA cho các hệ thống Redis-cache cũng như ELk-CLuster
- Tạo thư mục lưu trữ file cấu hình `keepalived.conf` và `haproxy.cfg`, nội dung file config [tại đây](https://github.com/thang290298/Ghi-chep-Logs/tree/main/ELK-Stack/master/2-Source/02-Multi-node/04-elk-service01/keepalived-haproxy)
```sh
mkdir -p keepalived-haproxy/ keepalived-haproxy/keepalived keepalived-haproxy/haproxy
```

**Cài đặt service**
```sh
docker compose up -d
```
Bước 4: kiểm tra kết quả:

```sh
root@elk-service01:/elk-cluster-setup# docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS          PORTS     NAMES
cdee928e6bc7   pelin/haproxy-keepalived:v1.0.0                        "/haproxy-keepalived"    27 minutes ago   Up 37 seconds             keepalived-haproxy01
792bc972a8ca   docker.elastic.co/logstash/logstash:7.17.5             "/usr/local/bin/dock…"   27 minutes ago   Up 28 seconds             elk-logstash01
119753b37e7b   docker.elastic.co/elasticsearch/elasticsearch:7.17.5   "/bin/tini -- /usr/l…"   27 minutes ago   Up 36 seconds             elk-coordinating01
bdeb2ea36eb1   docker.elastic.co/kibana/kibana:7.17.5                 "/bin/tini -- /usr/l…"   27 minutes ago   Up 28 seconds             elk-kibana01
root@elk-service01:/elk-cluster-setup#
```

**kiểm tra các dịch vụ**
- haproxy-keepalived: Do IP VIP mới được thiết lập trên 1 node dẫn đến ở thời điểm hiện tại node này sẽ là được gán IP VIP cho dải Public và Private
<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/66.png"></h3>

- elk-coordinating01
```sh
root@elk-service01:~/elk-cluster-setup# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-coordinating01:9200'
{
  "name" : "elk-coordinating01",
  "cluster_name" : "elk-cluster",
  "cluster_uuid" : "G_kv-XzfTMOUNMLZ42jUKw",
  "version" : {
    "number" : "7.17.5",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d61b4f7ddf931f219e3745f295ed2bbc50c8e84",
    "build_date" : "2022-06-23T21:57:28.736740635Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@elk-service01:~/elk-cluster-setup#
```

- kibana01 sử dụng IP VIP: https://192.168.70.63:5602
<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/67.png"></h3>

### 2.5: ELk-Service02
**Chuần bị**
- Thực hiện tương tự node `elk-service01` :
- Nội dung files tương ứng
  - Cấu hình config [keepalived-haproxy](https://github.com/thang290298/Ghi-chep-Logs/tree/main/ELK-Stack/master/2-Source/02-Multi-node/05-elk-service02/keepalived-haproxy)
  - Nội dung cấu hình `config` và `pipeline` sử dụng cho [logstash](https://github.com/thang290298/Ghi-chep-Logs/tree/main/ELK-Stack/master/2-Source/02-Multi-node/05-elk-service02/logstash)
  - file config [elasticsearch-elk-coordinating02.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/05-elk-service02/elasticsearch-elk-coordinating02.yml)
  - file config [elk-kibana02.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/05-elk-service02/elk-kibana02.yml)
  - file config [docker-compose.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/05-elk-service02/docker-compose.yml)

**Cài đặt service**
```sh
docker compose up -d
```
**kiểm tra service**
- Kiểm tra trạng thái container
```sh
root@elk-service02:/elk-cluster-setup# docker ps -a
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS          PORTS     NAMES
b4f842c7aa7c   pelin/haproxy-keepalived:v1.0.0                        "/haproxy-keepalived"    20 seconds ago   Up 17 seconds             keepalived-hapro
888829ac5811   docker.elastic.co/logstash/logstash:7.17.5             "/usr/local/bin/dock…"   20 seconds ago   Up 16 seconds             elk-logstash02
a769d8cca586   docker.elastic.co/elasticsearch/elasticsearch:7.17.5   "/bin/tini -- /usr/l…"   20 seconds ago   Up 16 seconds             elk-coordinating
226f8eeb2bde   docker.elastic.co/kibana/kibana:7.17.5                 "/bin/tini -- /usr/l…"   20 seconds ago   Up 16 seconds             elk-kibana02
root@elk-service02:/elk-cluster-setup#
```
- haproxy-keepalived: khi node service02 hoàn thành setup IP VIP local sẽ được chuyển về node này do được cấu hình làm master trong file config

<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/68.png"></h3>

- elk-coordinating02
```sh
root@elk-service01:~/elk-cluster-setup# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-coordinating01:9200'
{
  "name" : "elk-coordinating02",
  "cluster_name" : "elk-cluster",
  "cluster_uuid" : "G_kv-XzfTMOUNMLZ42jUKw",
  "version" : {
    "number" : "7.17.5",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "8d61b4f7ddf931f219e3745f295ed2bbc50c8e84",
    "build_date" : "2022-06-23T21:57:28.736740635Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@elk-service01:~/elk-cluster-setup#
```
### 2.6: ELk-Service03

**Chuần bị**
- Cấu trúc file và thao tác thực hiện tương tự node `elk-service01` và `elk-service02`

- Nội dung file setup và config:
  - Cấu hình config [keepalived-haproxy](https://github.com/thang290298/Ghi-chep-Logs/tree/main/ELK-Stack/master/2-Source/02-Multi-node/06-elk-service03/keepalived-haproxy)
  - Nội dung cấu hình `config` và `pipeline` sử dụng cho [logstash](https://github.com/thang290298/Ghi-chep-Logs/tree/main/ELK-Stack/master/2-Source/02-Multi-node/06-elk-service03/logstash)
  - file config [elasticsearch-elk-coordinating02.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/06-elk-service03/elasticsearch-elk-coordinating03.yml)
  - file config [elk-kibana02.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/06-elk-service03/elk-kibana03.yml)
  - file config [docker-compose.yml](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/06-elk-service03/docker-compose.yml)

**Cài đặt service**
```sh
docker compose up -d
```
**kiểm tra service**

- Thực hiển các bước kiểm tra tương tự với 2 node bên trên

>### **`1. Trên các node data`**

- Các Node data thực hiện tương tự đối với các node master 

### 2.7: ELk-data01

**Giả nén SSL**
```sh
cd /elasticsearch/certs/ 
unzip ca.zip
unzip certs.zip
```
**Chuẩn bị các file setup**

- Truy cập thư mục **`/elk-setup`** để thực hiện các bước tiếp theo
```sh
cd /elk-setup
```
- Tạo file config `elasticsearch-elk-data01.yml` sử dụng cho elasticsearch có [nội dung](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/07-elk-data01/elasticsearch-elk-data01.yml)

- Tạo file setup `docker-compose.yml` nội dung [tại đây](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/07-elk-data01/docker-compose.yml)

- chạy lệnh setup
```sh
docker compose up -d
```
**Kiểm tra**



### 2.8: ELk-data02

- Tạo file config `elasticsearch-elk-data02.yml` sử dụng cho elasticsearch có [nội dung](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/07-elk-data02/elasticsearch-elk-data02.yml)

- Tạo file setup `docker-compose.yml` nội dung [tại đây](https://github.com/thang290298/Ghi-chep-Logs/blob/main/ELK-Stack/master/2-Source/02-Multi-node/07-elk-data02/docker-compose.yml)

- chạy lệnh setup
```sh
docker compose up -d
```

# Phần III. Kiểm tra hệ thống sau khi cài đặt
- sủ dụng IP VIP
### 1. Elasticsearch
-  Kiểm tra danh sách các node trong cụm sau khi đã cài đặt
```sh
root@elk-master03:~# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master03:9200/_cat/nodes?v'
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.10.10.11           68          78   0    0.58    0.59     0.56 hims      -      elk-master02
10.10.10.16           32          59   0    0.00    0.04     0.12 clrtw     -      elk-data01
10.10.10.12           42          65   0    0.28    0.07     0.02 himsv     -      elk-master03
10.10.10.17           28          65   0    0.26    0.36     0.27 clrtw     -      elk-data02
10.10.10.14           19          91   3    0.35    0.24     0.21 -         -      elk-coordinating02
10.10.10.10           44          95   1    0.82    0.86     0.82 hims      *      elk-master01
10.10.10.13           30          98   1    0.08    0.06     0.07 -         -      elk-coordinating01
10.10.10.15            8          81   2    0.04    0.05     0.09 -         -      elk-coordinating03
root@elk-master03:~#
```

- Kiểm tra thông tin node master:
  - Master:
```sh
root@elk-master03:~# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master03:9200/_cat/master?v'
id                     host        ip          node
iNVEG-hmTgeYYkb_HYnFkw 10.10.10.10 10.10.10.10 elk-master01
root@elk-master03:~#
```


- Kiểm tra trạng thái cụm:
```sh
root@elk-master03:~# curl -s -XGET POST --cacert /elasticsearch/certs/ca/ca.crt -u elastic:Password2022 'https://elk-master03:9200/_cluster/health?pretty'{
  "cluster_name" : "elk-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 8,
  "number_of_data_nodes" : 5,
  "active_primary_shards" : 13,
  "active_shards" : 26,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
root@elk-master03:~#
```


### 1. Kibana
- Link: http://192.168.70.63:5602

<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/69.png"></h3>

### HAproxy

- Link: http://192.168.70.63:8080/stats

<h3 align="center"><img src="https://raw.githubusercontent.com/sirluu/ELK-Stack/master/03-Images/dosc/50.png"></h3>
