# Báo Cáo: Relational Databases (SQL)

---

## 1. Giới Thiệu

**Relational Databases** (Cơ sở dữ liệu quan hệ) là hệ thống lưu trữ dữ liệu theo dạng bảng (table) có cấu trúc, với các mối quan hệ rõ ràng giữa các bảng. SQL (Structured Query Language) là ngôn ngữ chuẩn để truy vấn và thao tác dữ liệu trong các hệ thống này.


---

## 2. Concepts

### 2.1 SQL Queries — CRUD

CRUD là bốn thao tác cơ bản trên dữ liệu: **Create**, **Read**, **Update**, **Delete**.

VD:
```sql
-- CREATE: Thêm dữ liệu mới
INSERT INTO users (name, email, age)
VALUES ('Nguyễn Văn A', 'a@example.com', 25);

-- READ: Truy vấn dữ liệu
SELECT name, email FROM users
WHERE age > 18
ORDER BY name ASC;

-- UPDATE: Cập nhật dữ liệu
UPDATE users
SET email = 'new@example.com'
WHERE id = 1;

-- DELETE: Xóa dữ liệu
DELETE FROM users
WHERE id = 1;
```

### 2.2 SQL Queries — Joins

JOIN cho phép kết hợp dữ liệu từ nhiều bảng dựa trên điều kiện liên kết.

| Loại JOIN | Mô tả |
|-----------|-------|
| `INNER JOIN` | Chỉ lấy hàng khớp ở **cả hai** bảng |
| `LEFT JOIN` | Lấy tất cả hàng bảng trái, NULL nếu không khớp bên phải |
| `RIGHT JOIN` | Lấy tất cả hàng bảng phải, NULL nếu không khớp bên trái |
| `FULL OUTER JOIN` | Lấy tất cả hàng của cả hai bảng |
| `CROSS JOIN` | Tích Descartes — mọi tổ hợp hàng giữa hai bảng |

```sql
-- INNER JOIN: lấy đơn hàng kèm thông tin khách hàng
SELECT o.id, o.total, u.name, u.email
FROM orders o
INNER JOIN users u ON o.user_id = u.id;

-- LEFT JOIN: lấy tất cả users, kể cả chưa có đơn hàng
SELECT u.name, COUNT(o.id) AS total_orders
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```

### 2.3 SQL Queries — Subqueries

Subquery (truy vấn con) là câu lệnh SELECT lồng bên trong một câu lệnh SQL khác.

```sql
-- Subquery trong WHERE
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) FROM employees
);

-- Subquery trong FROM 
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_avg
WHERE avg_sal > 50000;

-- Subquery với EXISTS
SELECT name FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

### 2.4 Normalization (Chuẩn hóa dữ liệu)

Normalization là quá trình tổ chức dữ liệu để giảm trùng lặp và đảm bảo tính toàn vẹn.

| Dạng chuẩn | Yêu cầu |
|------------|---------|
| **1NF** | Mỗi ô chứa một giá trị nguyên tử, không có nhóm lặp |
| **2NF** | Đạt 1NF + mọi thuộc tính phụ thuộc hoàn toàn vào khóa chính |
| **3NF** | Đạt 2NF + không có phụ thuộc bắc cầu (transitive dependency) |
| **BCNF** | Dạng mạnh hơn 3NF, mọi determinant đều là superkey |

**Ví dụ vi phạm 1NF và cách sửa:**

```
-- Sai (vi phạm 1NF): nhiều giá trị trong một ô
| order_id | products          |
|----------|-------------------|
| 1        | Laptop, Chuột     |

-- Đúng (đạt 1NF): tách thành nhiều hàng
| order_id | product |
|----------|---------|
| 1        | Laptop  |
| 1        | Chuột   |
```

### 2.5 Transactions (Giao dịch)

Transaction là một nhóm thao tác SQL được thực thi như một đơn vị nguyên tử — hoặc tất cả thành công, hoặc tất cả thất bại.

**Thuộc tính ACID:**

| Thuộc tính | Ý nghĩa |
|------------|---------|
| **Atomicity** | Tất cả hoặc không có gì được thực thi |
| **Consistency** | Dữ liệu luôn ở trạng thái hợp lệ trước và sau transaction |
| **Isolation** | Các transaction không ảnh hưởng lẫn nhau |
| **Durability** | Kết quả được lưu vĩnh viễn sau khi COMMIT |

```sql
BEGIN;

UPDATE accounts SET balance = balance - 1000000
WHERE id = 1;  -- Trừ tiền người gửi

UPDATE accounts SET balance = balance + 1000000
WHERE id = 2;  -- Cộng tiền người nhận

-- Nếu có lỗi xảy ra ở bước nào → hủy toàn bộ
ROLLBACK;

-- Nếu tất cả thành công → xác nhận
COMMIT;
```

### 2.6 Indexing (Chỉ mục)

Index giúp tăng tốc độ truy vấn bằng cách tạo cấu trúc tra cứu nhanh, tương tự mục lục của sách.

```sql
-- Tạo index đơn
CREATE INDEX idx_users_email ON users(email);

-- Tạo composite index (nhiều cột)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Tạo unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Xem các index của bảng (PostgreSQL)
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'users';

-- Xóa index
DROP INDEX idx_users_email;
```

**Khi nào nên dùng Index:**

- Cột thường xuất hiện trong mệnh đề `WHERE`, `JOIN ON`, `ORDER BY`
- Bảng có lượng dữ liệu lớn (hàng chục nghìn bản ghi trở lên)
- Cột có tính phân biệt cao (nhiều giá trị khác nhau)

**Lưu ý:** Index tăng tốc đọc nhưng làm chậm ghi (`INSERT`, `UPDATE`, `DELETE`) vì cần cập nhật cả index.

---

## 3. Hệ Quản Trị Cơ Sở Dữ Liệu (Systems)

### 3.1 PostgreSQL

PostgreSQL là hệ quản trị CSDL quan hệ mã nguồn mở mạnh mẽ nhất hiện nay, nổi tiếng với tính tuân thủ chuẩn SQL và khả năng mở rộng cao.

**Đặc điểm nổi bật:**

| Tính năng | Mô tả |
|-----------|-------|
| ACID compliant | Hỗ trợ đầy đủ transaction |
| JSON/JSONB | Lưu trữ và truy vấn dữ liệu bán cấu trúc |
| Full-text search | Tìm kiếm văn bản tích hợp sẵn |
| Extensions | `PostGIS` (địa lý), `pg_trgm` (tìm kiếm mờ)... |
| Partitioning | Chia bảng lớn thành các phân vùng |
| Replication | Hỗ trợ streaming replication |

```sql
-- Ví dụ PostgreSQL: truy vấn JSON
SELECT data->>'name' AS name
FROM products
WHERE data->>'category' = 'electronics';

-- Window function (PostgreSQL mạnh về điều này)
SELECT name, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;

-- CTE (Common Table Expression)
WITH top_customers AS (
    SELECT user_id, SUM(total) AS revenue
    FROM orders
    GROUP BY user_id
    ORDER BY revenue DESC
    LIMIT 10
)
SELECT u.name, t.revenue
FROM top_customers t
JOIN users u ON t.user_id = u.id;
```

### 3.2 ClickHouse

**ClickHouse** là hệ quản trị CSDL cột (columnar database) mã nguồn mở được phát triển bởi Yandex, tối ưu cho phân tích dữ liệu lớn (OLAP) với tốc độ truy vấn cực nhanh.

**Đặc điểm nổi bật:**

| Tính năng | Mô tả |
|-----------|-------|
| Columnar storage | Lưu trữ theo cột, nén dữ liệu tốt hơn |
| OLAP tối ưu | Xử lý hàng tỷ bản ghi trong vài giây |
| Vectorized execution | Xử lý dữ liệu theo batch, tận dụng CPU |
| Real-time ingestion | Nhận dữ liệu liên tục với độ trễ thấp |
| Horizontal scaling | Mở rộng bằng cách thêm node |
| SQL support | Hỗ trợ phần lớn cú pháp SQL chuẩn |

```sql
-- Tạo bảng trong ClickHouse
CREATE TABLE events (
    event_date Date,
    user_id    UInt64,
    event_type String,
    properties Map(String, String)
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (user_id, event_date);

-- Truy vấn phân tích nhanh trên hàng tỷ bản ghi
SELECT
    toStartOfDay(event_date) AS day,
    event_type,
    COUNT()                  AS cnt,
    uniq(user_id)            AS unique_users
FROM events
WHERE event_date BETWEEN '2024-01-01' AND '2024-03-31'
GROUP BY day, event_type
ORDER BY day, cnt DESC;
```



## 4. Tools

### 4.1 DBeaver

**DBeaver** là công cụ quản lý CSDL đa nền tảng mã nguồn mở, hỗ trợ hầu hết các loại database phổ biến qua giao diện đồ họa (GUI) thân thiện.

**Tính năng chính:**

| Tính năng | Mô tả |
|-----------|-------|
| Multi-database | Kết nối PostgreSQL, MySQL, ClickHouse, SQLite, Oracle... |
| SQL Editor | Gợi ý code (autocomplete), highlight cú pháp |
| ER Diagram | Tự động vẽ sơ đồ quan hệ giữa các bảng |
| Data Export | Xuất dữ liệu ra CSV, Excel, JSON, SQL |
| Query History | Lưu lịch sử các câu lệnh đã chạy |
| Data Compare | So sánh dữ liệu giữa hai database |

**Kết nối PostgreSQL trong DBeaver:**

```
Host:     localhost
Port:     5432
Database: my_database
Username: postgres
Password: ********
```

**Kết nối ClickHouse trong DBeaver:**

```
Host:     localhost
Port:     8123 (HTTP) hoặc 9000 (Native)
Database: default
Username: default
Password: ********
```

