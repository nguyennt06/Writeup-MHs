<img width="838" height="338" alt="image" src="https://github.com/user-attachments/assets/49a5df53-567f-48cd-9954-533d67861225" />

# [Race condition là gì?](https://portswigger.net/web-security/race-conditions)
## 1. Bản chất và Nguồn cơn của Lỗ hổng (Root Cause)
**Đây là nguồn cơn cốt lõi trong kiến trúc phần mềm dẫn đến việc Server bị khai thác**

***\- Lỗ hổng Race Condition trên web hiện đại xuất phát 100% từ phía Server***

### a. Nhóm Nguyên nhân Gốc rễ (Root Causes)

\- Concurrency (Xử lý đồng thời / Đa luồng): Khả năng máy chủ mở nhiều luồng (threads) cùng lúc để xử lý nhiều yêu cầu (requests) nhằm tăng hiệu suất. Nếu không có đa luồng, sẽ không có Race Condition

\- Shared State (Trạng thái dùng chung): Vùng dữ liệu (như số dư ví, giỏ hàng, biến lưu email tạm thời) bị nhiều luồng cùng trỏ vào để đọc hoặc ghi trong cùng một thời điểm

\- Thiếu cơ chế Locking (Khóa dữ liệu): *Nguyên nhân chí mạng nhất*. Lập trình viên không thiết lập lệnh "khóa" tạm thời cái Shared State lại khi một luồng đang xử lý. Việc cửa mở hớ hênh cho phép các luồng khác tự do nhảy vào can thiệp giữa chừng

### b. Nhóm Sai hỏng Logic (Logical Flaws)
**Khi 3 nguyên nhân gốc rễ ở trên hội tụ, chúng sẽ sinh ra các lỗ hổng logic cụ thể:**

\- TOCTOU (Time-of-Check to Time-of-Use): Lỗi khoảng hở thời gian. Server chia một nghiệp vụ làm 2 bước rời rạc: 
- Bước 1 là "Kiểm tra" (Check - ví dụ: xem đủ tiền không)
- Bước 2 là "Thực thi" (Use - ví dụ: trừ tiền). Kẻ tấn công lách vào khoảng hở vài mili-giây giữa 2 bước này để đánh tráo dữ liệu (nhét thêm hàng vào giỏ)

\- Partial Construction (Khởi tạo không hoàn chỉnh): Quá trình khởi tạo một đối tượng (như User) bị chia làm nhiều nhịp. Trong một tích tắc, tài khoản tồn tại trong Database nhưng bị "khuyết" dữ liệu (ví dụ: Token xác nhận bị rỗng). Kẻ tấn công gửi lệnh xác thực ngay đúng tích tắc đó để ép server duyệt thành công

\- Collision / State Override (Va chạm / Ghi đè trạng thái): Server dùng chung một biến tạm cho nhiều luồng (VD: pending_email). Luồng 2 chạy nhanh hơn đè dữ liệu lên biến tạm của Luồng 1, khiến Luồng 1 thực thi sai đích đến (Gửi nhầm token sang mail của attacker)

### c. Nhóm Yếu tố Mạng & Thời gian (Network & Timing)
**Đây là các yếu tố vật lý quyết định việc khai thác thành công hay thất bại.**

\- Time Latency (Độ trễ thời gian): Tổng thời gian một gói tin đi từ máy tính của bạn đến máy chủ. (Khái niệm cơ bản, không mang tính quyết định trong khai thác)

\- Network Jitter (Biến thiên độ trễ mạng): Sự sai lệch thời gian đến đích của các gói tin gửi đi cùng lúc do tình trạng mạng Internet không ổn định. Đây là rào cản lớn nhất, khiến các request đến Server bị so le nhau, không tạo ra được sự va chạm luồng

\- Single-packet Attack (Tấn công gói tin đơn): Tuyệt chiêu vượt qua Network Jitter. Gom hàng chục request lại, nhét chung vào đúng một gói TCP (dựa trên HTTP/2) và bắn đi. Khi gói tin tới nơi, mọi request bung ra cùng một lúc, ép Server phải tiếp nhận song song tuyệt đối

\- Time Resolution / Internal Jitter (Độ phân giải thời gian CPU): Kẻ thù cuối cùng ở mức vi mô. Dù gói tin đến cùng lúc, CPU của Server khi chia việc cho các luồng vẫn có thể bị lệch nhau vài micro-giây. Để chiến thắng yếu tố hên xui này, ta phải tăng "hỏa lực" (gửi 20-30 cặp request cùng lúc) để ép ít nhất 2 luồng phải trùng khít thời gian thực thi
## 2. Bước ngoặt công nghệ: Single-packet Attack

\- Trong quá khứ, Race Condition thường bị coi là "hên xui" vì sự cản trở của Độ trễ mạng (Network Jitter) – các gói tin gửi đi cùng lúc nhưng đến Server lại lệch nhau vài mili-giây.

\- [Bài nghiên cứu của PortSwigger](https://portswigger.net/research/smashing-the-state-machine) đã chỉ ra cách triệt tiêu hoàn toàn sự hên xui này bằng Single-packet Attack (tận dụng kiến trúc HTTP/2):
- Công cụ sẽ gom 20-30 yêu cầu (requests) lại, kìm byte cuối cùng của chúng, đóng gói vào một gói tin TCP duy nhất và bắn đi.

  <img width="940" height="252" alt="image" src="https://github.com/user-attachments/assets/7eb4975c-21d7-4bef-b98f-75f8448f8571" />

- Khi gói tin này chạm card mạng của Server, toàn bộ yêu cầu bung ra tại đúng một micro-giây, ép hệ thống bộc lộ sơ hở một cách tuyệt đối.

## 3. Dấu hiệu nhận biết mục tiêu (Reconnaissance)
\- Có áp đặt Giới hạn định lượng (Limits/Quotas): Bất cứ tính năng nào liên quan đến số đếm. Ví dụ: Số dư ví, số lần áp dụng voucher, giới hạn số lần nhập sai mật khẩu (rate-limit), số lượng hàng tồn kho

\- Có luồng xử lý Đa điểm (Multi-endpoint): Khi nhiều API khác nhau cùng thao tác lên một luồng dữ liệu. Ví dụ điển hình là quy trình thương mại điện tử: /cart (thêm đồ) và /checkout (chốt đơn) cùng can thiệp vào một Session giỏ hàng

<img width="940" height="184" alt="image" src="https://github.com/user-attachments/assets/e9488bb9-801e-4606-b6b8-1365123cd068" />


\- Có trạng thái nhạy cảm thời gian (Time-sensitive/Partial Construction): Các quy trình tạo ra trạng thái tạm thời. Ví dụ: Đăng ký tài khoản (Tạo User vào DB trước, sinh Token xác nhận sau), hoặc sinh Token quên mật khẩu dựa trên dấu thời gian (Timestamp) của máy chủ

## 4. Kịch bản khai thác thực tế & CTF:
### a. Trong môi trường CTF (Tập trung vào Logic Bypass)
\- Vượt rào xác thực (Partial Construction): Lợi dụng khoảng hở giữa việc Server khởi tạo bản ghi người dùng nhưng chưa kịp sinh Token. Kẻ tấn công gửi ngay một request POST /confirm với tham số token rỗng. Server so sánh token rỗng với trường token chưa tồn tại trong DB, dẫn đến việc tài khoản được xác thực trái phép

\- Va chạm Token (Token Collision): Khi Server dùng thời gian hệ thống (Timestamp) để băm (Hash) ra Token đổi mật khẩu. Kẻ tấn công gửi đồng thời 2 request đổi mật khẩu cho mình và cho nạn nhân. Hai luồng chạy cùng một micro-giây sẽ đẻ ra 2 token giống hệt nhau, cho phép lấy token của mình để đặt lại mật khẩu của nạn nhân

\- Thao túng trạng thái giỏ hàng (TOCTOU &rarr; time-of-check time-of-use): Đợi luồng Thanh toán vừa kiểm tra xong số dư hợp lệ cho một món đồ rẻ tiền, lập tức dùng luồng Giỏ hàng chèn một cờ (Flag item) giá trị cao vào. Server chốt đơn mù và cấp cờ dù số dư không đủ

<img width="1212" height="635" alt="image" src="https://github.com/user-attachments/assets/b7d2e3f9-bff1-48b2-8cdc-0233bb5516fe" />

### b. Trong Pentest Doanh nghiệp (Tập trung vào Thiệt hại tài chính)
\- Thương mại điện tử (Limit-overrun): Nhân bản mã giảm giá. Gửi 20 request áp dụng cùng một mã voucher 100k bằng Single-packet. Cả 20 luồng đều vượt qua khâu "Kiểm tra" và trừ thẳng 2.000.000 VNĐ vào hóa đơn trước khi mã đó bị khóa

\- Tài chính / Fintech (Double Spending): Lỗ hổng kinh điển nhất. Người dùng có 10.000.000 VNĐ, đặt lệnh rút toàn bộ số tiền này về 2 số tài khoản ngân hàng khác nhau cùng một tích tắc. Server duyệt cả hai lệnh, dẫn đến việc rút thành công 20.000.000 VNĐ từ số dư ban đầu

\- Bảo mật tài khoản (Rate-limit Bypass): Chức năng nhập mã OTP giới hạn 3 lần sai sẽ khóa tài khoản. Kẻ tấn công gom 50 mã OTP khác nhau vào một gói tin. Hệ thống kiểm tra số lần sai (đang là 0) cùng lúc cho cả 50 luồng, cho phép thử hàng loạt mã bảo mật mà không bị chặn.

\- Một số chức năng nói rằng "Có thể tạo **tối đa** xx sản phẩm, có thể làm **tối đa** yy nhiệm vụ,..."


# Lab 01

<img width="1068" height="523" alt="image" src="https://github.com/user-attachments/assets/f1cd5f46-6c94-4b9a-b263-6c50ed422677" />

\- Với đề bài như trên, ta nghĩ ngay đến việc:
- Thu thập request áp voucher
- Gửi vào repeater
- Tạo nhóm cho 15-20 request
- Single-packer attack



<img width="1863" height="737" alt="image" src="https://github.com/user-attachments/assets/8696242c-cf78-4b8f-b3f0-9bf0637534bc" />

\- Bấm `Send group (parallel)` và khai hoả toàn bộ 17 request áp dụng mã giảm giá, ta được giá siêu mềm (so với budget 50$):

<img width="1209" height="434" alt="image" src="https://github.com/user-attachments/assets/6530e430-b43a-45fe-a5b6-0039b47fe864" />

\- Đặt lệnh mua và hoàn thành bài lab 💗

# Lab 02
<img width="1046" height="587" alt="image" src="https://github.com/user-attachments/assets/a2d86f75-47f4-4fb3-972a-5afe2758ac09" />

\- Bài này yêu cầu ta vượt qua cơ chế rate limit và đăng nhập thành công vào tài khoản carlos

\- Nghĩ đến việc không thể Duplicate 30 request và sửa body của từng request một với wordlist cho trước, ta sử dụng extension Turbo Intruder

\- Chuột phải vào request &rarr; Extensions &rarr; Turbo Intruder &rarr; Send to turbo intruder
- Thay mật khẩu hiện tại của request thành `%s` (vị trí để thay từng dòng của wordlist vào)
- Lưu sẵn wordlist mà lab cho vào bộ nhớ tạm (Ctrl + C)
- Khi này ta craft một [payload](https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/race-single-packet-attack.py) như sau:
```
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2)

    # Lấy từng dòng vừa copy vào khay nhớ tạm
    for word in wordlists.clipboard:
        engine.queue(target.req, word.rstrip(), gate='race1')

    # Gửi đồng loạt
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```

$\implies$ Tìm tới request với status code 302, và kết quả:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e17d484a-3bfa-4e9f-a3a0-d84fbfb88610" />


\- Đã tìm thấy mật khẩu đúng, đăng nhập với credential `carlos:111111`, truy cập admin panel và xoá đi carlos

<img width="991" height="285" alt="image" src="https://github.com/user-attachments/assets/630d4abd-aad7-4b4d-8c31-0a3d77458209" />

# Lab 03
<img width="1079" height="820" alt="image" src="https://github.com/user-attachments/assets/6c0a67a5-fa70-4059-a094-64703ca050e3" />

\- Nhìn vào gợi ý và giao diện mua đồ, mình đoán được sẽ phải mua jacket thông qua một món hàng rẻ hơn, lợi dụng jitter, time latency mà chèn jacket vào request thanh toán
- Trước hết, thêm gift card vào giỏ hàng và thanh toán
- Đưa 2 request `POST /cart` và `POST /cart/checkout` vào Repeater
- Sửa `productID` từ 2 thành 1 (từ gift card thành jacket)
- Thêm sẵn một món đồ mà mình đủ tiền để mua vào trong giỏ hàng
- Tạo group và send group (single-packet attack)

<img width="1853" height="810" alt="image" src="https://github.com/user-attachments/assets/96ade7a9-f4e0-40f6-b464-dfc219260099" />

\- **Dấu hiệu nhận biết lỗ hổng và kiểu khai thác**:
- Khi ứng dụng có nhiều API hoặc endpoint khác nhau cùng trỏ vào và thao tác trên một khối dữ liệu chung của người dùng. Ví dụ: Endpoint `/cart` và `/cart/checkout` cùng can thiệp vào một giỏ hàng
- Các tính năng không "chốt" ngay lập tức trong một luồng duy nhất mà yêu cầu đi qua nhiều bước logic nối tiếp nhau. Ví dụ: `/cart` &rarr; `/cart/voucher` &rarr; `/cart/checkout`
- Áp đặt các "Giới hạn định lượng": Khi server kiểm tra và kiểm duyệt số dư, số lượng hàng, lượt sử dụng voucher,...

\- Thực tế bài lab phân tách một nghiệp vụ thành nhiều bước rời rạc nhưng lại bỏ quên cơ chế "khóa" (lock) dữ liệu tạm thời, khi 2 request được gửi đến server cùng một tích tắc (arrival time), 2 dữ liệu có sự va chạm, dẫn đến việc khi chuẩn bị thanh toán (đã check số dư > số tiền hàng) thì món jacket được thêm vào giỏ hàng, khiến nó được mua trót lọt dù số dư không đủ

# Lab 04
<img width="1069" height="761" alt="image" src="https://github.com/user-attachments/assets/2686c5ba-cc53-448d-a307-6e5c36954059" />

\- Dựa vào tên bài và mô tả, ta có thể lờ mờ đoán được việc gửi request đổi email sẽ dính bug Race condition

\- Với việc khi muốn đổi từ email A sang B, thì request xác nhận đổi email sẽ được gửi về B. Ta thử điền chính email của mình và nhận được:
<img width="1870" height="389" alt="image" src="https://github.com/user-attachments/assets/1654b9cd-0d58-406e-a6f2-72e16c93f9a3" />

\- Bấm vào link và nhận được dòng `Your email has been successfully updated`

\- Lần thử đầu tiên:
- Ta điền yêu cầu đổi email thành email của chính mình
- Lấy 2 request đổi email và xác nhận đổi email
- Tạo group và thực hiện single-packet attack
  <img width="1859" height="392" alt="image" src="https://github.com/user-attachments/assets/14a61d34-811c-4a07-9e4e-d9e666cadd8e" />

  &rarr; Có vẻ như, token đã được sử dụng hoặc đã bị huỷ (trong lần xác nhận trước để trích xuất request)
  
\- Đọc lại đề một lượt, `Single-endpoint` nhằm ám chỉ việc race condition trên 1 endpoint nhất định, có thể server không thể xử lí đa luồng, ném dữ liệu (token, link xác nhận) khi đổi email A sang cho email B

\- Lần thử tiếp theo:
- Gửi yêu cầu đổi email thành 2 email `wiener@exploit` và `carlos@ginandjuice.shop`
- Đưa 2 request vào 1 group
- Thực hiện single-packet attack
- Kiểm tra `Email client`, nếu thất bại thì thử lại

\- Sau 3-4 lần thử, mình đã nhận được email đổi mật khẩu thành `carlos@ginandjuice.shop`

<img width="1861" height="546" alt="image" src="https://github.com/user-attachments/assets/f28e48f0-1b65-47df-bf27-c4330e578af5" />

\- Bấm link xác nhận, truy cập `/admin` và xoá đi carlos 💞

<img width="1000" height="529" alt="image" src="https://github.com/user-attachments/assets/c22ddb43-7df4-4b7c-b2a8-4cb3a2d5ec97" />

**Phỏng đoán**: Có vẻ như khi server gửi xác nhận đổi email, nó lưu email vào chung một biến tạm, khi xử lí lại không khoá, dẫn đến chỉ yêu cầu gần nhất mới có đổi email thành công

&rarr; Như vậy, nếu gửi 2 hoặc nhiều hơn các luồng đổi email đến cùng lúc, server có thể gửi nhầm link xác nhận sang cho 1 bên không mong muốn 

- Luồng 1 (Carlos): Tạo xong Token cho Carlos và đang chuẩn bị gửi mail

- Luồng 2 (Wiener): Nhảy vào ghi đè địa chỉ email của mình vào biến tạm

- Luồng 1: Tiếp tục hành động gửi mail nhưng lại lấy địa chỉ email đã bị ghi đè (của mình) để làm đích đến.

# Lab 05

<img width="1077" height="551" alt="image" src="https://github.com/user-attachments/assets/eda29de6-eba8-4ad4-b346-e49a3b36d99f" />

\- Đọc lại phần docs, ta biết được một cách khai thác khá hay, đó là đưa timestamp đã được hash vào token khi gửi request đổi mật khẩu

\- Thực hiện tương tự:
- Gom 2 request với 2 username khác nhau là `carlos` và `wiener`
- Tạo group và thực hiện single-packet attack
- Truy cập email, lấy link đổi mật khẩu : `https://<EXPLOIT>.web-security-academy.net/forgot-password?user=wiener&token=f7a38b4...`
- Đổi user thành `carlos` và truy cập link

\- Nhưng, ta nhận được `"Invalid token"`, chứng tỏ đã thất bại

\- Đọc lại đề và mô tả, có vẻ như cách hướng khai thác của ta đã đúng. Nhìn kĩ vào request, thấy ở có thêm header `Cookie: phpsessionid=` và có thể đây là chìa khoá để giải bài lab này
- Mỗi session, server chỉ có thể xử lí một luồng cố định, dẫn đến 2 request cùng session sẽ lệch timestamp
- Do đó, từ 2 phiên khác nhau, gửi 2 request đổi mật khẩu vào cùng 1 thời điểm

\- **Khai thác triệt để:**
- Lấy request đổi mật khẩu cho `wiener`
- Trở lại `GET /forgot-password`, xoá đi Cookie
  
  <img width="966" height="210" alt="image" src="https://github.com/user-attachments/assets/736b70f9-4fe8-49fd-93d0-dc689455cade" />
- Gửi lại request đấy, khi này response sẽ trả về cặp `phpsessionid` và `csrf` mới (csrf token ở form `forgot-password` ở dưới cùng của response)
  
  <img width="1611" height="487" alt="image" src="https://github.com/user-attachments/assets/56d4c609-d7e1-4ec3-bd22-f936c98f5599" />


- Duplicate thêm một `POST /forgot-password`, với cookie và csrf token mới cho username `carlos`, khai hoả send parallel
- Đổi biến `?user` thành `carlos`, có thể mất vài lần gửi đi gửi lại, và kết quả:

  <img width="889" height="264" alt="image" src="https://github.com/user-attachments/assets/1f18966b-fedf-4ced-80c2-03859ef06571" />

  # Lab 06

  <img width="1082" height="784" alt="image" src="https://github.com/user-attachments/assets/5b3c19d1-8408-4ec5-8472-21c16b702733" />

  \- Đề bài và mô tả cho ta biết được, đây có thể là một bài sử dụng đến `Turbo Intruder`, craft riêng một payload và khai thác

  \- Khi thực hiện đăng ký tài khoản, đọc source code, tìm được file `users.js`:

```
  const confirmEmail = () => {
    const container = document.getElementsByClassName('confirmation')[0];

    const parts = window.location.href.split("?");
    const query = parts.length == 2 ? parts[1] : "";
    const action = query.includes('token') ? query : "";

    const form = document.createElement('form');
    form.method = 'POST';
    form.action = '/confirm?' + action;

    const button = document.createElement('button');
    button.className = 'button';
    button.type = 'submit';
    button.textContent = 'Confirm';

    form.appendChild(button);
    container.appendChild(form);
}
```
- File này không cho biết logic xử lí token hay xác nhận email như nào, nhưng đã để lộ ra một endpoint là `/confirm` và một biến đi kèm trên URL là `?token`

\- Một ý nghĩ đã được hình thành như sau:
- Liên tục gửi request đăng kí tài khoản
- Server sẽ sinh ra token cho từng tài khoản (không thể brute-force)
- Nhưng sẽ có time gap khi một tài khoản chưa có token (?token=null)
- Nếu cùng lúc có request xác nhận tài khoản và token: null=null✅
&rarr; Khai thác thành công

\- Thực chiến:
- Lấy request `POST /register` &rarr; Extensions &rarr; Turbo Intruder
- Thay username thành `%s`
- Truyền payload:

<details>
    <summary>Payload ban đầu</summary>

```
  def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2)


    req_register = target.req
    req_confirm = '''POST /confirm?token= HTTP/2
Host: 0a6200f104f92033c4cca1ba003e0096.web-security-academy.net
Cookie: phpsessionid=5gY0vfOdj0bmPLzwk8PqbHy8HHMIZlaX
Content-Length: 0
'''
    for i in range(20):
        current_username = "admin" + str(i)
        gate_name = "race_cluster_" + str(i)

        engine.queue(req_register, current_username, gate=gate_name)

        for c in range(50):
            engine.queue(req_confirm, gate=gate_name)
        #  Thả đồng loạt cụm request này đi
        engine.openGate(gate_name)

def handleResponse(req, interesting):
    table.add(req)
```

</details>

\- Nhận được kết quả: 

<img width="1621" height="501" alt="image" src="https://github.com/user-attachments/assets/d174796e-b059-441f-af63-a7aef8c7bd1d" />

&rarr; 1000 request được gửi đi nhưng không nhận về kết quả
Dù đã có cách khai thác và payload đúng hướng nhưng chưa được hưởng trái ngọt

<img width="1638" height="193" alt="image" src="https://github.com/user-attachments/assets/9b17a9fd-2813-482b-819e-3c307625fca9" />

\- Nhận ra rằng, nếu không điền giá trị cho `token`, server lập tức cấm và không qua bước validate `token` chứ chưa nói đến so sánh giá trị của nó

\- Thử Type Confusion, chuyển biến `?token` thành dạng mảng `?token[]`, nhận được
```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 24

"Incorrect token: Array"
```
&rarr; Giờ đây, `token` đã được đem ra để so sánh, dẫn đến response trả ra sai token thay vì bị cấm

\- Sửa payload .py
- Thay thế `current_username` (do đã được sử dụng trong payload đầu tiên nhằm gửi mail xác nhận)
- Đổi `email` thành một giá trị khác (cho chắc, tránh sự trùng lặp)
<details>
    <summary>Payload đã sửa</summary>

```
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2)


    req_register = target.req
    #  Thêm [], biến biến token thành 1 mảng
    req_confirm = '''POST /confirm?token[]= HTTP/2
Host: 0a6200f104f92033c4cca1ba003e0096.web-security-academy.net
Cookie: phpsessionid=vOdYoTWNPOH4rn0ohQbF311NVcJVIS8B
Content-Length: 0
'''
    for i in range(20):
        current_username = "shin" + str(i)
        gate_name = "race_cluster_" + str(i)

        engine.queue(req_register, current_username, gate=gate_name)

        for c in range(50):
            engine.queue(req_confirm, gate=gate_name)
        #  Thả đồng loạt cụm request này đi
        engine.openGate(gate_name)

def handleResponse(req, interesting):
    table.add(req)
```

</details>

\- Đã hoàn thành việc tạo tài khoản, bypass email confirmation

<img width="1617" height="579" alt="image" src="https://github.com/user-attachments/assets/56e4e2ed-810c-406e-b46b-7c0ae1de91ce" />

\- Thực hiện đăng nhập với credential `shin13:1` và truy cập Admin panel 

<img width="1564" height="555" alt="image" src="https://github.com/user-attachments/assets/1832f4c0-95a3-4f6d-9e7f-0409b10f85af" />



  













