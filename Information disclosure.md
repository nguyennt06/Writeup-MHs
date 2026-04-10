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

# Lab
<img width="1051" height="283" alt="image" src="https://github.com/user-attachments/assets/6403110c-65ed-45e1-9b5e-c8cd0f842d71" />
\- Truy cập trang web, nhận thấy web chỉ có chức năng submit phiên bản của webserver và xem sản phẩm

\- Bấm vào sản phẩm bất kì, thêm dấu nháy đơn trên thanh URL khi truy vấn sản phẩm : `/product?productId=1'`, nhận được

```
Internal Server Error: java.lang.NumberFormatException: For input string: "1'"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:661)
	at java.base/java.lang.Integer.parseInt(Integer.java:777)
	at lab.e.u.z.n.a(Unknown Source)
	at lab.g.m.n.u.z(Unknown Source)
	at lab.g.m.g.k.g.G(Unknown Source)
	at lab.g.m.g.g.lambda$handleSubRequest$0(Unknown Source)
	at m.c.h.j.lambda$null$3(Unknown Source)
	at m.c.h.j.D(Unknown Source)
	at m.c.h.j.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.g.m.g.g.o(Unknown Source)
	at lab.server.q.v.q.l(Unknown Source)
	at lab.g.m.b.P(Unknown Source)
	at lab.g.m.b.l(Unknown Source)
	at lab.server.q.v.w.p.T(Unknown Source)
	at lab.server.q.v.w.c.lambda$handle$0(Unknown Source)
	at lab.e.h.s.t.m(Unknown Source)
	at lab.server.q.v.w.c.r(Unknown Source)
	at lab.server.q.v.f.o(Unknown Source)
	at m.c.h.j.lambda$null$3(Unknown Source)
	at m.c.h.j.D(Unknown Source)
	at m.c.h.j.lambda$uncheckedFunction$4(Unknown Source)
	at lab.server.fd.d(Unknown Source)
	at lab.server.q.v.f.C(Unknown Source)
	at lab.server.q.k.f.d(Unknown Source)
	at lab.server.q.r.H(Unknown Source)
	at lab.server.q.m.H(Unknown Source)
	at lab.server.f9.w(Unknown Source)
	at lab.server.f9.U(Unknown Source)
	at lab.p.a.lambda$consume$0(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)

Apache Struts 2 2.3.31
```
> 2 2.3.31

<img width="1067" height="301" alt="image" src="https://github.com/user-attachments/assets/81e1e96d-90ac-4502-8eea-1affbc957b42" />
\- Đọc mã nguồn của trang (Ctrl + U) của trang web và nhận được dòng comment chưa bị xoá:

```
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```
\- Truy cập endpoint trên và tìm giá trị của `SECRET_KEY`

> xkf8zd8514ofxryb95e9fzfx364z3k9q


<img width="1053" height="297" alt="image" src="https://github.com/user-attachments/assets/c6e4d5c0-91ab-41d3-9993-48c90b07aa38" />

\- Sau khi đọc page source code, không tìm tháy comment nào khả nghi chứa endpoint có thể gây rò rỉ thông tin, thực hiện fuzzing:

```
dirsearch -u https://0ab8007f044eb144807008c2006700f6.web-security-academy.net/ -t 50 -x 403
```

<img width="828" height="237" alt="image" src="https://github.com/user-attachments/assets/8d7224ac-8ff3-4b90-82ae-18dcd87c368e" />

\- Truy cập ngay vào `/backup`:

<img width="464" height="218" alt="image" src="https://github.com/user-attachments/assets/7c57398d-8a39-4770-a67f-2c0363a480b5" />

\- Bấm vào file backup trên màn hình, nhận được mã nguồn Java:
```
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "hgtrdjrhatgvrko5ophgaz7d9j3pahvs"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

> hgtrdjrhatgvrko5ophgaz7d9j3pahvs

<img width="1071" height="493" alt="image" src="https://github.com/user-attachments/assets/154bcdbe-b331-4076-ac06-864148042c26" />

\- Truy cập vào trang web, view page source không cho ta bất cứ thông tin gì giá trị, từ đó, thực hiện fuzzing:
```
ffuf -u https://0a6100ac0338138e842a1ebd00de006e.web-security-academy.net/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -t 50 -ic
```

<img width="1084" height="489" alt="image" src="https://github.com/user-attachments/assets/d320dac6-4bd9-41d2-b3ec-7d2be70fcb11" />

\- Truy cập endpoint `/admin`, nhận được một dòng:

```
Admin interface only available to local users
```

\- Kèm với description của bài, ta nghĩ ngay đến header `X-Forward-For` hoặc `X-Real-IP`:

<img width="1859" height="551" alt="image" src="https://github.com/user-attachments/assets/fa5f20a0-8862-407e-9304-33fa1de95fb7" />

\- Đời không như mơ, ta không thể dễ dành vào admin panel chỉ với những header này

\- Đọc lại mô tả, có vẻ như ta cần biết thêm về một header nào đó khác hoàn toàn

\- Sử dụng HTTP Method `TRACE` để truy cập endpoint `/admin`, ta nhận được

<img width="1836" height="491" alt="image" src="https://github.com/user-attachments/assets/69278bbc-e4fc-414a-8960-c1f76d832f2e" />

- `X-Custom-IP-Authorization` được Frontend (Proxy) tự động gắn thêm vào ngay khi request vừa chạm tới hệ thống để "báo cáo" với Backend IP thật của attacker là gì.

\- Trong tay đã có chìa khoá của bài toán, ta chỉ việc set giá trị của header:

<img width="1694" height="286" alt="image" src="https://github.com/user-attachments/assets/b3676191-265c-4e48-bc9e-fa2ca18561d7" />


$\implies$ Sửa endpoint thành `/admin/delete?username=carlos` và hoàn thành lab ❤

<img width="1072" height="348" alt="image" src="https://github.com/user-attachments/assets/2b161a77-e6e4-43ef-a453-7a8aba7ff119" />

### Enumeration/fuzzing

\- Nhờ việc cài extension [DotGit](https://chromewebstore.google.com/detail/dotgit/pampamgoihgcedonnphgehgondkhikel), sau khi truy cập trang web, nó đã báo với mình việc lộ endpoint /.git trên website:

<img width="713" height="401" alt="image" src="https://github.com/user-attachments/assets/d574e6cb-0630-4e19-ac03-53b5b506b749" />

\- Thêm vào đó, ta có thể fuzzing nhằm đảo bảo không bỏ sót một endpoint/thư mục vào của web:

```
dirsearch -u https://0ac600e60370b1fa8131b12700ee00d8.web-security-academy.net/ -t 50 -x 503,401,403
```
<img width="882" height="813" alt="image" src="https://github.com/user-attachments/assets/8ae5c12e-3916-45d7-9a06-329baa416062" />


$\implies$ Khai thác rò rỉ thông tin uqa thư mục kho chứa Git hoặc lịch sử kiểm soát phiên bản

\- **Lý do tại sao có lỗ hổng:** Dev thường dùng lệnh `git clone` hoặc `git pull` trực tiếp trên máy chủ để cập nhật mã nguồn cho nhanh. Khi làm vậy, Git sẽ tự động tạo ra thư mục ẩn .git để quản lý phiên bản

\- Hậu quả:
- Trace được những gì họ đã thêm, đã sửa, và quan trọng nhất là những gì họ đã cố gắng xóa đi (như mật khẩu, API key, endpoint ẩn)
- Làm lộ cấu hình nội bộ: Thông tin về server database, các nhánh (branches) đang phát triển dang dở

### Khai thác
<img width="734" height="670" alt="image" src="https://github.com/user-attachments/assets/4e100f72-af78-49ed-a1c9-006ca118e286" />

\- Truy cập ngay thư mục `/.git/logs/refs/heads` và vào đọc file `master`, ta nhận được:
```
0000000000000000000000000000000000000000 1fa8b27a5914d22088834912c6491b62592dd791 Carlos Montoya <carlos@carlos-montoya.net> 1775787050 +0000	commit (initial): Add skeleton admin panel
1fa8b27a5914d22088834912c6491b62592dd791 61b59646590e3b2650470d3b506c80756e9fcf2d Carlos Montoya <carlos@carlos-montoya.net> 1775787050 +0000	commit: Remove admin password from config
```
- Initial commit: Thêm mật khẩu cho admin
- Một bản cập nhật được đẩy lên nhằm xoá đi mật khẩu
- **Lý do tại sao có lỗ hổng:**
  	- Thông thường, khi dev xóa một mật khẩu khỏi mã nguồn và commit, họ nghĩ rằng nó đã biến mất. Nhưng thực tế, Git lưu lại toàn bộ trạng thái của file tại mỗi mốc hash

	- Việc server để lộ thư mục `/.git` cho phép bạn đọc được file log này. Nhìn vào đây, biết chính xác rằng ở phiên bản nào mật khẩu vẫn còn nằm trong file config

 \- Sử dụng `git-dumper` để dump toàn bộ version control history và khôi phục phiên bản trước khi mật khẩu admin bị xoá:

 ```
git-dumper https://0ac600e60370b1fa8131b12700ee00d8.web-security-academy.net/.git/ portswig
```

\- Ta đã tải được toàn bộ /.git về và được bonus thêm 2 file nữa liên quan đến admin, với mật khẩu admin được lưu trong `admin.conf`:
<img width="1096" height="237" alt="image" src="https://github.com/user-attachments/assets/c4969e89-d3ef-4ead-bbb8-d5f3d5451464" />

\- Quay ngược về thời điểm trước khi việc xoá mật khẩu được commit:

```
cd portswig
git show 1fa8b27a5914d22088834912c6491b62592dd791:admin.conf
```

hoặc

```
cd portswig
git checkout 1fa8b27a5914d22088834912c6491b62592dd791
cat admin.conf
```

<img width="1019" height="145" alt="image" src="https://github.com/user-attachments/assets/f61173b5-b1c5-450c-90d6-b0fa58985bb9" />

\- Thực hiện đăng nhập với `administrator:pnsq53p7z3j4ph5fc1ps` và xoá carlos

<img width="675" height="410" alt="image" src="https://github.com/user-attachments/assets/76e198f9-543d-46fa-8667-2c397b0b46f9" />











