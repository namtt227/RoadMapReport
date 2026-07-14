# Integration Testing 

---

## 1. Integration Testing là gì?

Integration Testing (kiểm thử tích hợp) là cấp độ kiểm thử tập trung vào việc kiểm tra **sự tương tác giữa hai hay nhiều thành phần** trong hệ thống khi hoạt động cùng nhau — ví dụ: giữa service và database, giữa các microservice với nhau, giữa backend và một API bên thứ ba.

Khác với unit test (kiểm tra từng phần cô lập, mock hết dependency), integration test cho phép các thành phần **giao tiếp thật** với nhau (hoặc gần giống thật) để phát hiện các lỗi chỉ xuất hiện khi tích hợp — như sai định dạng dữ liệu truyền qua lại, sai cấu hình kết nối, lỗi transaction, race condition...

---

## 2. Các khái niệm cốt lõi

### 2.1 Testing interactions between components 
Kiểm tra luồng dữ liệu và logic khi nhiều thành phần phối hợp, ví dụ:
- Controller gọi Service, Service gọi Repository, Repository thao tác với Database — kiểm tra toàn bộ chuỗi này hoạt động đúng.
- Service A gọi API của Service B (trong kiến trúc microservices) — kiểm tra request/response đúng định dạng, đúng logic nghiệp vụ.

Phạm vi integration test rộng hơn unit test nhưng hẹp hơn E2E test — nó thường không đi qua giao diện người dùng (UI) mà kiểm tra trực tiếp ở tầng service/API.

### 2.2 Database Testing
Kiểm tra các thao tác với database ở mức thực tế:
- **CRUD operations**: Create, Read, Update, Delete có hoạt động đúng không.
- **Transaction**: đảm bảo tính toàn vẹn khi có nhiều thao tác trong một transaction (commit/rollback đúng).
- **Constraint & schema**: kiểm tra ràng buộc dữ liệu (unique, foreign key, not null) có được database enforce đúng không.
- **Migration**: kiểm tra script migration chạy đúng khi thay đổi schema.

Hai chiến lược phổ biến:
| Chiến lược | Mô tả | Ưu điểm | Nhược điểm |
|---|---|---|---|
| Dùng database thật (qua container) | Test chạy trên database giống hệt production | Độ tin cậy cao | Chạy chậm hơn, cần setup |
| Dùng in-memory database (VD: H2, SQLite) | Database giả lập chạy trong bộ nhớ | Nhanh | Có thể khác hành vi so với DB thật (VD: PostgreSQL) |

---

## 3. Công cụ sử dụng

### 3.1 Postman
- Công cụ gửi HTTP request (GET, POST, PUT, DELETE...) để kiểm tra API endpoint theo cách thủ công hoặc tự động.
- Cho phép viết **test script** bằng JavaScript ngay trong Postman để assert response:
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has correct user id", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(1);
});
```
- Hỗ trợ **Collection** (tập hợp nhiều request) và **Environment** (biến môi trường như base URL, token) để tổ chức test theo từng môi trường (dev/staging/production).
- Có thể chạy tự động qua **Newman** (CLI runner của Postman) để tích hợp vào CI/CD pipeline.

### 3.2 Testcontainers
- Thư viện cho phép khởi tạo các service phụ thuộc (database, message queue, cache...) dưới dạng **Docker container tạm thời** ngay trong quá trình chạy test.
- Giúp test integration chạy trên môi trường **giống hệt production thật** (VD: PostgreSQL thật, Redis thật) thay vì dùng bản giả lập (in-memory), tăng độ tin cậy của test.
- Container tự động khởi tạo trước khi test chạy và tự động dọn dẹp (teardown) sau khi test xong — đảm bảo test không để lại dữ liệu rác.

Ví dụ (Java, dùng Testcontainers với PostgreSQL):
```java
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:15");

    @Test
    void testSaveUser() {
        // Kết nối tới postgres.getJdbcUrl() và test thao tác lưu user
    }
}
```
- Testcontainers cũng hỗ trợ nhiều ngôn ngữ khác: Python (`testcontainers-python`), Node.js (`testcontainers-node`), Go.

---

## 4. Kiểm thử API Endpoint

Khi test API endpoint ở cấp độ integration, cần kiểm tra đầy đủ các khía cạnh:
1. **Status code**: đúng mã trạng thái HTTP tương ứng (200, 201, 400, 404, 500...).
2. **Response schema**: cấu trúc và kiểu dữ liệu của response đúng như tài liệu API (OpenAPI/Swagger).
3. **Business logic**: dữ liệu trả về đúng theo nghiệp vụ (VD: tạo đơn hàng thành công thì trạng thái đơn phải là "pending").
4. **Authentication/Authorization**: kiểm tra endpoint có yêu cầu token đúng, từ chối truy cập trái phép.
5. **Error handling**: kiểm tra hệ thống trả lỗi hợp lý khi input sai hoặc thiếu.



