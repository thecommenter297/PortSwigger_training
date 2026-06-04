## Lấy Chrome làm trình duyệt thay vì Chromium


Trình duyệt Google Chrome hỗ trợ các tham số dòng lệnh (command-line flags) rất mạnh mẽ. Chúng ta có thể tận dụng chúng để viết một script ngắn (bằng file Batch/Bash hoặc trực tiếp bằng Python). Khi chạy, script này sẽ:
1. Tự động định tuyến toàn bộ traffic qua Burp (`127.0.0.1:8080`).
2. Tự động bỏ qua mọi cảnh báo lỗi SSL/Certificate (không cần cài chứng chỉ thủ công).
3. Tự động tạo một profile người dùng sạch sẽ (không chung đụng với tài khoản Chrome cá nhân của bạn, tránh bị lộ lịch sử hay cookie cá nhân khi pentest).

Dưới đây là file script bạn có thể lưu lại để dùng ngay:

### Sử dụng Script Python (Chạy được trên cả Windows, macOS, Linux)

Nếu bạn thích dùng Python, bạn hãy tạo một file tên là `launch_burp_chrome.py` và dán đoạn code sau vào:

```python
import os
import platform
import subprocess


def launch_chrome():
    system = platform.system()

    # Xác định đường dẫn mặc định của Google Chrome theo từng hệ điều hành
    if system == "Windows":
        chrome_path = r"C:\Program Files\Google\Chrome\Application\chrome.exe"
        # Tạo thư mục lưu profile tạm thời trên Windows
        profile_dir = os.path.join(
            os.environ["LOCALAPPDATA"], "Google", "Chrome", "BurpProfile"
        )
    elif system == "Darwin":  # macOS
        chrome_path = (
            "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
        )
        profile_dir = os.path.expanduser(
            "~/Library/Application Support/Google/Chrome/BurpProfile"
        )
    else:  # Linux
        chrome_path = "google-chrome"
        profile_dir = os.path.expanduser("~/.config/google-chrome/BurpProfile")

    # Các cờ cấu hình tự động
    flags = [
        "--proxy-server=http://127.0.0.1:8080",  # Trỏ proxy về Burp
        "--ignore-certificate-errors",  # Tự bỏ qua lỗi HTTPS (Không cần cài Certificate)
        f"--user-data-dir={profile_dir}",  # Tách biệt profile để tránh xung đột với Chrome chính
        "https://www.google.com",  # Trang mở sẵn khi khởi động
    ]

    print("[*] Đang khởi chạy Google Chrome dành cho Burp Suite...")
    try:
        subprocess.Popen([chrome_path] + flags)
    except FileNotFoundError:
        print(
            "[-] Không tìm thấy Google Chrome tại đường dẫn mặc định. Hãy kiểm tra lại."
        )


if __name__ == "__main__":
    launch_chrome()
```

---

### Cài chứng chỉ của Burp Suite vào profile Chrome

* Truy cập `http://burp` hoặc `http://127.0.0.1:8080`
* Click vào nút **CA Certificate** ở góc trên cùng bên phải để tải file chứng chỉ về máy (thường là file `cacert.der`)
* Trên Chrome này, truy cập vào phần quản lý chứng chỉ bằng cách dán đường dẫn này vào thanh địa chỉ `chrome://settings/security`
* Tìm chọn **Manage certificates** (Quản lý chứng chỉ). Tìm cách **Import** chứng chỉ **CA Certificate** (`cacert.der`) đã tải dưới dạng Trusted Ceritficate.
