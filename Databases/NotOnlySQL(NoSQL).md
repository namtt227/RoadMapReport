# Báo Cáo: NoSQL Databases

---

## 1. Giới Thiệu

**NoSQL** (Not Only SQL) là nhóm hệ quản trị cơ sở dữ liệu không sử dụng mô hình quan hệ truyền thống. NoSQL được thiết kế để xử lý dữ liệu phi cấu trúc hoặc bán cấu trúc, phù hợp với các ứng dụng yêu cầu khả năng mở rộng cao, độ trễ thấp và linh hoạt về schema.

**Khi nào chọn NoSQL thay vì SQL:**

| Tiêu chí | SQL | NoSQL |
|----------|-----|-------|
| Cấu trúc dữ liệu | Cố định (schema) | Linh hoạt (schema-less) |
| Quan hệ phức tạp | Tốt | Hạn chế |
| Khả năng mở rộng | Vertical scaling | Horizontal scaling |
| Tốc độ đọc/ghi | Trung bình | Rất nhanh |
| Tính nhất quán | ACID | BASE (Eventually consistent) |

Báo cáo này trình bày ba nhóm chính: **Systems** (MongoDB, Redis), **Tools** (MongoDB Compass, RedisInsight) và **Projects** (các dự án thực tế).

---

## 2. Hệ Quản Trị Cơ Sở Dữ Liệu (Systems)

### 2.1 MongoDB

**MongoDB** là hệ quản trị CSDL hướng tài liệu (document-oriented) phổ biến nhất hiện nay. Dữ liệu được lưu dưới dạng **BSON** (Binary JSON), cho phép lưu trữ các cấu trúc lồng nhau phức tạp.

**Đặc điểm nổi bật:**

| Tính năng | Mô tả |
|-----------|-------|
| Document model | Lưu dữ liệu dạng JSON linh hoạt |
| Schema-less | Không cần định nghĩa cấu trúc trước |
| Aggregation Pipeline | Xử lý và phân tích dữ liệu mạnh mẽ |
| Sharding | Phân tán dữ liệu ngang hàng |
| Replication | Replica Set đảm bảo high availability |
| Atlas Search | Tìm kiếm full-text tích hợp |

**Các thao tác cơ bản với MongoDB:**

```javascript
// Kết nối và chọn database
use my_database

// INSERT — Thêm document
db.users.insertOne({
  name: "Nguyễn Văn A",
  email: "a@example.com",
  age: 25,
  address: {
    city: "Hà Nội",
    district: "Hai Bà Trưng"
  },
  hobbies: ["đọc sách", "lập trình"]
})

// INSERT nhiều documents
db.users.insertMany([
  { name: "Trần Thị B", age: 22 },
  { name: "Lê Văn C",   age: 30 }
])

// READ — Truy vấn
db.users.find({ age: { $gt: 20 } })         // age > 20
db.users.find({ "address.city": "Hà Nội" }) // truy vấn lồng nhau
db.users.findOne({ email: "a@example.com" })

// UPDATE
db.users.updateOne(
  { email: "a@example.com" },
  { $set: { age: 26 }, $push: { hobbies: "du lịch" } }
)

// DELETE
db.users.deleteOne({ email: "a@example.com" })
db.users.deleteMany({ age: { $lt: 18 } })

// INDEX
db.users.createIndex({ email: 1 }, { unique: true })
db.users.createIndex({ name: "text" }) // full-text search
```

**Aggregation Pipeline:**

```javascript
// Thống kê người dùng theo thành phố
db.users.aggregate([
  { $match: { age: { $gte: 18 } } },          // Lọc người >= 18 tuổi
  { $group: {
      _id: "$address.city",
      count: { $sum: 1 },
      avg_age: { $avg: "$age" }
  }},
  { $sort: { count: -1 } },                    // Sắp xếp giảm dần
  { $limit: 5 }                                // Lấy top 5
])
```

---

### 2.2 Redis

**Redis** (Remote Dictionary Server) là hệ thống lưu trữ dữ liệu **in-memory** dạng key-value, nổi tiếng với tốc độ cực nhanh (dưới 1ms). Redis thường được dùng làm cache, message broker hoặc session store.

**Các kiểu dữ liệu Redis hỗ trợ:**

| Kiểu dữ liệu | Lệnh chính | Ứng dụng |
|--------------|------------|----------|
| **String** | `SET`, `GET`, `INCR` | Cache, counter, session |
| **Hash** | `HSET`, `HGET`, `HGETALL` | User profile, config |
| **List** | `LPUSH`, `RPUSH`, `LRANGE` | Queue, message feed |
| **Set** | `SADD`, `SMEMBERS`, `SINTER` | Tags, unique visitors |
| **Sorted Set** | `ZADD`, `ZRANGE`, `ZRANK` | Leaderboard, ranking |
| **Stream** | `XADD`, `XREAD` | Event log, real-time feed |

**Các lệnh Redis cơ bản:**

```bash
# String — lưu cache với TTL
SET user:1:name "Nguyễn Văn A"
GET user:1:name
SET session:abc123 "user_data" EX 3600    # hết hạn sau 1 giờ
TTL session:abc123                         # xem thời gian còn lại

# Counter
INCR page:views:homepage
INCRBY page:views:homepage 5

# Hash — lưu object
HSET user:1 name "Nguyễn Văn A" age 25 email "a@example.com"
HGET user:1 name
HGETALL user:1

# List — queue/stack
RPUSH queue:emails "email_1" "email_2"
LPOP queue:emails          # lấy và xóa phần tử đầu
LRANGE queue:emails 0 -1   # xem toàn bộ list

# Sorted Set — leaderboard
ZADD leaderboard 1500 "player:1"
ZADD leaderboard 2300 "player:2"
ZADD leaderboard 1800 "player:3"
ZREVRANGE leaderboard 0 2 WITHSCORES   # top 3 điểm cao nhất

# Pub/Sub — message broker
SUBSCRIBE channel:notifications
PUBLISH channel:notifications "Bạn có tin nhắn mới"
```

**So sánh MongoDB và Redis:**

| Tiêu chí | MongoDB | Redis |
|----------|---------|-------|
| Loại | Document store | Key-value / In-memory |
| Lưu trữ | Disk (persistent) | RAM (có thể persist) |
| Tốc độ | Nhanh | Cực nhanh (< 1ms) |
| Dữ liệu phức tạp | Rất tốt | Hạn chế |
| Dung lượng | Lớn (TB+) | Bị giới hạn bởi RAM |
| Dùng cho | CRUD, analytics | Cache, session, queue |

---

## 3. Tools

### 3.1 MongoDB Compass

**MongoDB Compass** là công cụ GUI chính thức của MongoDB, giúp quản lý, truy vấn và phân tích dữ liệu trực quan mà không cần dùng command line.

**Tính năng chính:**

| Tính năng | Mô tả |
|-----------|-------|
| Visual Query Builder | Xây dựng filter trực quan bằng form |
| Schema Analyzer | Phân tích cấu trúc dữ liệu tự động |
| Aggregation Pipeline | Thiết kế pipeline bằng giao diện kéo thả |
| Index Management | Xem, tạo và xóa index |
| Performance Advisor | Gợi ý tối ưu truy vấn chậm |
| Import/Export | Nhập xuất dữ liệu JSON, CSV |

**Kết nối MongoDB Compass:**

```
Connection String:
mongodb://localhost:27017

Hoặc MongoDB Atlas:
mongodb+srv://username:password@cluster.mongodb.net/
```

### 3.2 RedisInsight

**RedisInsight** là công cụ GUI chính thức của Redis (do Redis Ltd phát triển), hỗ trợ quản lý và debug dữ liệu Redis một cách trực quan.

**Tính năng chính:**

| Tính năng | Mô tả |
|-----------|-------|
| Key Browser | Duyệt, tìm kiếm và lọc keys |
| CLI tích hợp | Chạy lệnh Redis ngay trong giao diện |
| Memory Analyzer | Phân tích dung lượng bộ nhớ từng key |
| Slow Log | Xem các lệnh chạy chậm |
| Pub/Sub Monitor | Theo dõi kênh pub/sub real-time |
| Stream Viewer | Xem và quản lý Redis Streams |

**Kết nối RedisInsight:**

```
Host:     localhost
Port:     6379
Password: (nếu có)

Hoặc Redis Cloud:
redis-xxxxx.c1.us-east-1-2.ec2.cloud.redislabs.com:12345
```

---

## 4. Thực hành và ví dụ
Thực hành Insight Redis
![alt text](photo/image-2.png)