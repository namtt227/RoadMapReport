#   Message Queue

---

## 1. Giới Thiệu

**Message Queue** (Hàng đợi tin nhắn) là cơ chế giao tiếp bất đồng bộ giữa các service, cho phép một service gửi tin nhắn mà không cần chờ service kia xử lý xong ngay lập tức.

**Tại sao cần Message Queue:**

```
Không có Message Queue:          Có Message Queue:
Service A → gọi thẳng → B       Service A → Queue → Service B
            ↓                                ↓
     A phải chờ B xong          A gửi xong là tiếp tục việc khác
     B chết → A bị lỗi theo     B chết → message vẫn còn trong queue
```

**Các lợi ích chính:**

| Lợi ích | Mô tả |
|---------|-------|
| **Decoupling** | Các service độc lập, không phụ thuộc trực tiếp |
| **Async** | Producer không cần chờ Consumer xử lý |
| **Resilience** | Service chết thì message vẫn không mất |
| **Scalability** | Thêm Consumer để xử lý nhanh hơn |
| **Load Leveling** | Cân bằng tải khi traffic đột biến |

---

## 2. Kafka

**Apache Kafka** là nền tảng streaming phân tán hiệu năng cao, được thiết kế để xử lý hàng triệu message mỗi giây. Ban đầu được phát triển bởi LinkedIn.

### 2.1 Các Khái Niệm Cơ Bản

```
Producer → [Topic: orders] → Consumer Group
                │
         ┌──────┴──────┐
      Partition 0   Partition 1   ← song song, tăng throughput
```

| Khái niệm | Mô tả |
|-----------|-------|
| **Producer** | Service gửi message vào Kafka |
| **Consumer** | Service đọc và xử lý message |
| **Topic** | Kênh phân loại message (giống tên hàng đợi) |
| **Partition** | Phân vùng của topic, cho phép xử lý song song |
| **Broker** | Server Kafka lưu trữ message |
| **Consumer Group** | Nhóm consumer cùng xử lý một topic |
| **Offset** | Vị trí của message trong partition |

### 2.2 Đặc Điểm Nổi Bật

| Tính năng | Mô tả |
|-----------|-------|
| **Throughput cực cao** | Hàng triệu message/giây |
| **Lưu trữ lâu dài** | Message được giữ theo thời gian cấu hình (ngày, tuần...) |
| **Replay** | Consumer có thể đọc lại message cũ |
| **Distributed** | Phân tán trên nhiều broker |
| **Ordering** | Đảm bảo thứ tự trong cùng partition |

### 2.3 Ví Dụ Thực Tế

```javascript
// Producer — gửi message (Node.js với kafkajs)
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

async function sendOrderEvent(order) {
  await producer.connect();
  await producer.send({
    topic: 'orders',
    messages: [
      {
        key: String(order.id),       // cùng key → cùng partition → đảm bảo thứ tự
        value: JSON.stringify(order),
      }
    ]
  });
  console.log('Đã gửi order event:', order.id);
}
```

```javascript
// Consumer — nhận và xử lý message
const consumer = kafka.consumer({ groupId: 'order-processing-group' });

async function startConsumer() {
  await consumer.connect();
  await consumer.subscribe({ topic: 'orders', fromBeginning: false });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const order = JSON.parse(message.value.toString());
      console.log(`Xử lý order #${order.id} từ partition ${partition}`);
      await processOrder(order);
    }
  });
}
```

```yaml
# Docker Compose cho Kafka
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
    depends_on:
      - zookeeper

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8090:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    depends_on:
      - kafka
```

### 2.4 Tool: Kafka UI

**Kafka UI** (provectuslabs/kafka-ui) là giao diện web miễn phí để quản lý và monitor Kafka.

**Tính năng:**

| Tính năng | Mô tả |
|-----------|-------|
| Topic Manager | Xem, tạo, xóa topic |
| Message Browser | Đọc message trong topic theo offset |
| Consumer Groups | Theo dõi lag của consumer group |
| Producer | Gửi message thử trực tiếp từ UI |
| Broker Info | Xem trạng thái các broker |

Truy cập tại: `http://localhost:8090`

---

## 3. RabbitMQ

**RabbitMQ** là message broker truyền thống, triển khai chuẩn **AMQP** (Advanced Message Queuing Protocol), dễ dùng và phù hợp với hầu hết use case thông thường.

### 3.1 Các Khái Niệm Cơ Bản

```
Producer → Exchange → Queue → Consumer
              │
        (định tuyến message
         theo routing rules)
```

| Khái niệm | Mô tả |
|-----------|-------|
| **Producer** | Gửi message vào Exchange |
| **Exchange** | Nhận message và định tuyến vào Queue |
| **Queue** | Hàng đợi lưu message chờ xử lý |
| **Consumer** | Nhận và xử lý message từ Queue |
| **Binding** | Quy tắc kết nối Exchange với Queue |
| **Routing Key** | Khóa định tuyến message đến đúng queue |

**4 loại Exchange:**

| Loại | Cách định tuyến | Dùng cho |
|------|----------------|---------|
| `direct` | Routing key khớp chính xác | Task queue |
| `fanout` | Gửi đến tất cả queue | Broadcast, notifications |
| `topic` | Routing key theo pattern (`*.error`, `order.#`) | Log theo category |
| `headers` | Theo header của message | Phức tạp, ít dùng |

### 3.2 Đặc Điểm Nổi Bật

| Tính năng | Mô tả |
|-----------|-------|
| **Dễ setup** | Đơn giản hơn Kafka nhiều |
| **Flexible routing** | Exchange với nhiều kiểu định tuyến |
| **Acknowledgment** | Consumer xác nhận đã xử lý xong |
| **Dead Letter Queue** | Message lỗi được chuyển sang queue riêng |
| **Management UI** | Có sẵn giao diện web tích hợp |

### 3.3 Ví Dụ Thực Tế

```javascript
// Producer — gửi message (Node.js với amqplib)
const amqp = require('amqplib');

async function sendEmailNotification(email) {
  const conn = await amqp.connect('amqp://localhost');
  const channel = await conn.createChannel();

  const queue = 'email_notifications';
  await channel.assertQueue(queue, { durable: true }); // queue tồn tại sau khi restart

  channel.sendToQueue(
    queue,
    Buffer.from(JSON.stringify(email)),
    { persistent: true }  // message không mất khi RabbitMQ restart
  );

  console.log('Đã gửi email notification:', email.to);
  await conn.close();
}
```

```javascript
// Consumer — nhận và xử lý message
async function startEmailWorker() {
  const conn = await amqp.connect('amqp://localhost');
  const channel = await conn.createChannel();

  const queue = 'email_notifications';
  await channel.assertQueue(queue, { durable: true });

  channel.prefetch(1); // mỗi lần chỉ nhận 1 message, xử lý xong mới nhận tiếp

  channel.consume(queue, async (msg) => {
    const email = JSON.parse(msg.content.toString());
    try {
      await sendEmail(email);
      channel.ack(msg);   // xác nhận xử lý thành công → xóa khỏi queue
    } catch (err) {
      channel.nack(msg);  // xử lý thất bại → đưa lại vào queue
    }
  });
}
```

```yaml
# Docker Compose cho RabbitMQ
services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: rabbitmq
    ports:
      - "5672:5672"    # AMQP port (kết nối từ app)
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: password
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  rabbitmq_data:
```

**Management UI** có sẵn tại `http://localhost:15672` — không cần cài thêm tool như Kafka.

---

## 4. So Sánh Kafka và RabbitMQ

| Tiêu chí | Kafka | RabbitMQ |
|----------|-------|----------|
| **Throughput** | Cực cao (triệu msg/s) | Cao (nghìn msg/s) |
| **Độ phức tạp** | Cao (cần Zookeeper) | Thấp, dễ setup |
| **Lưu message** | Lâu dài (ngày/tuần) | Xóa sau khi consumer ACK |
| **Replay message** | Có | Không |
| **Routing** | Đơn giản (topic/partition) | Linh hoạt (exchange types) |
| **Management UI** | Cần cài thêm (Kafka UI) | Có sẵn |
| **Phù hợp** | Stream processing, log, event sourcing | Task queue, notifications, RPC |

---

## 5. Khi Nào Dùng Gì?

| Tình huống | Chọn |
|------------|------|
| Gửi email, SMS sau khi đặt hàng | RabbitMQ |
| Xử lý hàng triệu log mỗi giây | Kafka |
| Thông báo real-time cho nhiều service | RabbitMQ (fanout) |
| Event sourcing, audit trail | Kafka (lưu lâu dài) |
| Task queue đơn giản | RabbitMQ |
| Data pipeline, analytics | Kafka |
| Microservice thông thường | RabbitMQ |
| Hệ thống tài chính, tracking | Kafka |

---

## 6. Tổng Kết

| Nhóm | Nội dung |
|------|----------|
| **Kafka** | Streaming phân tán, throughput cao, lưu lâu dài — Tool: Kafka UI |
| **RabbitMQ** | Message broker linh hoạt, dễ dùng — Tool: Management UI có sẵn |

