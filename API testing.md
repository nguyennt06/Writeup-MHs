# API testing
## Nội dung
### API là gì
- APIs (Application Programming Interfaces) cho phép giao tiếp và chia sẻ dữ liệu giữa hệ thống phần mềm và ứng dụng
- Lỗ hổng API có thể gây ảnh hưởng đến tính bí mật *(confidentiality)*, tính toàn vẹn *(integrity)* và tính khả dụng *(availability)*

### API recon
-- Sử dụng thư viện`/usr/share/seclists/Discovery/Web-Content/api/api-enpoints.txt` hoặc tương tự

-- Sử dụng tất cả các chức năng của web để tìm ra những request sử dụng API 
-- Các tài liệu API (swagger/openapi/docs) có thể truy cập trực tiếp trên thanh URL

### Attack Surface
- Các endpoint được phát hiện (URL)
- Phương thức HTTP được hỗ trợ (GET/POST/PATCH/DELETE…)
- Tài liệu API 
- Content-Type được chấp nhận (JSON/XML/…)
- Các param ẩn không được liệt kê trong request và response
- Các trường không được validate


## Khai thác
-- Thử logic tampering
- Thay đổi giá, số lượng
- Đổi giá trị thành số âm, chuỗi, kí tự đặc biệt
- Field không được validate

-- Test các HTTP method

- POST → DELETE
- GET → PATCH

-- Test các kiểu Content-Type

- JSON
- XML
- Form

-- Test parameter pollution 

- `id=1&id=2`
- duplicate fields

-- Test mass assignment (cấp quyền hàng loạt)

- Thêm field admin, role, a, administrator…
- Cung cấp giá trị cho `isAdmin` 

$\implies$ Nếu giá trị `isAmin` trong request được gắn cho đối tượng người dùng mà không hề có validation, người dùng thường sẽ được cấp quyền trái phép. 

# Lab
## Exploiting an API endpoint using documentation
![image](https://hackmd.io/_uploads/SkGTy8hOZx.png)
![image](https://hackmd.io/_uploads/SkOxQEeFWl.png)


-- Thực hiện đăng nhập với credentials được cho trước 

-- Ta tìm được tài liệu về API của trang web: `/api`
![image](https://hackmd.io/_uploads/S13Rjflt-x.png)

-- Khi truy cập vào endpoint này, ta nhận được thông tin về các chức năng của RESTful API
![image](https://hackmd.io/_uploads/HJ_NsfgYbe.png)

-- Tập trung vào việc xoá `carlos`, ta cần gửi một request với method `DELETE` vào enpoint `/api/user/carlos`, và hoàn thành lab
![image](https://hackmd.io/_uploads/ryGt3zlKWx.png)

## Finding and exploiting an unused API endpoint
![image](https://hackmd.io/_uploads/rk8TkQlt-l.png)
![image](https://hackmd.io/_uploads/H1ptaQxFWe.png)

-- Thực hiện đăng nhập với credential cho trước
-- Với store credit là $0.00, ta không thể mua bất kì món đồ gì. Từ đó nghĩ đến việc điều chỉnh giá trị của jacket nhằm đạt được yêu cầu bài lab

-- Thêm jacket vào giỏ hàng, nhận được 1 loạt request, trong đó:  
![image](https://hackmd.io/_uploads/Skx25h7gKbe.png)
- `POST /cart` không chứa param giá tiền trong body
- `GET /product?productId=1`  chỉ tải thông tin về sản phẩm

-- Tập trung vào `GET /api/products/1/price`, khi nó chứa endpoint `/price` khá khả nghi
-- Theo phần `Required knowlegde`, ta cần biết cách thay đổi HTTP Method để phát hiện thêm chức năng phụ:
- Method `OPTIONS` thường được dùng để kiểm tra các phương thức giao tiếp được hỗ trợ
![image](https://hackmd.io/_uploads/Byq_a7xtbe.png)
- Vậy endpoint này còn hỗ trợ method `PATCH`, được dùng để thay đổi giá tiền chăng?

![image](https://hackmd.io/_uploads/S1rsR7gY-l.png)

-- Dựa trên request gốc khi trả ra json, ta đổi method thành `PATCH` đồng thời sửa `Content-Type` từ "application/x-www-form-urlencoded" $\implies$ "application/json" kèm body là:
![image](https://hackmd.io/_uploads/HkapJNxtZx.png)
- Vậy backend yêu cầu biến `price` là một số nguyên không âm thay vì một chuỗi 

-- Sửa lại body:
```json!
{"price": 0}
```
![image](https://hackmd.io/_uploads/SyV9l4gY-l.png)

-- Thành công điều chỉnh giá, ra checkout và mua thành công jacket "l33t"
![image](https://hackmd.io/_uploads/SyP-ZNlFWg.png)






## Exploiting a mass assignment vulnerability
![image](https://hackmd.io/_uploads/r19m9XxFZx.png)
![image](https://hackmd.io/_uploads/r1yfXElYbg.png)
-- Đăng nhập với credential cho trước
-- Thực hiện mua jacket với số dư $0.00, ta nhận được các request
![image](https://hackmd.io/_uploads/By-xU4gF-g.png)

-- Check từng response để xem, liệu server có tiết lộ một param ẩn nào không?

- Với request `POST /api/checkout` để thanh toán thực sự sản phẩm, ta gửi đi request với body:  
```json!
{
    "chosen_products":[
        {
            "product_id":"1",
            "quantity":1
        }
    ]
}
```

- Với request `GET /api/checkout` để đến trang thanh toán, ta nhận được: 
![image](https://hackmd.io/_uploads/BJAV_ExF-e.png)

-- Ở đây có trường `percentage` trong đối tượng `chosen discount` khá đáng nghi khi đây có thể là số % được giảm giá của sản phẩm

-- Gửi trực tiếp json này trong body của request `POST /api/checkout`, sửa 0 thành 100:
```json!
{
    "chosen_discount":{
        "percentage":100
    },
    "chosen_products":[
        {
            "product_id":"1",
            "name":"Lightweight \"l33t\" Leather Jacket",
            "quantity":1,
            "item_price":133700
        }
    ]
}
```
![image](https://hackmd.io/_uploads/SJGr8HgtZx.png)


$\implies$ Thành công mua jacket "l33t"


