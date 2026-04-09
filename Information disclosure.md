<img width="837" height="254" alt="image" src="https://github.com/user-attachments/assets/56b7741c-46cb-420f-9927-fcca0c2382eb" />

# Định nghĩa
\- Server-side vuln 

\- Lộ, lọt thông tin là lỗ hổng khi website vô tình tiết lộ thông tin nhạy cảm cho người dùng, dựa vào từng ngữ cảnh, có thể làm lộ toàn bộ thông tin cần thiết cho attacker trong việc pentest web
- Thông tin về người dùng khác, như `username`, `email` hoặc thông tin về tài chính: số thẻ, số tài khoản 
- Dữ liệu về doanh nghiệp, thương mai, buôn bán trao đổi....  
- Thông tin kĩ thuật về website và hạ tầng của nó (cấu trúc thư mục, framework của một bên thứ ba,....)

\- Việc làm lộ thông tin về người dùng, doanh nghiệp và đặc biệt là technical details là sự khởi đầu của mở rộng bề mặt tấn công  (attack surface), nơi có thể chứa các lỗ hổng nghiệm trọng, các CVE đã được public

\- Khi này các hành vi, phản hồi của web sẽ được đưa vào scope nhằm khai thác triệt để việc lộ lọt thông tin 

\- Ví dụ về information leakage:
- Tiết lộ tên của thư mục ẩn, cấu trúc của nó và nội dung thông qua `robots.txt`, `.DS_Store`, `UPDATE.txt` hoặc liệt kê thư mục
- Cung cấp quyền truy cập vào source code thông qua sao lưu tạm thời
- Đề cập rõ ràng về bảng, cột trong database thông qua thông báo lỗi
- Nhúng trực tiếp API key vào trong mã nguồn ứng dụng (Hard-coding API keys), IP addresses, database credentials
    - Thay vì sử dụng các biến môi trường hoặc tệp cấu hình bên ngoài để quản lý bảo mật, mã khóa được viết cứng vào các file như `.py`, `.js`, `.java`

- Gợi ý về sự tồn tại (hoặc biến mất) của các tài nguyên thông qua sự khác nhau giữa các hành vi của web

# Nguồn cơn và ảnh hưởng của lộ lọt thông tin
### Tại sao có lỗ hổng này
\- Không xoá những nội dung nội bộ khi public sản phẩm, ví dụ: comment của dev có thể được đọc bởi người dùng khi sử dụng trang web

\- Cấu hình không bảo mật cho web và các công nghệ liên quan, ví dụ: không tắt chức năng debug và diagnose (có thể thêm param `&debug` nhằm xem kết quả câu lệnh trả ra) hoặc thông báo lỗi quá rõ ràng về ứng dụng, tiện ích và website đang sử dụng

\- Lỗ hổng khi thiết kế trang web và hành vi của web, ví dụ: dựa vào các phản hồi khác nhau khi đăng nhập, attacker có thể dựa trên những hành vi nhỏ, đoán xem `username` có tồn tại hay không (user enumeration) 
- Thông báo lỗi `username that is already taken` $\implies$ brute-force

### Tác động (Tác hại)
\- Trực tiếp: Lộ thông tin cốt lõi, ví dụ: thông tin giao dịch, thẻ ngân hàng, thẻ tín dụng của khách hàng khiến, gây hậu quả nặng nề đối với uy tín doanh nghiệp

\- Gián tiếp: Rò rỉ thông tin kĩ thuật, cấu trúc thư mục, mã nguồn khiến đây là bàn đạp tấn công cho attacker, từ đó vạch ra các chuỗi khai thác (attacker chain) phức tạp

### Đánh giá mức độ nghiêm trọng của lỗ hổng
\- Mức độ nghiêm trọng tuỳ thuộc vào attacker chứng minh được khả năng lợi dụng thông tin rò rỉ (PoC) để thực hiện các hành vị nguy hiểm

\- Đặc biệt, nếu thông tin bị lộ là password, khoá API key của admin hoặc token đăng nhập, đây là những thông tin cực nhạy cảm và tác động rất lớn đến sự bảo mật của web

