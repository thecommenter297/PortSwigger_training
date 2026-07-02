   - Thử ' -> xác định có lỗi sqli
   - tấn công union based để dò tìm số cột cụ thể của bảng hiện tại
   - dò tìm cột cụ thể có kiểu char
   - dò tìm phiên bản cụ thể của database server
   - dựa vào cấu trúc bảng này, tra cứu tất cả các tên bảng và tên cột nằm trong cấu trúc information_schema của database: 
   
   <img width="1007" height="408" alt="Screenshot 2026-06-24 125238" src="https://github.com/user-attachments/assets/6ccf93ee-cc5d-45f5-813c-e48be24a3674" />

   - Sau khi đã truy ra tên bảng là "users_..." bằng "Pets' UNION SELECT table_name, NULL FROM information_schema.tables -- " (chú ý việc dump tên bảng dùng table_name),
   => Truy tìm các tên cột trong bảng đó bằng "Pets' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_bcejng' -- ",

   - Sau khi đã có tên các cột, ta dump ra giá trị các cột mình muốn bằng "Pets' UNION SELECT username_..., password_... FROM users_bcejng -- ",
   
