#    Go (Golang)

---

## 1. Giới Thiệu

**Go** (Golang) là ngôn ngữ lập trình biên dịch được phát triển bởi Google (2009), nổi tiếng với hiệu năng cao, concurrency đơn giản, thời gian build nhanh và binary độc lập không cần runtime.

| Tiêu chí | Mô tả |
|----------|-------|
| Typing | Static, strong |
| Paradigm | Compiled, concurrent |
| Garbage Collection | Có (low-latency) |
| Binary | Độc lập, không cần runtime |
| Phù hợp | Microservice, CLI, high-performance API |

---

## 2. Key Concepts

### 2.1 Concurrency — Goroutines

**Goroutine** là luồng nhẹ (lightweight thread) do Go runtime quản lý, chi phí tạo rất thấp (~2KB RAM). Có thể chạy hàng triệu goroutine đồng thời mà không tốn nhiều tài nguyên.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func fetchData(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    time.Sleep(100 * time.Millisecond) // giả lập I/O
    fmt.Printf("Fetched data for user %d\n", id)
}

func main() {
    var wg sync.WaitGroup

    // Chạy 1000 goroutine đồng thời
    for i := 1; i <= 1000; i++ {
        wg.Add(1)
        go fetchData(i, &wg) // từ khóa "go" tạo goroutine
    }

    wg.Wait()
    fmt.Println("Tất cả goroutine hoàn thành")
}
```

**So sánh Thread vs Goroutine:**

| Tiêu chí | OS Thread | Goroutine |
|----------|-----------|-----------|
| Bộ nhớ khởi tạo | ~1MB | ~2KB |
| Quản lý bởi | OS | Go runtime |
| Số lượng thực tế | Hàng trăm | Hàng triệu |
| Context switch | Chậm | Nhanh |

### 2.2 Concurrency — Channels

**Channel** là cơ chế giao tiếp an toàn giữa các goroutine, tránh race condition theo triết lý:

> *"Do not communicate by sharing memory; instead, share memory by communicating."*

```go
// Unbuffered channel — gửi và nhận phải đồng bộ
ch := make(chan int)

// Buffered channel — gửi không cần nhận ngay
ch := make(chan int, 5)

// Producer - Consumer pattern
func producer(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i // Gửi dữ liệu vào channel
    }
    close(ch)
}

func consumer(ch <-chan int) {
    for val := range ch {
        fmt.Printf("Nhận: %d\n", val)
    }
}

func main() {
    ch := make(chan int, 5)
    go producer(ch)
    consumer(ch)
}
```

**Select — xử lý nhiều channel:**

```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() { ch1 <- "từ channel 1" }()
    go func() { ch2 <- "từ channel 2" }()

    select {
    case msg := <-ch1:
        fmt.Println(msg)
    case msg := <-ch2:
        fmt.Println(msg)
    case <-time.After(1 * time.Second):
        fmt.Println("Timeout!")
    }
}
```

### 2.3 Pointers

Go sử dụng pointer để truyền tham chiếu, tránh copy dữ liệu lớn và cho phép thay đổi giá trị gốc.

```go
type User struct {
    ID    int
    Name  string
    Email string
}

// Truyền theo giá trị — không thay đổi bên ngoài
func printUser(u User) {
    fmt.Println(u.Name)
}

// Truyền theo pointer — thay đổi ảnh hưởng bên ngoài
func updateEmail(u *User, email string) {
    u.Email = email // thay đổi giá trị gốc
}

func main() {
    user := User{ID: 1, Name: "Nguyễn Văn A", Email: "old@example.com"}

    updateEmail(&user, "new@example.com") // truyền địa chỉ
    fmt.Println(user.Email)               // new@example.com

    ptr := &user           // lấy địa chỉ
    fmt.Println(ptr.Name)  // Go tự dereference
}
```

### 2.4 Interfaces

Interface trong Go được implement **ngầm định** (implicit) — không cần khai báo `implements`, chỉ cần có đủ các method.

```go
// Định nghĩa interface
type Repository interface {
    FindByID(id int) (*User, error)
    Save(user *User) error
    Delete(id int) error
}

// PostgreSQL implementation
type PostgresRepo struct{ db *sql.DB }

func (r *PostgresRepo) FindByID(id int) (*User, error) {
    var user User
    err := r.db.QueryRow("SELECT * FROM users WHERE id=$1", id).Scan(&user)
    return &user, err
}
func (r *PostgresRepo) Save(user *User) error   { /* ... */ return nil }
func (r *PostgresRepo) Delete(id int) error      { /* ... */ return nil }

// MongoDB implementation — dễ dàng swap
type MongoRepo struct{ col *mongo.Collection }

func (r *MongoRepo) FindByID(id int) (*User, error) { /* ... */ return nil, nil }
func (r *MongoRepo) Save(user *User) error           { /* ... */ return nil }
func (r *MongoRepo) Delete(id int) error             { /* ... */ return nil }

// Service không quan tâm implementation cụ thể
type UserService struct {
    repo Repository // nhận bất kỳ impl nào
}

func NewUserService(repo Repository) *UserService {
    return &UserService{repo: repo}
}
```

### 2.5 Error Handling

Go không có exception — lỗi được trả về như giá trị thông thường, buộc lập trình viên xử lý tường minh.

```go
// Hàm trả về (result, error)
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("không thể chia cho 0")
    }
    return a / b, nil
}

// Custom error type
type NotFoundError struct {
    Resource string
    ID       int
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %d not found", e.Resource, e.ID)
}

func findUser(id int) (*User, error) {
    user, ok := db[id]
    if !ok {
        return nil, &NotFoundError{Resource: "User", ID: id}
    }
    return user, nil
}

// Xử lý error
func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Lỗi:", err)
        return
    }
    fmt.Println("Kết quả:", result)

    // Kiểm tra loại error cụ thể
    user, err := findUser(99)
    var notFound *NotFoundError
    if errors.As(err, &notFound) {
        fmt.Println("Không tìm thấy:", notFound.Resource)
    }
}
```

---

## 3. Frameworks

### 3.1 Gin

Gin là HTTP framework nhanh nhất cho Go, hiệu năng cao và API rõ ràng — lựa chọn phổ biến nhất cho REST API.

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default() // bao gồm Logger và Recovery middleware

    // Route groups
    api := r.Group("/api/v1")
    {
        users := api.Group("/users")
        users.GET("", getUsers)
        users.GET("/:id", getUserByID)
        users.POST("", createUser)
        users.PUT("/:id", updateUser)
        users.DELETE("/:id", deleteUser)
    }

    r.Run(":8080")
}

func getUsers(c *gin.Context) {
    // Query params: /users?page=1&limit=10
    page := c.DefaultQuery("page", "1")
    limit := c.DefaultQuery("limit", "10")

    users := []gin.H{
        {"id": 1, "name": "Nguyễn Văn A"},
    }
    c.JSON(http.StatusOK, gin.H{
        "data": users,
        "page": page,
        "limit": limit,
    })
}

func getUserByID(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{"id": id})
}

type CreateUserInput struct {
    Name  string `json:"name" binding:"required"`
    Email string `json:"email" binding:"required,email"`
}

func createUser(c *gin.Context) {
    var input CreateUserInput
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusCreated, gin.H{"message": "Tạo user thành công"})
}
```

### 3.2 Echo

Echo là framework hiệu năng cao với middleware phong phú, API tương tự Express.js — dễ tiếp cận với developer từ Node.js.

```go
import (
    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
)

func main() {
    e := echo.New()

    // Middleware
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())
    e.Use(middleware.CORS())
    e.Use(middleware.RateLimiter(middleware.NewRateLimiterMemoryStore(20)))

    // JWT protected routes
    api := e.Group("/api")
    api.Use(middleware.JWT([]byte("secret")))
    api.GET("/users", getUsers)
    api.POST("/users", createUser)

    e.Logger.Fatal(e.Start(":8080"))
}

func getUsers(c echo.Context) error {
    return c.JSON(200, map[string]string{"message": "ok"})
}
```

### 3.3 Fx

Fx là framework **Dependency Injection** cho Go do Uber phát triển, phù hợp ứng dụng lớn cần quản lý dependency phức tạp.

```go
import "go.uber.org/fx"

// Các provider
func NewDatabase() *sql.DB {
    db, _ := sql.Open("postgres", "...")
    return db
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func NewUserService(repo *UserRepository) *UserService {
    return &UserService{repo: repo}
}

func NewHTTPServer(service *UserService) *http.Server {
    // setup router với service
    return &http.Server{Addr: ":8080"}
}

func main() {
    fx.New(
        fx.Provide(
            NewDatabase,
            NewUserRepository,
            NewUserService,
            NewHTTPServer,
        ),
        fx.Invoke(func(server *http.Server) {
            go server.ListenAndServe()
        }),
    ).Run()
}
```

**So sánh 3 frameworks Go:**

| Tiêu chí | Gin | Echo | Fx |
|----------|-----|------|----|
| Mục đích | HTTP framework | HTTP framework | Dependency Injection |
| Hiệu năng | Rất cao | Rất cao | N/A |
| Learning curve | Thấp | Thấp | Trung bình |
| Middleware | Phong phú | Phong phú | N/A |
| Phù hợp | API nhanh, đơn giản | API, WebSocket | Ứng dụng lớn, phức tạp |

---

## 4. Projects

### 4.1 High-performance API

Go phù hợp xây dựng API cần xử lý lượng lớn request với độ trễ thấp, thường dùng Gin hoặc Echo.

```go
func setupRouter() *gin.Engine {
    gin.SetMode(gin.ReleaseMode)
    r := gin.New()
    r.Use(gin.Recovery())

    // Connection pool cho database
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(25)
    db.SetConnMaxLifetime(5 * time.Minute)

    v1 := r.Group("/api/v1")
    v1.GET("/products", getProducts)
    v1.GET("/products/:id", getProductByID)

    return r
}
```

### 4.2 Microservice

Go là lựa chọn hàng đầu cho microservice nhờ binary nhỏ (~10MB), khởi động nhanh (<100ms) và tiêu tốn ít RAM.

```
Kiến trúc microservice với Go:

API Gateway (Go/Gin)
    ├── User Service    (Go + PostgreSQL)   :8001
    ├── Order Service   (Go + PostgreSQL)   :8002
    ├── Product Service (Go + MongoDB)      :8003
    └── Notify Service  (Go + Redis)        :8004
```

```go
// Giao tiếp giữa services qua HTTP
func (s *OrderService) createOrder(userID int, items []Item) error {
    // Gọi User Service để verify user
    resp, err := http.Get(fmt.Sprintf("http://user-service:8001/users/%d", userID))
    if err != nil || resp.StatusCode != 200 {
        return errors.New("user không hợp lệ")
    }
    // Xử lý tạo order...
    return nil
}
```

### 4.3 Command-line Tool

Go tạo ra binary độc lập, không cần cài runtime — lý tưởng cho CLI tools phân phối dễ dàng.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
)

type WeatherAPI struct {
	Weather []struct {
		ID          int    `json:"id"`
		Main        string `json:"main"`
		Description string `json:"description"`
		Icon        string `json:"icon"`
	} `json:"weather"`
	Main struct {
		Temp      float64 `json:"temp"`
		FeelsLike float64 `json:"feels_like"`
	} `json:"main"`
	Sys struct {
		Country string `json:"country"`
	} `json:"sys"`
	Name string `json:"name"`
}

func main() {
	res, err := http.Get("https://api.openweathermap.org/data/2.5/weather?q=HaNoi&units=metric&appid=535c22f660a122b762427eab60cd221c")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer res.Body.Close()

	if res.StatusCode != http.StatusOK {
		fmt.Println("Error: Weather API error:", res.Status)
		return
	}

	body, err := io.ReadAll(res.Body)
	if err != nil {
		fmt.Println("Error reading response body:", err)
		return
	}
	

	var weather WeatherAPI
	err = json.Unmarshal(body, &weather)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Weather in %s, %s:\n", weather.Name, weather.Sys.Country)
	fmt.Printf("Temperature: %.2f°C (feels like %.2f°C)\n", weather.Main.Temp, weather.Main.FeelsLike)
	for _, w := range weather.Weather {
		fmt.Printf("- %s: %s (%s)\n", w.Main, w.Description, w.Icon)
	}
}

```

```bash
# Sử dụng CLI
go run main.go
```

---

## 5. Tổng Kết

| Nhóm | Nội dung |
|------|----------|
| **Key Concepts** | Goroutines, Channels, Pointers, Interfaces, Error Handling |
| **Frameworks** | Gin (HTTP nhanh), Echo (HTTP linh hoạt), Fx (DI) |
| **Projects** | High-performance API, Microservice, CLI Tool |


