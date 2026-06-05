## Tạo file script Python để gửi request đến Lab trên PortSwigger (có sử dụng Burp Suite)

* Nhấn **Copy as curl command (bash)**

<img width="1913" height="970" alt="image" src="https://github.com/user-attachments/assets/e4885ea3-b7da-4ca7-a7e2-72b922a77f4f" />

* Vào trang web <a href="https://curlconverter.com/">https://curlconverter.com/</a> và Paste đoạn mã `curl` vừa copy. Sau đó nhấn **Copy to Clipboard**.

<img width="1654" height="883" alt="image" src="https://github.com/user-attachments/assets/bcc93833-0ef5-4fd3-ae3c-f98008157c0f" />

* Dán đoạn mã kia vào file `sqli.py` dưới đây:


```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# ================ Đoạn mã copy từ trang curlconverter ===============

# -> PASTE đoạn vừa COPY vào đây

# ====================================================================


# Xử lý dữ liệu (Đoạn này tự tùy chỉnh)

print(response.text)

if ("Internal Server Error" in response.text):
    print("Error: Internal Server Error")
else:
    print("Sucess")
```


