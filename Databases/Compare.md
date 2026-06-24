# So Sánh SQL và NoSQL

## 1. Tổng Quan

| Tiêu chí | SQL | NoSQL |
|----------|-----|-------|
| Mô hình dữ liệu | Bảng (table) | Document, Key-Value, Column, Graph |
| Schema | Cố định | Linh hoạt |
| Ngôn ngữ truy vấn | SQL chuẩn |Đa Dạng Tùy hệ thống |
| Tính nhất quán | ACID | BASE |
| Mở rộng | Vertical scaling | Horizontal scaling |

---

## 2. SQL

**Ưu điểm:**
- Cấu trúc rõ ràng, dễ bảo trì
- ACID đảm bảo toàn vẹn dữ liệu
- Quan hệ giữa bảng chặt chẽ
- Cộng đồng lớn, tài liệu phong phú

**Nhược điểm:**
- Schema cứng, khó thay đổi
- Khó mở rộng ngang (horizontal scaling)
- Hiệu suất giảm khi dữ liệu rất lớn

**Dùng khi:** Dữ liệu có quan hệ phức tạp, yêu cầu giao dịch, hệ thống tài chính.

---

## 3. NoSQL

**Ưu điểm:**
- Schema linh hoạt, dễ thay đổi
- Mở rộng ngang dễ dàng
- Tốc độ đọc/ghi cao
- Phù hợp dữ liệu phi cấu trúc

**Nhược điểm:**
- Không đảm bảo ACID đầy đủ
- Truy vấn phức tạp hạn chế
- Thiếu chuẩn chung giữa các hệ thống

**Dùng khi:** Dữ liệu lớn, cần scale nhanh, cấu trúc thay đổi thường xuyên.

---

## 4. Ứng dụng

| Lĩnh Vực | |
|------------|------|
| Hệ thống ngân hàng, thanh toán | SQL |
| Mạng xã hội, feed người dùng | NoSQL |
| ERP, quản lý doanh nghiệp | SQL |
| Cache, session, real-time | NoSQL (Redis) |
| Báo cáo, analytics dữ liệu lớn | NoSQL (ClickHouse) |
| Lưu trữ hồ sơ người dùng linh hoạt | NoSQL (MongoDB) |
| Ứng dụng cần JOIN phức tạp | SQL |