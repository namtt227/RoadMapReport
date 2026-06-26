# Node.js (JavaScript / TypeScript)

---

## 1. Giới Thiệu

**Node.js** là runtime cho phép chạy JavaScript phía server, xây dựng trên V8 engine của Chrome. **TypeScript** là superset của JavaScript bổ sung kiểu tĩnh (static typing), giúp code dễ bảo trì và ít lỗi hơn trong dự án lớn.

| Tiêu chí | JavaScript | TypeScript |
|----------|------------|------------|
| Typing | Dynamic | Static |
| Compile | Không cần | Cần transpile |
| IDE support | Tốt | Rất tốt |
| Phù hợp | Prototype, script nhỏ | Dự án lớn, team |

---

## 2. Key Concepts

### 2.1 Asynchronous Programming

Node.js hoạt động theo mô hình **non-blocking I/O** — không chờ một tác vụ hoàn thành mới xử lý tác vụ tiếp theo, giúp xử lý hàng nghìn request đồng thời với tài nguyên ít.

```
Blocking (truyền thống):        Non-blocking (Node.js):
Request 1 → Xử lý → Trả về     Request 1 ──┐
Request 2 → Xử lý → Trả về     Request 2 ──┤ → Xử lý song song
Request 3 → Xử lý → Trả về     Request 3 ──┘
```

### 2.2 Event Loop

Event Loop là cơ chế cốt lõi của Node.js, liên tục kiểm tra và xử lý các tác vụ trong hàng đợi.

```
   ┌─────────────────────────┐
   │        Call Stack       │  ← Thực thi code đồng bộ
   └────────────┬────────────┘
                │
   ┌────────────▼────────────┐
   │       Event Loop        │  ← Kiểm tra hàng đợi liên tục
   └────────────┬────────────┘
                │
   ┌────────────▼────────────┐
   │    Callback Queue       │  ← setTimeout, I/O callbacks
   └─────────────────────────┘
```

**Thứ tự ưu tiên trong Event Loop:**

1. `process.nextTick()` — ưu tiên cao nhất
2. `Promise.then()` — microtask queue
3. `setTimeout` / `setInterval` — macrotask queue
4. I/O callbacks
5. `setImmediate`

### 2.3 Callbacks

Cách xử lý bất đồng bộ truyền thống — truyền hàm vào làm tham số, gọi lại khi tác vụ hoàn thành.

```javascript
const fs = require('fs');

// Callback
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Lỗi:', err);
    return;
  }
  console.log(data);
});
```

> **Callback Hell** — lồng nhiều callback gây code khó đọc, khó bảo trì:

```javascript
// Callback hell 
getUser(id, (err, user) => {
  getOrders(user.id, (err, orders) => {
    getProducts(orders[0].id, (err, product) => {
      // lồng mãi...
    });
  });
});
```

### 2.4 Promises

Promises giải quyết callback hell bằng cách chain các tác vụ bất đồng bộ.

```javascript
// Promise
fetchUser(userId)
  .then(user => fetchOrders(user.id))
  .then(orders => processOrders(orders))
  .catch(err => console.error('Lỗi:', err))
  .finally(() => console.log('Hoàn thành'));

// Promise.all — chạy song song
Promise.all([
  fetchUser(1),
  fetchOrders(1),
  fetchProducts()
]).then(([user, orders, products]) => {
  console.log({ user, orders, products });
});
```

### 2.5 Async / Await

Async/Await là cú pháp hiện đại nhất, giúp viết code bất đồng bộ trông như đồng bộ.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Async/Await với TypeScript
async function getUserWithOrders(userId: number): Promise<void> {
  try {
    const user: User = await fetchUser(userId);
    const orders = await fetchOrders(user.id);
    const result = await processOrders(orders);
    console.log('Kết quả:', result);
  } catch (err) {
    console.error('Lỗi:', err);
  }
}

// Chạy song song với async/await
async function fetchAll(userId: number) {
  const [user, products] = await Promise.all([
    fetchUser(userId),
    fetchProducts()
  ]);
  return { user, products };
}
```

**So sánh 3 cách xử lý bất đồng bộ:**

| Cách | Ưu điểm | Nhược điểm |
|------|---------|-----------|
| Callback | Đơn giản, không cần polyfill | Callback hell, khó debug |
| Promise | Chain rõ ràng, xử lý lỗi tốt | Vẫn có thể phức tạp |
| Async/Await | Dễ đọc nhất, debug dễ | Cần transpile cho môi trường cũ |

---

## 3. Frameworks

### 3.1 Express.js

Express là framework Node.js nhẹ và linh hoạt nhất, không áp đặt cấu trúc — phù hợp với dự án nhỏ hoặc khi cần toàn quyền kiểm soát.

VD:
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Middleware xác thực
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  next();
};

// Routes
app.get('/api/users', authMiddleware, async (req, res) => {
  const users = await db.query('SELECT * FROM users');
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const { name, email } = req.body;
  const user = await db.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email]
  );
  res.status(201).json(user.rows[0]);
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Cấu trúc project Express điển hình:**

```
src/
├── routes/
│   ├── users.js
│   └── orders.js
├── middlewares/
│   ├── auth.js
│   └── errorHandler.js
├── controllers/
│   └── userController.js
├── services/
│   └── userService.js
└── app.js
```

### 3.2 NestJS

NestJS là framework TypeScript có kiến trúc rõ ràng (Module → Controller → Service), lấy cảm hứng từ Angular — phù hợp dự án lớn, cần cấu trúc chặt chẽ.

VD:

```typescript
// user.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

```typescript
// users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  async findOne(id: number): Promise<User> {
    const user = await this.usersRepository.findOneBy({ id });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }
}
```

**So sánh Express.js và NestJS:**

| Tiêu chí | Express.js | NestJS |
|----------|------------|--------|
| Cấu trúc | Tự do | Module / Controller / Service |
| TypeScript | Tùy chọn | Mặc định |
| Learning curve | Thấp | Trung bình |
| Dependency Injection | Không có sẵn | Tích hợp sẵn |
| Testing | Tự cấu hình | Tích hợp sẵn |
| Phù hợp | API nhỏ, prototype | Dự án lớn, enterprise |

---

## 4. Tổng Kết

| Nhóm | Nội dung |
|------|----------|
| **Key Concepts** | Async Programming, Event Loop, Callbacks, Promises, Async/Await |
| **Frameworks** | Express.js (linh hoạt), NestJS (có cấu trúc) |


