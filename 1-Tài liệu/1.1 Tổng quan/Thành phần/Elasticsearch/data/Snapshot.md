<h1 align="center">Elasticsearch Snapshot - Retore</h1>

# Phần I. Tổng quan
- Elasticsearch không sử dụng tính năng backup thay vào đó sẽ sử dụng tính năng Snapshot để tạo ra các bản sao lưu dữ liệu tùy vào từng thời điểm nhất định dễn ra một cách định kỳ.
- Snapshot là phương pháp hữu hiệu nhất khi sử dụng để thực hiện sao lưu dữ liệu của 1 cụm. Vì không thế thực hiện backup thư mục và dữ liệu elasticsearch của từng node để rồi thực hiện restore lên hệ thống Elasticsearch. Khi thực hiện như thế không cần sẽ phát sinh ra lỗi thiếu file dẫn đến restore không thành công gây ra lỗi hệ thống 

- Snapshot là dữ liệu coppy của cụm Elasticserch cluster, thường được sử dụng cho những mục đích sau:
  - Sao lưu dữ liệu cụm elasticsearch đình kỳ hàng ngày, hàng tuần, hàng tháng, hàng năm
  - Đảm bảo tính an toàn dữ liệu 
  - Giảm dung lượng lưu trữ dữ liệu và tạo không gian lưu trữ dữ mới
  - Khôi phục dữ liệu khi cần thiết
  - hỗ trợ di chuyển dữ liệu giữa các cụm cluster
  - Giảm thiểu chi phí lưu trữ dữ liệu bằng cách sử dụng Searchable snapshots trong các lớp data_hot và data_warm

- Elasticsearch hỗ trợ người dùng lưu trữ dữ liệu ra nhiều loại kho lưu trữ khác nhau khác nhau để lưu trữ dữ liệu:
  - Shared file system
  - Read-only URL
- Ngoài việc hỗ trợ lưu trữ  dữ liệu trên các thiết bị lưu trữ, elasticsearch hỗ trợ lưu trữ trên các điện toán đám mây, cloud như là:
  - AWS S3
  - Google Cloud Storage (GCS)
  - Microsoft Azure

> Lưu ý: Dữ liệu snapshot sẽ được lưu trữ đồng bộ trên tất cả các node được thiết lập repo cho lưu trữ snapshot
# Phần II. Luồng hoạt động của Snapshot
Để có thể tạo và lưu trữ dữ liệu snapshot từ Elasticsearch cần thực hiện thực hiện các bước sau:
- b1: khởi tạo Repo lưu trữ (kho lưu trữ ): Sử dụng để lưu trữ dữ liệu snapshot
- b2: thiết lập cấu hình `path.repo` trong file `elasticsearch.yml` để thiết lập cho phép lưu trữ trên các node cần thiết
- b3: Khởi tạo Repository, với mỗi Repository có thể gắn 1 repo lưu trữ riêng biêt tùy vào mưc đích sử dụng cho các repo đó
- b4: Thiết lập Policy đặt lịch backup đối với các index theo các mốc thời gian cụ thể


Có thể gắn các policy snapshot vào các `Index Lifecycle Policies`

# Phần III. Nội dung lưu trữ và cách thức hoạt động bên trong snapshot

- Đối với 1 bản snapshot mặc định sẽ lưu trữ tất cả dư liệu về trạng thái, data stream, indices trên cụm elasticsearch:
  - Persistent cluster settings
  - index template
  - Legacy index templates
  - Ingest pipelines
  - ILM policies

- Ngoài việc dữ snapshot all data thì có thể thực hiện snapshot các indeces , data stream chỉ động. Tuy nhiên một số indeces mặc định của hệt thống sẽ đươc mặc định được thực hiện snapshot trên tất cả các bản snapshot

- những nội dung không được thực hiện lưu trữ bên trong các bản snapshot:
  - Cấu hình cài đặt tạm thời
  - File cấu hình thiết lập trên các node
  - file cấu hình hình bảo mật


## 1. Cách thức hoạt động

- Snapshot hoạt động có khả năng động cho phép loại bỏ trùng lặp dữ liệu để tránh tiêu tốn dung lượng và đường truyền. Khi thực hiện 1 snapshots mới sẽ thực hiện đối chiếu dữ liệu với bản snapshot trước đó và ghi vào những thay đổi mới nhất kể từ bản snapshot gần nhất
- khi thực hiện xóa 1 bản snapshot thì elasticsearch chỉ thực hiện xóa duy nhất các phân đoạn sử dụng cho bản snapshot đó mà không ảnh hưởng đến các bản snapshots còn lại được lưu trữu trong cùng một repository

- Do cụm hoạt động cluster nên khi xóa bản snapshot trên 1 node thì các node còn lại cũng sẽ mất
## 2. Snapshot và shards allocation

- Để thực hiện tạo các bản snapshot thì elasticsearch thị hiện coppy các index primary shards, trong trường hợp mà các primary này đang được chuyển để tái phân bổ giữa các node thì elasticsearch sẽ thực hiện chờ cho đến việc di chuyển các index primary shards được hoàn tất. Trong trường hợp một hoặc nhiều primary có trạng thái available việc khởi tạo snapshot sẽ dẫn đến lỗi
- Trong quá trình thực hiện snapshot thì elasticsearch không thực hiện di chuyển vị trí lưu trữ của các shard mà sẽ thực hiện phân bổ lại vị trí lưu trữ sau khi snapshot hoàn thành coppy dữ liệu 

## 3. Cấu trúc file lưu trữ bên trong một repository
```sh
 STORE_ROOT
 |- index-N           - JSON serialized {@link org.elasticsearch.repositories.RepositoryData} containing a list of all snapshot ids
 |                      and the indices belonging to each snapshot, N is the generation of the file
 |- index.latest      - contains the numeric value of the latest generation of the index file (i.e. N from above)
 |- incompatible-snapshots - list of all snapshot ids that are no longer compatible with the current version of the cluster
 |- snap-20131010.dat - SMILE serialized {@link org.elasticsearch.snapshots.SnapshotInfo} for snapshot "20131010"
 |- meta-20131010.dat - SMILE serialized {@link org.elasticsearch.cluster.metadata.Metadata } for snapshot "20131010"
 |                      (includes only global metadata)
 |- snap-20131011.dat - SMILE serialized {@link org.elasticsearch.snapshots.SnapshotInfo} for snapshot "20131011"
 |- meta-20131011.dat - SMILE serialized {@link org.elasticsearch.cluster.metadata.Metadata } for snapshot "20131011"
 .....
 |- indices/ - data for all indices
    |- Ac1342-B_x/ - data for index "foo" which was assigned the unique id Ac1342-B_x (not to be confused with the actual index uuid)
    |  |             in the repository
    |  |- meta-20131010.dat - JSON Serialized {@link org.elasticsearch.cluster.metadata.IndexMetadata} for index "foo"
    |  |- 0/ - data for shard "0" of index "foo"
    |  |  |- __1                      \  (files with numeric names were created by older ES versions)
    |  |  |- __2                      |
    |  |  |- __VPO5oDMVT5y4Akv8T_AO_A |- files from different segments see snap-* for their mappings to real segment files
    |  |  |- __1gbJy18wS_2kv1qI7FgKuQ |
    |  |  |- __R8JvZAHlSMyMXyZc2SS8Zg /
    |  |  .....
    |  |  |- snap-20131010.dat - SMILE serialized {@link org.elasticsearch.index.snapshots.blobstore.BlobStoreIndexShardSnapshot} for
    |  |  |                      snapshot "20131010"
    |  |  |- snap-20131011.dat - SMILE serialized {@link org.elasticsearch.index.snapshots.blobstore.BlobStoreIndexShardSnapshot} for
    |  |  |                      snapshot "20131011"
    |  |  |- index-123         - SMILE serialized {@link org.elasticsearch.index.snapshots.blobstore.BlobStoreIndexShardSnapshots} for
    |  |  |                      the shard (files with numeric suffixes were created by older versions, newer ES versions use a uuid
    |  |  |                      suffix instead)
    |  |
    |  |- 1/ - data for shard "1" of index "foo"
    |  |  |- __1
    |  |  |- index-Zc2SS8ZgR8JvZAHlSMyMXy - SMILE serialized {@code BlobStoreIndexShardSnapshots} for the shard
    |  |  .....
    |  |
    |  |-2/
    |  ......
    |
    |- 1xB0D8_B3y/ - data for index "bar" which was assigned the unique id of 1xB0D8_B3y in the repository
    ......
```

- Đối với 1 bản snapshot sẽ có cấu trúc lưu trữ như trên, khi một bản mới snapshot được tạo ra , giá trị `index-N` sẽ tăng lên thành `index-N+1` và chưa thông tin về bản snapshot mới nhất. Việc ghi dữ liệu cũng sẽ được ghi vào bên trong các folder dựa theo cấu trúc vào các file đã tạo trước đó và sẽ có khách biệt về mặt thời gian khởi tạo
<h3 align="center"><img src="../../../../../../ELK-Stack/03-Images/dosc/74.png"></h3>
- Khi thưc hiện xóa 1 bản snapshot elastcisearch thực tìm đế xóa các file lưu trữ bên trong cấu trúc lưu trữ snapshot mà không thưc hiện xóa các folder đường dẫn có liên quan

# Phần IV. Restore
- Restore hỗ trợ khôi phục dữ liệu từ các bản snapshot đã khởi tạo trước đó. Restore cũng có thể được sử dụng để dấp ứng cho viêc di chuyển dữ liệu từ cụm này sang một cụm khác bằng cách duy chuyển snapshot từ cụm cũ sang cụm và tiến hành restore vào hệ thống 
- trên các node để có thể thực hiện restore dữ liệu cần có những quyền sau:
  - Cluster privileges: monitor, manage_slm, cluster:admin/snapshot, and cluster:admin/repository
  - Index privilege: all on the monitor index

- Việc thực hiện restore cho một cụm chỉ được thực hiện trên node master

- Khi dữ liệu được thực hiện restore thì dữ liệu sẽ được phân bổ về lớp data ở thời điểm thực hiện bản snapshot đó. Dữ liệu sẽ được phân bổ ngẫu nhiên về các node dựa theo số primary và replicas




# Tài liệu tham khảo

- https://mincong.io/en/elasticsearch-snapshot-repository-structure/
- https://www.elastic.co/guide/en/elasticsearch/reference/7.17/snapshots-register-repository.html#snapshots-read-only-repository