# Network

---

## 1. Giới Thiệu

**Network** (Mạng máy tính) là hệ thống kết nối các thiết bị để trao đổi dữ liệu. Hiểu về mạng là nền tảng không thể thiếu với mọi lập trình viên backend, DevOps hay system engineer.


---

## 2. 7 Network Layers (Mô Hình OSI)

Mô hình OSI (Open Systems Interconnection) chia quá trình truyền dữ liệu thành 7 tầng, mỗi tầng có chức năng riêng biệt.

| Tầng | Tên | Chức năng | Giao thức / Thiết bị |
|------|-----|-----------|----------------------|
| 7 | Application | Giao tiếp với ứng dụng người dùng | HTTP, HTTPS, FTP, SMTP, DNS |
| 6 | Presentation | Mã hóa, nén, định dạng dữ liệu | SSL/TLS, JPEG, UTF-8 |
| 5 | Session | Quản lý phiên kết nối | NetBIOS, RPC |
| 4 | Transport | Truyền dữ liệu đầu-cuối, kiểm soát lỗi | TCP, UDP |
| 3 | Network | Định tuyến gói tin | IP, ICMP, Router |
| 2 | Data Link | Truyền frame giữa 2 node kề nhau | Ethernet, MAC, Switch |
| 1 | Physical | Truyền bit qua môi trường vật lý | Cáp, Wi-Fi, Hub |


**Trong thực tế hay gặp:**
- **Tầng 4 (Transport):** TCP đảm bảo truyền tin cậy, UDP truyền nhanh và không đảm bảo thường dùng cho video stream, game online
- **Tầng 3 (Network):** IP routing — cơ sở của toàn bộ internet
- **Tầng 7 (Application):** HTTP/HTTPS — giao thức web phổ biến nhất 

---

## 3. Domain / Subdomain / Subnet

### 3.1 Domain

**Domain** (tên miền) là địa chỉ dễ đọc thay thế cho địa chỉ IP.

```
https://api.shop.example.com/products
        │   │       │
        │   │       └── TLD (Top-Level Domain)
        │   └────────── Second-Level Domain
        └────────────── Subdomain
```

**Các loại TLD phổ biến:**

| TLD | Ý nghĩa |
|-----|---------|
| `.com` | Thương mại |
| `.org` | Tổ chức phi lợi nhuận |
| `.vn` | Việt Nam (country code) |
| `.io` | Phổ biến trong tech startup |
| `.dev` | Dành cho developer |

### 3.2 Subdomain

Subdomain là phần mở rộng đứng trước domain chính, thường dùng để phân tách môi trường hoặc dịch vụ.

```
example.com          → Trang chính
www.example.com      → Web (truyền thống)
api.example.com      → REST API
admin.example.com    → Trang quản trị
staging.example.com  → Môi trường test
mail.example.com     → Email server
```

### 3.3 Subnet (Mạng con)

**Subnet** là phân vùng nhỏ hơn trong một mạng lớn, giúp tổ chức và bảo mật hệ thống.

```
Mạng: 192.168.1.0/24
├── Subnet A: 192.168.1.0/26   → 62 hosts (servers)
├── Subnet B: 192.168.1.64/26  → 62 hosts (workstations)
└── Subnet C: 192.168.1.128/26 → 62 hosts (IoT devices)
```

**CIDR Notation phổ biến:**

| CIDR | Số host khả dụng | Dùng cho |
|------|-----------------|---------|
| `/32` | 1 | Single host |
| `/24` | 254 | Mạng nhỏ (LAN) |
| `/16` | 65.534 | Mạng lớn |
| `/8` | 16.777.214 | ISP, cloud provider |

---

## 4. VPN (Virtual Private Network)

**VPN** tạo ra đường hầm (tunnel) mã hóa giữa thiết bị và server, giúp bảo mật kết nối và ẩn địa chỉ IP thực.

**Cách VPN hoạt động:**

```
Thiết bị → [Mã hóa] → VPN Server → [Giải mã] → Internet
                ↑
          Tunnel an toàn
```

**Các giao thức VPN phổ biến:**

| Giao thức | Tốc độ | Bảo mật | Dùng cho |
|-----------|--------|---------|---------|
| **OpenVPN** | Trung bình | Cao | Self-hosted VPN |
| **WireGuard** | Rất nhanh | Cao | VPN hiện đại, đơn giản |
| **IPSec/IKEv2** | Nhanh | Cao | Mobile, enterprise |
| **L2TP** | Chậm | Trung bình | Legacy systems |

**Ứng dụng VPN trong thực tế:**

- **Remote work:** Nhân viên kết nối an toàn vào mạng nội bộ công ty
- **Site-to-Site VPN:** Kết nối 2 văn phòng ở 2 địa điểm khác nhau
- **Dev/Ops:** Truy cập server nội bộ, database không public ra internet
- **Bảo mật:** Mã hóa traffic trên Wi-Fi công cộng

```bash
# Kết nối WireGuard VPN
wg-quick up wg0

# Kiểm tra trạng thái
wg show

# Ngắt kết nối
wg-quick down wg0
```

---

## 5. IP (Internet Protocol)

### 5.1 IPv4

**IPv4** là phiên bản IP phổ biến nhất, gồm 4 octet (32-bit).

```
192  .  168  .   1   .  100
 │        │       │       │
Octet 1  Octet 2 Octet 3 Octet 4
(0-255)  (0-255) (0-255) (0-255)
```

**Các dải IP đặc biệt:**

| Dải IP | Loại | Mô tả |
|--------|------|-------|
| `10.0.0.0/8` | Private | Mạng nội bộ lớn |
| `172.16.0.0/12` | Private | Mạng nội bộ trung bình |
| `192.168.0.0/16` | Private | Mạng gia đình, văn phòng |
| `127.0.0.1` | Loopback | Localhost |
| `0.0.0.0` | Wildcard | Tất cả interfaces |

### 5.2 IPv6

**IPv6** là phiên bản mới (128-bit), giải quyết vấn đề cạn kiệt địa chỉ IPv4.

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
                    ↓ Rút gọn
2001:db8:85a3::8a2e:370:7334
```

### 5.3 DNS — Chuyển Domain sang IP

```
Người dùng nhập: google.com
      ↓
DNS Resolver → Root Server → TLD Server → Authoritative Server
      ↓
Trả về IP: 142.250.185.46
      ↓
Trình duyệt kết nối tới IP đó
```

**Các lệnh network hữu ích:**

```bash
# Xem IP hiện tại
ip addr show          # Linux
ipconfig              # Windows

# Kiểm tra kết nối
ping google.com
traceroute google.com # Linux
tracert google.com    # Windows

# Tra cứu DNS
nslookup google.com
dig google.com

# Xem các kết nối đang mở
netstat -tuln
ss -tuln              # Thay thế netstat hiện đại hơn
```

---

## 6. Ví Dụ Thực Tế: Dùng ZeroTier để Remote Máy ở Nhà

ZeroTier là giải pháp VPN peer-to-peer miễn phí, cho phép tạo mạng ảo riêng giữa các thiết bị mà không cần cấu hình router hay port forwarding.

Các bước thiết lập:

Bước 1 — Tạo Network trên ZeroTier Central:

1. Đăng ký tài khoản tại https://my.zerotier.com
2. Tạo Network mới → nhận Network ID (vd: a1b2c3d4e5f6a7b8)
3. Chọn chế độ Private (phải approve thủ công)
![alt text](photo/image.png)

Bước 2 — Cài ZeroTier trên máy ở nhà (Windows):

powershell# Tải installer tại https://www.zerotier.com/download/
# Sau khi cài, join network
zerotier-cli join a1b2c3d4e5f6a7b8
UI:
![alt text](photo/image-1.png)
# Kiểm tra trạng thái
zerotier-cli status
zerotier-cli listnetworks

Bước 3 — Cài ZeroTier trên máy remote (Linux/Mac):

bash# Linux
curl -s https://install.zerotier.com | sudo bash
sudo zerotier-cli join a1b2c3d4e5f6a7b8


Bước 4 — Approve thiết bị trên ZeroTier Central:

1. Vào https://my.zerotier.com → chọn Network
2. Mục Members → tick ô Auth cho cả 2 thiết bị
3. Mỗi thiết bị sẽ được cấp IP ảo (vd: 192.168.191.x)

![alt text](photo/image-2.png)

Bước 5 — Remote Desktop vào máy ở nhà:

# Bật Remote Desktop trên máy ở nhà (Windows)
Settings → System → Remote Desktop → Enable

# Từ máy remote, kết nối qua IP ảo ZeroTier
mstsc /v:192.168.195.110

---
## 7. Tổng Kết

| Chủ đề | Nội dung chính |
|--------|----------------|
| **7 Network Layers** | Mô hình OSI — từ Physical đến Application |
| **Domain/Subdomain/Subnet** | Cách tổ chức địa chỉ mạng và phân vùng |
| **VPN** | Bảo mật kết nối qua tunnel mã hóa |
| **IP** | IPv4, IPv6, DNS resolution |


