
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



  
