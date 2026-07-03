# Báo Cáo: gRPC

---

## 1. Giới Thiệu

**gRPC** (Google Remote Procedure Call) là framework giao tiếp giữa các service hiệu năng cao, được phát triển bởi Google. Thay vì gửi JSON qua HTTP như REST, gRPC dùng **Protocol Buffers** để serialize dữ liệu và truyền qua **HTTP/2**.

**So sánh gRPC và REST:**

| Tiêu chí | REST | gRPC |
|----------|------|------|
| Giao thức | HTTP/1.1 | HTTP/2 |
| Định dạng dữ liệu | JSON (text) | Protocol Buffers (binary) |
| Tốc độ | Trung bình | Nhanh hơn ~7-10x |
| Streaming | Hạn chế | Hỗ trợ đầy đủ |
| Contract | Không bắt buộc | Bắt buộc (`.proto` file) |
| Debug | Dễ  | Khó hơn (binary) |
| Phù hợp | Public API, web | Internal microservice |

---

## 2. Concepts

### 2.1 Protocol Buffers (Protobuf)

**Protocol Buffers** là ngôn ngữ định nghĩa dữ liệu (IDL) và cơ chế serialize nhị phân của Google. Dữ liệu được định nghĩa trong file `.proto`, sau đó compile ra code cho nhiều ngôn ngữ.

```protobuf
// user.proto
syntax = "proto3";

package user;
option go_package = "goshop/proto/user";

// Định nghĩa message (giống struct)
message User {
  uint32 id     = 1;
  string name   = 2;
  string email  = 3;
  string role   = 4;
}

message GetUserRequest {
  uint32 id = 1;
}

message CreateUserRequest {
  string name     = 1;
  string email    = 2;
  string password = 3;
}

message UserResponse {
  bool   success = 1;
  string message = 2;
  User   user    = 3;
}

message UserListResponse {
  repeated User users = 1;  // repeated = array
  int32 total         = 2;
}
```

**Ưu điểm so với JSON:**

| Tiêu chí | JSON | Protobuf |
|----------|------|----------|
| Kích thước | Lớn (text) | Nhỏ hơn ~3-5x (binary) |
| Parse speed | Chậm hơn | Nhanh hơn ~5x |
| Type safety | Không | Có (compile-time check) |
| Human readable | Có | Không |

---

### 2.2 Service Definitions

**Service Definition** là nơi định nghĩa các RPC method trong file `.proto` — giống như định nghĩa interface của API.

```protobuf
// Định nghĩa service trong .proto file
service UserService {
  // Unary RPC
  rpc GetUser(GetUserRequest) returns (UserResponse);
  rpc CreateUser(CreateUserRequest) returns (UserResponse);
  rpc DeleteUser(GetUserRequest) returns (UserResponse);

  // Server Streaming
  rpc ListUsers(Empty) returns (stream User);

  // Client Streaming
  rpc BulkCreateUsers(stream CreateUserRequest) returns (UserListResponse);

  // Bidirectional Streaming
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message Empty {}
message ChatMessage {
  string sender  = 1;
  string content = 2;
}
```

Sau khi có file `.proto`, dùng `protoc` compiler để generate code Go:

```bash
protoc --go_out=. --go-grpc_out=. proto/user.proto
```

Generated code sẽ tạo ra:
- `user.pb.go` — structs cho các message
- `user_grpc.pb.go` — interface cho server và client

---

### 2.3 Unary RPC

**Unary** là kiểu đơn giản nhất — client gửi 1 request, server trả về 1 response. Giống REST.

```
Client ──── request ────► Server
Client ◄─── response ─── Server
```

```go
// Server implementation
type UserServer struct {
    proto.UnimplementedUserServiceServer
    db *gorm.DB
}

func (s *UserServer) GetUser(ctx context.Context, req *proto.GetUserRequest) (*proto.UserResponse, error) {
    var user models.User
    if err := s.db.First(&user, req.Id).Error; err != nil {
        return nil, status.Errorf(codes.NotFound, "user %d not found", req.Id)
    }

    return &proto.UserResponse{
        Success: true,
        User: &proto.User{
            Id:    uint32(user.ID),
            Name:  user.Name,
            Email: user.Email,
        },
    }, nil
}

// Client call
func getUser(client proto.UserServiceClient, id uint32) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    resp, err := client.GetUser(ctx, &proto.GetUserRequest{Id: id})
    if err != nil {
        log.Printf("GetUser error: %v", err)
        return
    }
    fmt.Printf("User: %s (%s)\n", resp.User.Name, resp.User.Email)
}
```

---

### 2.4 Server Streaming

**Server Streaming** — client gửi 1 request, server trả về nhiều response liên tục theo thời gian.

```
Client ──── request ────► Server
Client ◄─── response 1 ── Server
Client ◄─── response 2 ── Server
Client ◄─── response 3 ── Server
            ...
```

**Use case:** Lấy danh sách lớn, live feed, export data

```go
// Server
func (s *UserServer) ListUsers(req *proto.Empty, stream proto.UserService_ListUsersServer) error {
    var users []models.User
    s.db.FindInBatches(&users, 100, func(tx *gorm.DB, batch int) error {
        for _, user := range users {
            if err := stream.Send(&proto.User{
                Id:    uint32(user.ID),
                Name:  user.Name,
                Email: user.Email,
            }); err != nil {
                return err
            }
        }
        return nil
    })
    return nil
}

// Client
func listUsers(client proto.UserServiceClient) {
    stream, err := client.ListUsers(context.Background(), &proto.Empty{})
    if err != nil {
        log.Fatal(err)
    }

    for {
        user, err := stream.Recv()
        if err == io.EOF {
            break // Server đã gửi xong
        }
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("Received: %s\n", user.Name)
    }
}
```

---

### 2.5 Client Streaming

**Client Streaming** — client gửi nhiều request liên tục, server trả về 1 response sau khi nhận xong.

```
Client ──── request 1 ───► Server
Client ──── request 2 ───► Server
Client ──── request 3 ───► Server
Client ◄─── response ──── Server
```

**Use case:** Upload file lớn, bulk insert dữ liệu

```go
// Server
func (s *UserServer) BulkCreateUsers(stream proto.UserService_BulkCreateUsersServer) error {
    var count int32
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            // Client gửi xong, trả về kết quả
            return stream.SendAndClose(&proto.UserListResponse{
                Total: count,
            })
        }
        if err != nil {
            return err
        }

        // Tạo user trong DB
        s.db.Create(&models.User{Name: req.Name, Email: req.Email})
        count++
    }
}

// Client
func bulkCreate(client proto.UserServiceClient, users []models.User) {
    stream, _ := client.BulkCreateUsers(context.Background())

    for _, user := range users {
        stream.Send(&proto.CreateUserRequest{
            Name:  user.Name,
            Email: user.Email,
        })
    }

    resp, _ := stream.CloseAndRecv()
    fmt.Printf("Created %d users\n", resp.Total)
}
```

---

### 2.6 Bidirectional Streaming

**Bidirectional Streaming** — client và server đều có thể gửi nhiều message cho nhau đồng thời.

```
Client ──── message ─────► Server
Client ◄─── message ────── Server
Client ──── message ─────► Server
Client ◄─── message ────── Server
            ...
```

**Use case:** Chat realtime, collaborative editing, game

```go
// Server
func (s *ChatServer) Chat(stream proto.UserService_ChatServer) error {
    for {
        msg, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }

        // Nhận message từ client rồi gửi lại (echo) hoặc broadcast
        response := &proto.ChatMessage{
            Sender:  "Server",
            Content: fmt.Sprintf("Echo: %s", msg.Content),
        }
        if err := stream.Send(response); err != nil {
            return err
        }
    }
}
```

**Tóm tắt 4 loại RPC:**

| Loại | Request | Response | Use case |
|------|---------|----------|---------|
| Unary | 1 | 1 | CRUD thông thường |
| Server Streaming | 1 | Nhiều | Live feed, export |
| Client Streaming | Nhiều | 1 | Upload, bulk insert |
| Bidirectional | Nhiều | Nhiều | Chat, realtime |

---

## 3. Tools

### 3.1 protoc Compiler

**protoc** là compiler chính thức để compile file `.proto` thành code của ngôn ngữ tương ứng.

**Cài đặt:**

```bash
# macOS
brew install protobuf

# Ubuntu
apt install -y protobuf-compiler

# Windows
choco install protoc
```

**Cài Go plugins:**

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

**Compile .proto file:**

```bash
# Compile 1 file
protoc --go_out=. --go-grpc_out=. proto/user.proto

# Compile toàn bộ thư mục proto/
protoc --go_out=. --go-grpc_out=. proto/*.proto

# Với import path tùy chỉnh
protoc \
  --go_out=. \
  --go_opt=paths=source_relative \
  --go-grpc_out=. \
  --go-grpc_opt=paths=source_relative \
  proto/user.proto
```

**Cấu trúc thư mục điển hình:**

```
project/
├── proto/
│   ├── user.proto          ← file định nghĩa
│   └── order.proto
├── pb/                     ← generated code (không sửa tay)
│   ├── user.pb.go
│   ├── user_grpc.pb.go
│   ├── order.pb.go
│   └── order_grpc.pb.go
├── server/
│   └── main.go
└── client/
    └── main.go
```

---

### 3.2 grpcurl

**grpcurl** là công cụ command-line để test gRPC API — tương đương cURL nhưng dành cho gRPC.

**Cài đặt:**

```bash
# macOS
brew install grpcurl

# Go
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

**Các lệnh hay dùng:**

```bash
# List tất cả services (server phải bật reflection)
grpcurl -plaintext localhost:50051 list

# List methods của 1 service
grpcurl -plaintext localhost:50051 list user.UserService

# Mô tả message type
grpcurl -plaintext localhost:50051 describe user.User

# Gọi Unary RPC
grpcurl -plaintext \
  -d '{"id": 1}' \
  localhost:50051 \
  user.UserService/GetUser

# Gọi với metadata (tương đương HTTP header)
grpcurl -plaintext \
  -H "authorization: Bearer <token>" \
  -d '{"id": 1}' \
  localhost:50051 \
  user.UserService/GetUser

# Server Streaming
grpcurl -plaintext \
  localhost:50051 \
  user.UserService/ListUsers
```

**Bật reflection trong Go server (để grpcurl list được service):**

```go
import "google.golang.org/grpc/reflection"

s := grpc.NewServer()
proto.RegisterUserServiceServer(s, &UserServer{})
reflection.Register(s) // ← thêm dòng này
```

---

## 4. Projects




**Docker Compose cho microservices gRPC:**

```yaml
services:
  user-service:
    build: ./user-service
    ports:
      - "50051:50051"

  product-service:
    build: ./product-service
    ports:
      - "50053:50053"

  order-service:
    build: ./order-service
    ports:
      - "50052:50052"
    environment:
      - USER_SERVICE_ADDR=user-service:50051
      - PRODUCT_SERVICE_ADDR=product-service:50053
    depends_on:
      - user-service
      - product-service

  api-gateway:
    build: ./api-gateway      # REST → gRPC
    ports:
      - "8080:8080"
    depends_on:
      - order-service
```

---

## 5. Tổng Kết

| Nhóm | Nội dung |
|------|----------|
| **Concepts** | Protocol Buffers, Service Definitions, Unary, Server/Client Streaming, Bidirectional Streaming |
| **Tools** | protoc (compile .proto), grpcurl (test API) |
| **Projects** | Inter-service communication trong microservices |

