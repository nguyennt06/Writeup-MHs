# Nhận diện bug SSRF
![image](https://hackmd.io/_uploads/SyLXD9u8Zx.png)
### 1. Chức năng nghiệp vụ (Business Logic)
-- Bất kỳ chức năng nào yêu cầu máy chủ "đi lấy" dữ liệu từ một nguồn khác: hệ thống nội bộ lẫn hệ thống bên ngoài, trang web khác
-- Ví dụ: 
- Import data (file, ảnh) từ URL
- Preview link (xem trước link có gì -> tải dữ liệu về trang web)
- HTML to PDF, web redirection

### 2. Nhận diện thông qua tham số

-- Các tên tham số khả nghi như: `url`, `path`, `link`, `target`, `dest`, `source`, `redirect`, `u`, `uri`, `callback`, `api`

-- Giá trị của chúng:
- `http://www.example.com`
- `192.168.0.1`
- `/api/check/v1`

### 3. Kiểm thử
#### a) Thực hiện kết nối ra bên ngoài
-- Thay url, link, path của request gốc thành `webhook.site` hoặc `Burp Collaborator` của mình
-- Nếu phía ta nhận được request từ IP của máy chủ ứng dụng -> SSRF *(pingback -> không quá nguy hiểm)*
-- Kết nối đến IP không tồn tại (404 Not Found) hoặc IP tồn tại nhưng đóng port (500 Server Error) --> xác nhận có kết nối ra ngoài

#### b) Thực hiện kết nối nội bộ
-- Đổi URL thành `http://localhost` và `http://127.0.0.1` 
-- Sử dụng [IP Obfuscation](https://h.43z.one/ipconverter/), để bypass blacklist filter từ phía server
-- Truy vập vào các endpoint như `/admin.php`, `/admin` để thực hiện leo quyền




# Lab 04
![image](https://hackmd.io/_uploads/rkdSAFu8-l.png)
Đề bài cho biết dev đã sử dụng 2 lớp phòng thủ yếu SSRF. Đọc documentation của lỗ hổng này trên Port, ta biết được:
![image](https://hackmd.io/_uploads/Hkx6JcuIWg.png)
- Server block ip loopback
- Server block endpoint lạ trên URL
 
### Khai thác
1. Bypass IP block:
- Sử dụng [Công cụ đổi IP Obfuscation](https://h.43z.one/ipconverter/) 
- Thử nghiệm với `stockApi=http://127.0.1/`, ta nhận về được trang web mà ta truy cập vào lúc gửi request đi

$\implies$ Bypass thành công

2.  Bypass Blacklist Path 
- Khi gọi đến `http://127.0.1/admin`, ta thử URL-encode một chữ bất kì hoặc URL-encode cả 'admin', thành:
```
 stockApi=http://127.0.1/%61%64%6d%69%6e
``` 

![image](https://hackmd.io/_uploads/HkJw79O8bg.png)

--> Như vậy backend đã có thể sử dụng kĩ thuật **Double URL Encoding** 
- Ta thử URL-encode `%61%64%6d%69%6e` (admin) thêm một lần nữa:
```
stockApi=http://127.0.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65
```

$\implies$ Thành công bypass

> Nhiều ứng dụng thực hiện URL decode đầu vào một lần rồi mới kiểm tra blacklist. Sau khi kiểm tra xong, nó lại chuyển chuỗi đó cho backend xử lý, và backend lại decode một lần nữa

3. Thực hiện yêu cầu xoá `carlos`

- Khi thành công truy cập với tư cách `admin`, ta nhận được

```htmlmixed!
<div>
    <span>wiener - </span>
        <a href="/admin/delete?username=wiener">Delete</a>
</div>
<div>
    <span>carlos - </span>
        <a href="/admin/delete?username=carlos">Delete</a>
</div>
```

- Gửi request đến URL vừa nãy, thêm endpoint `/delete`

```
stockApi=http://127.0.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65/delete?username=carlos
```
![image](https://hackmd.io/_uploads/Sk5ev9uUZe.png)

# Lab 05
![image](https://hackmd.io/_uploads/H1-X5h_8We.png)
- Trong bài này, web server gửi req POST, gọi vào một chức năng api nội bộ và lấy ra dữ liệu, thay vì gọi đến 1 URL rồi làm điều tương tự

-- Thử cho server gọi đến trang `admin` của chính nó:
```
stockApi=http://127.0.0.1/admin
```
-- Ta nhận được ![image](https://hackmd.io/_uploads/SJqLZMF8bx.png)

- Điều này chứng tỏ server đã không lấy dữ liệu từ một url/link bất kì nào nữa

-- Theo hướng giải mà đề bài cho, ta quan sát thấy thêm 1 chức là là `Next post`, khi server tạo một request chứa path của bài viết tiếp theo và hiển thị ra
![image](https://hackmd.io/_uploads/HJDYwWF8-x.png)
Xem req qua Burp, ta nhận được
```
GET /product/nextProduct?currentProductId=2&path=/product?productId=3 HTTP/2
```
-- Khi thực hiện request, server sẽ chuyển hướng sang endpoint tiếp theo, là giá trị của biến `path`, hiển thị toàn bộ source code của trang web

-- Quay trở về chức năng check stock ban đầu, ta thử sử dụng biến `stockApi` gọi trực tiếp đến endpoint `/product/nextProduct` (vai trò là bộ điều hướng), *mượn tay* server redirect đến trang admin mà ta cần:
```
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
```

$\implies$ Thành công có được mã nguồn của `/admin`, ta chỉ việc xoá người dùng theo đường dẫn được gợi ý

```html!
<div>
     <span>wiener - </span>
           <a href="/http://192.168.0.12:8080/admin/delete?username=wiener">Delete</a>
</div>
<div>
     <span>carlos - </span>
           <a href="/http://192.168.0.12:8080/admin/delete?username=carlos">Delete</a>
</div>
```
> Trỏ đến đúng URL với port mà đề bài cho

![image](https://hackmd.io/_uploads/S1XZFfFLbx.png)

# Lab 06 - Whitelist-based
![image](https://hackmd.io/_uploads/SyFFJy9U-g.png)
## Khai thác
-- Khi thực hiện chức năng `Check stock` như những lab trước, ta thử gọi về hệ thống nội bộ để thực hiện leo quyền
```
stockApi=http://localhost/admin
```
-- Nhận được :+1: ![image](https://hackmd.io/_uploads/rkwU_k5LWx.png)
- Như vậy server đã whitelist --> chỉ cho check hàng thông qua 1 domain cố định


$\implies$ Hình thành hướng khai thác thông qua việc: vẫn gọi đến domain được whitelist nhưng khi thực hiện request, bypass bộ lọc, trình duyệt và server sẽ xử lí khác đi
- Lý thuyết trên Port đã gợi ý về việc sử dụng `@` , `#`, `.`

-- Ta có thể tự craft một payload dựa trên misconfig giữa server và trình duyệt trong việc xử lí URL
- Trong URL, mở đầu vẫn sẽ là `http://localhost`, nhằm đánh lạc hướng, ta để ở sau là dấu `@`
- Khi xử lí, trình duyệt sẽ tưởng nhầm trước dấu `@` là credential vô hại và cho qua, coi phần đằng sau mới là domain chính 
$\implies$ Bypass thành công Whitelist
- Thế nhưng còn việc khiến trình duyệt trả ra trang `/admin`, ta cần truy cập vào nội bộ server
- Khi này, ta thêm `#` vào trước `@`, nhằm comment toàn bộ phần đằng sau để trình duyệt chỉ trả về trang `/admin` mà ta cần
`stockApi=http://localhost#@stock.weliketoshop.net`

- Nhận được Response

![image](https://hackmd.io/_uploads/Sk3_2JqL-e.png)

- Chắc chắn server đã double-decoding rồi mới xử lí dữ liệu, vì vậy ta phải URL-encode 2 lần kí tự bị blacklist, là `#`

```
stockApi=http://localhost%25%32%33@stock.weliketoshop.net/
```
$\implies$ Thành công vào được hệ thống nội bộ của web
- Đến đây, truy cập vào endpoint của admin:

```
stockApi=http://localhost%25%32%33@stock.weliketoshop.net/admin
```
- Thực hiện xoá carlos: `/admin/delete?username="carlos"`

![image](https://hackmd.io/_uploads/ByP6gl98Wl.png)


## Workflow của trình duyệt và server khi xử lí một url
### Double URL Encoding (%2523)
-- Mục đích: Để đưa ký tự `#` đi qua được lớp cửa bảo vệ đầu tiên (Web Server/Load Balancer) mà không bị trình duyệt hoặc server cắt bỏ sớm

-- Quy trình: Kẻ tấn công gửi %2523 -> Server decode lần 1 thành %23 -> Bộ lọc kiểm tra %23 (thấy an toàn) -> Thư viện HTTP decode lần 2 thành `#` 
### Dấu '@'
-- Dùng để ngăn cách giữa thông tin đăng nhập và địa chỉ máy chủ
-- Lợi dụng chức năng này: Dùng để ***ngụy trang*** cấu trúc đối với bộ lọc, server  lầm tưởng rằng `localhost%23` chỉ là thông tin credential vô hại, và cho rằng `stock.weliketoshop.net` phía sau mới là domain đích thực --> PASS
![image](https://hackmd.io/_uploads/S14eNk5IZx.png)
-- Khi server bắt buộc check host là một giá trị cố định, ta cần '@' để cho bypass filter, đưa localhost vào trong phần xử lí tiếp theo

### Dấu '#' (comment)
-- Thường chỉ dùng ở phía client và không được gửi trong HTTP request
-- Báo hiệu bắt đầu một fragment (phân đoạn)
-- Đối với server thực hiện request, mọi thứ sau dấu '#' sẽ bị bỏ qua hoặc không ảnh hưởng đến việc định tuyến (routing). Nó giúp loại bỏ phần domain `stock.weliketoshop.net` ra khỏi logic kết nối thực tế, buộc server kết nối về `localhost` đứng trước đó
