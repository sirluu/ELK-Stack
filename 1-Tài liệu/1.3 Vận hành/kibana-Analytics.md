<h1 align="center">Hướng dẫn sử dụng : Kibana Analytics</h1>

## Mục lục
I. [Discover](#Discover)

II. [Dashboard](#Dashboard)

III. [Canvas](#Canvas)

IV. [Maps](#thanhphan)

<h3 align="center">-----------------------------------------</h3>

## Phần I. <a name="Discover"></a>Discover

- Discover là tính năng hỗ trợ người dùng hiển thị, tìm kiếm và phân tích dữ liệu 
- Discover hiển thị dữ liệu bằng cách sử dụng các `index Parttern` để gọi đến các `data index` được lưu trữ bên trong Elasticsearch.
### 1. Show logs in Discover
**` Bước 1`**: Kiểm tra danh sách data index đang tồn tại 
- Tại giao diện màn hình chính tích chọn `menu` có biểu tượng 3 dấu gạch ngang -> `Management` -> `Stack Management`

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/15.png"></h3>

- Tại mục `Data` chọn mục `index Management` để hiện tất cả các trường data index đang tồn tại

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/16.png"></h3>

Sau khi kiểm tra và xác định được các trường index cần show ra màn hình sẽ thực hiện khởi tại các index Parttern để truy vấn.

- Tại mục `kibana` -> `Index Partterns` -> `Create index Partterns`:
  - Ở đây có thể tạo 1 index để thu thập dữ liệu từ nhiều file index khác nhau sử dụng cú pháp: `acb*`. Đối với cũ pháp này có thể thu thập dữ liệu các file index có các ký tự đầu là `acb`

- Tạo index pattern:
  - `name`: access_log*
  - `Timestamp field`: @timestamp  // sử dụng để có thể lọc dữ liệu theo thời gian 

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/17.png"></h3>
- Kết quả:

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/18.png"></h3>

- Sau đó quay lại màn hình discover để kiểm tra

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/19.png"></h3>

### 1. Option có trong Discover
**`Tìm kiếm dữ liệu dựa trên Index Parttern`**
- Có thể thay đổi các trường Index Parttern để thay đổi có kiểu dữ liệu logs cần hiển thị ra màn hình.

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/20.png"></h3>

- Tùy vào đối tượng, loại và mục đích tìm kiếm dữ liệu mà và có các khung hiển thị khoảng thời gian hiển thị dữ liệu khác nhau.
- Có thể thực hiện thay đổi linh hoạt khoảng thời gina tìm kiếm dữ liệu khác nhau từ các đơn vị: giây, phút, tháng, năm.


<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/21.png"></h3>

- Ngoài việc hiển thị data ở dạng `text` thì để đưa ra cái nhìn khách quan hệ thống sẽ hiển thị 1 biểu đồ cột  với một số thông tin cơ bản về số lượng dữ liệu cũng như mốc thời gian tương ứng

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/22.png"></h3>

**` Tìm kiếm và lọc dữ liệu`**

Đáp ứng nhu cầu tìm kiếm và phân loại dữ liệu có thể sử dụng tính năng hỗ trợ tìm kiếm và phân loại: `Search`, `filter`

- `Search`: Tìm kiếm dữ liệu theo `fields` cũng với các giá trị cụ thể hoặc là keyword

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/23.png"></h3>

- `Availabe fields`: thực hiện lọc dữ liệu theo fields hiển thị ra màn hình

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/24.png"></h3>


- `add filter`: tạo bộ lọc tìm kiếm dữ liệu theo trường dữ liệu

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/25.png"></h3>

- Để có thể xem top các giá trị phổ biến của fields chọn nhấp vào để hiển thị

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/26.png"></h3>



## Phần II. <a name="Dashboard"></a>Dashboard

- Dashboard cung cấp một giao diện trực quan, theo dõi thông qua các bảng biểu

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/27.png"></h3>

Để thực hiện khởi tạo một dashboard cần thực hiện các điều sau:

Tại `Dashboards` -> `Create Dashboards` -> `Creats visualization` sau đó tiến hành khởi tạo theo ý muốn:

<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/28.png"></h3>
<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/29.png"></h3>
<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/30.png"></h3>

- Những điều cần thiết để khởi tạo dashboard:
  - Visualization type: Bar, Table, Area, line, Pie,...
  - Data: Index Parttern đã khởi tạo trước đó
  - Xác định các Fields

Ví dụ:
<h3 align="center"><img src="../../../ELK-Stack/03-Images/dosc/31.png"></h3>


