#  RESTful APIs

---

## 1. Giới Thiệu

**REST** (Representational State Transfer) là kiến trúc thiết kế API phổ biến nhất hiện nay, sử dụng giao thức HTTP để giao tiếp giữa client và server. API tuân thủ nguyên tắc REST được gọi là **RESTful API**.

RESTful API là nền tảng của hầu hết các ứng dụng web và mobile hiện đại — từ mạng xã hội, thương mại điện tử đến các hệ thống doanh nghiệp.

---

## 2. Concepts

### 2.1 HTTP Methods

HTTP Methods (hay HTTP Verbs) định nghĩa hành động muốn thực hiện trên resource.

| Method | Hành động | Ví dụ |
|--------|-----------|-------|
| `GET` | Lấy dữ liệu | Lấy danh sách sản phẩm |
| `POST` | Tạo mới | Tạo đơn hàng mới |
| `PUT` | Cập nhật toàn bộ | Cập nhật toàn bộ thông tin user |
| `PATCH` | Cập nhật một phần | Chỉ cập nhật email user |
| `DELETE` | Xóa | Xóa sản phẩm |

```
GET    /api/products        → Lấy danh sách
GET    /api/products/1      → Lấy 1 sản phẩm
POST   /api/products        → Tạo mới
PUT    /api/products/1      → Cập nhật toàn bộ
PATCH  /api/products/1      → Cập nhật một phần
DELETE /api/products/1      → Xóa
```

---

### 2.2 Status Codes

HTTP Status Code là mã số server trả về để client biết kết quả của request.

**2xx — Thành công:**

| Code | Tên | Dùng khi |
|------|-----|---------|
| `200 OK` | Thành công | GET, PUT, PATCH thành công |
| `201 Created` | Tạo mới thành công | POST tạo resource mới |
| `204 No Content` | Thành công, không có data | DELETE thành công |

**4xx — Lỗi từ client:**

| Code | Tên | Dùng khi |
|------|-----|---------|
| `400 Bad Request` | Request sai | Thiếu field, sai định dạng |
| `401 Unauthorized` | Chưa xác thực | Chưa đăng nhập, thiếu token |
| `403 Forbidden` | Không có quyền | Đã đăng nhập nhưng không đủ quyền |
| `404 Not Found` | Không tìm thấy | Resource không tồn tại |
| `409 Conflict` | Xung đột | Email đã được đăng ký |
| `422 Unprocessable` | Dữ liệu không hợp lệ | Validation thất bại |

**5xx — Lỗi từ server:**

| Code | Tên | Dùng khi |
|------|-----|---------|
| `500 Internal Server Error` | Lỗi server | Lỗi không xác định |
| `502 Bad Gateway` | Gateway lỗi | Reverse proxy không kết nối được backend |
| `503 Service Unavailable` | Service không khả dụng | Server quá tải hoặc đang bảo trì |

---

### 2.3 Statelessness (Phi trạng thái)

**Stateless** là nguyên tắc quan trọng nhất của REST — mỗi request phải chứa đủ thông tin để server xử lý, server không lưu trạng thái của client giữa các request.

```
Stateful (không phải REST):
Request 1: Đăng nhập → Server lưu session
Request 2: Lấy data  → Server nhớ user từ session

Stateless (REST):
Request 1: GET /products (kèm token)  → Server xác thực token, trả data
Request 2: GET /orders   (kèm token)  → Server xác thực token, trả data
           ↑ Mỗi request tự mang thông tin xác thực
```

**Lợi ích của Stateless:**
- Dễ scale horizontal (thêm server mà không cần sync session)
- Dễ cache response
- Đơn giản hóa server

---

### 2.4 Resources

**Resource** là đối tượng dữ liệu được quản lý qua API. Mỗi resource có một URL định danh duy nhất.

```
Resource: User
URL:      /api/users
          /api/users/{id}

Resource: Product
URL:      /api/products
          /api/products/{id}

Resource lồng nhau (nested):
/api/users/{id}/orders          → Đơn hàng của user
/api/categories/{id}/products   → Sản phẩm thuộc danh mục
```

**Cấu trúc response chuẩn:**

```json
{
  "success": true,
  "message": "Lấy danh sách thành công",
  "data": {
    "items": [...],
    "total": 100,
    "page": 1,
    "limit": 10
  }
}
```

```json
{
  "success": false,
  "message": "Email already registered",
  "error": "CONFLICT"
}
```

---

### 2.5 Idempotency (Tính bất biến)

**Idempotent** — gọi nhiều lần vẫn cho kết quả giống nhau.

| Method | Idempotent? | Giải thích |
|--------|-------------|-----------|
| `GET` | Có | Gọi 100 lần vẫn trả về cùng data |
| `PUT` | Có | Update toàn bộ resource, kết quả như nhau |
| `DELETE` | Có | Xóa rồi xóa lại → vẫn là đã xóa |
| `PATCH` |  | Tùy implementation |
| `POST` | Không | Gọi 3 lần → tạo 3 record mới |

**Tại sao quan trọng?** Khi mạng chập chờn, client có thể retry request. Nếu method là idempotent thì retry an toàn, không gây ra dữ liệu trùng lặp.

---

## 3. Tools

### 3.1 Postman

**Postman** là công cụ GUI phổ biến nhất để test và document API.

**Tính năng chính:**

| Tính năng | Mô tả |
|-----------|-------|
| Request Builder | Tạo request với method, headers, body |
| Collections | Nhóm các request theo module |
| Environment | Quản lý biến môi trường (base URL, token) |
| Auto Tests | Viết script kiểm tra response tự động |
| API Documentation | Tự động sinh tài liệu từ collection |
| Mock Server | Tạo server giả để frontend test khi backend chưa xong |

**Ví dụ sử dụng Postman:**

```
1. Tạo Collection "GoShop API"
2. Tạo Environment:
   - base_url: http://localhost:8082/api/v1
   - token: (để trống, điền sau khi login)

3. Request Login:
   POST {{base_url}}/auth/login
   Body: { "email": "...", "password": "..." }
   Test script: pm.environment.set("token", pm.response.json().data.token)

4. Request lấy danh sách sản phẩm:
   GET {{base_url}}/products
   Headers: Authorization: Bearer {{token}}
```

---

### 3.2 cURL

**cURL** là công cụ command-line để gửi HTTP request, có sẵn trên mọi hệ điều hành — không cần cài đặt gì thêm trên Linux/macOS.

**Các lệnh cURL hay dùng:**

```bash
# GET
curl http://localhost:8082/api/v1/products

# GET với header Authorization
curl -H "Authorization: Bearer <token>" \
     http://localhost:8082/api/v1/orders

# POST với JSON body
curl -X POST http://localhost:8082/api/v1/auth/register \
     -H "Content-Type: application/json" \
     -d '{"name":"Nguyen Van A","email":"a@gmail.com","password":"123456"}'

# PUT — cập nhật toàn bộ
curl -X PUT http://localhost:8082/api/v1/products/1 \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <token>" \
     -d '{"name":"iPhone 16","price":30000000}'

# PATCH — cập nhật một phần
curl -X PATCH http://localhost:8082/api/v1/orders/1/cancel \
     -H "Authorization: Bearer <token>"

# DELETE
curl -X DELETE http://localhost:8082/api/v1/products/1 \
     -H "Authorization: Bearer <token>"

# Xem response header
curl -I http://localhost:8082/api/v1/products

# Format JSON output (cần jq)
curl http://localhost:8082/api/v1/products | jq
```

**So sánh Postman và cURL:**

| Tiêu chí | Postman | cURL |
|----------|---------|------|
| Giao diện | GUI | CLI |
| Dễ dùng | Rất dễ | Cần nhớ cú pháp |
| Tự động hóa | Script | Shell script |
| Chia sẻ | Collection export | Copy command |
| Phù hợp | Explore API, document | CI/CD, automation |

---

## 4. Projects

### 4.1 Build a Complete CRUD API

Xây dựng RESTful API đầy đủ cho một resource — ví dụ **Books API**.

**Thiết kế endpoints:**

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| `GET` | `/api/books` | Lấy danh sách sách (có phân trang, filter) |
| `GET` | `/api/books/:id` | Lấy chi tiết 1 cuốn sách |
| `POST` | `/api/books` | Tạo sách mới |
| `PUT` | `/api/books/:id` | Cập nhật toàn bộ thông tin sách |
| `PATCH` | `/api/books/:id` | Cập nhật một phần |
| `DELETE` | `/api/books/:id` | Xóa sách |

**Ví dụ implementation với Gin (Go):**

```go
type Book struct {
    ID          uint    `json:"id"`
    Title       string  `json:"title" binding:"required"`
    Author      string  `json:"author" binding:"required"`
    Price       float64 `json:"price" binding:"required"`
    PublishedAt string  `json:"published_at"`
}

// GET /api/books?page=1&limit=10&author=nguyen
func ListBooks(c *gin.Context) {
    page, _  := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "10"))
    author   := c.Query("author")

    var books []Book
    query := db.Offset((page - 1) * limit).Limit(limit)
    if author != "" {
        query = query.Where("author ILIKE ?", "%"+author+"%")
    }
    query.Find(&books)

    c.JSON(200, gin.H{"data": books, "page": page, "limit": limit})
}

// POST /api/books
func CreateBook(c *gin.Context) {
    var book Book
    if err := c.ShouldBindJSON(&book); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    db.Create(&book)
    c.JSON(201, gin.H{"data": book})
}

// DELETE /api/books/:id
func DeleteBook(c *gin.Context) {
    id := c.Param("id")
    if err := db.Delete(&Book{}, id).Error; err != nil {
        c.JSON(404, gin.H{"error": "Book not found"})
        return
    }
    c.JSON(204, nil)
}
```

**Checklist một CRUD API hoàn chỉnh:**

- [ ] Đầy đủ 5 endpoints (List, Get, Create, Update, Delete)
- [ ] Validate input (required fields, định dạng đúng)
- [ ] Trả về đúng HTTP status code
- [ ] Xử lý lỗi rõ ràng (not found, conflict, server error)
- [ ] Phân trang cho danh sách (page, limit)
- [ ] Filter và sort cơ bản
- [ ] Authentication (JWT hoặc API Key)
- [ ] Document bằng Postman Collection

---

## 5. Tổng Kết

| Nhóm | Nội dung |
|------|----------|
| **Concepts** | HTTP Methods (GET/POST/PUT/DELETE), Status Codes (2xx/4xx/5xx), Statelessness, Resources, Idempotency |
| **Tools** | Postman (GUI, document, test), cURL (CLI, automation) |
| **Projects** | CRUD API hoàn chỉnh cho Books, Users |

