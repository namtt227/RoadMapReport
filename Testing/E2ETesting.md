# End-to-End (E2E) Testing — Kiến Thức Chi Tiết

---

## 1. End-to-End Testing là gì?

End-to-End Testing là cấp độ kiểm thử cao nhất trong kim tự tháp kiểm thử, kiểm tra **toàn bộ luồng hoạt động của hệ thống từ đầu đến cuối**, mô phỏng đúng hành vi mà người dùng thật sẽ thực hiện — từ giao diện, qua backend, database, cho đến các service bên thứ ba (nếu có).

Mục tiêu của E2E test không phải kiểm tra logic chi tiết của từng thành phần (đã được unit/integration test đảm nhiệm), mà là xác nhận rằng **cả hệ thống hoạt động đúng như một tổng thể** khi được sử dụng theo đúng kịch bản thực tế.

---

## 2. Các khái niệm cốt lõi

### 2.1 Simulating User Flows
Mô phỏng các luồng thao tác thực tế mà người dùng thực hiện trên hệ thống, ví dụ:
- Đăng ký tài khoản → xác thực email → đăng nhập.
- Tìm sản phẩm → thêm vào giỏ hàng → thanh toán → nhận email xác nhận đơn hàng.

Mỗi luồng (flow) thường bao gồm nhiều bước liên tiếp, đi qua nhiều trang/màn hình và nhiều thành phần hệ thống khác nhau, nên một E2E test thường mất nhiều thời gian chạy hơn unit/integration test.

### 2.2 Browser Automation
Kỹ thuật điều khiển trình duyệt bằng code để tự động thực hiện các hành động mà người dùng thường làm bằng tay:
- Click vào nút, liên kết
- Nhập dữ liệu vào form
- Điều hướng giữa các trang
- Kiểm tra nội dung hiển thị trên giao diện (DOM)

Browser automation hoạt động thông qua các giao thức điều khiển trình duyệt (VD: WebDriver protocol, Chrome DevTools Protocol) để "giả lập" thao tác người dùng một cách chính xác và có thể lặp lại.

---

## 3. Công cụ sử dụng

### 3.1 Selenium
- Công cụ automation trình duyệt lâu đời nhất, hỗ trợ hầu hết các ngôn ngữ lập trình (Java, Python, C#, JavaScript...) và nhiều trình duyệt (Chrome, Firefox, Safari, Edge).
- Hoạt động dựa trên chuẩn **WebDriver**, giao tiếp với trình duyệt qua driver riêng cho từng loại (ChromeDriver, GeckoDriver...).
- Cú pháp cơ bản (Python):
```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://example.com/login")

driver.find_element(By.ID, "username").send_keys("test_user")
driver.find_element(By.ID, "password").send_keys("password123")
driver.find_element(By.ID, "login-button").click()

assert "Dashboard" in driver.title
driver.quit()
```
- Ưu điểm: hỗ trợ đa nền tảng rộng, cộng đồng lớn. Nhược điểm: cấu hình phức tạp hơn, tốc độ chạy chậm hơn các công cụ mới.

### 3.2 Cypress
- Công cụ E2E testing hiện đại, chạy trực tiếp trong trình duyệt (không qua WebDriver), giúp test nhanh và dễ debug hơn.
- Có giao diện trực quan (Cypress Test Runner) hiển thị từng bước test chạy theo thời gian thực, kèm ảnh chụp màn hình khi lỗi.
- Cú pháp (JavaScript):
```javascript
describe('Login flow', () => {
  it('should login successfully', () => {
    cy.visit('/login');
    cy.get('#username').type('test_user');
    cy.get('#password').type('password123');
    cy.get('#login-button').click();
    cy.url().should('include', '/dashboard');
  });
});
```
- Hạn chế: chủ yếu hỗ trợ trình duyệt gốc Chromium/Firefox, khó test đa tab hoặc đa domain phức tạp.

### 3.3 Playwright
- Công cụ automation hiện đại do Microsoft phát triển, hỗ trợ đa trình duyệt (Chromium, Firefox, WebKit) và đa ngôn ngữ (JavaScript/TypeScript, Python, Java, .NET).
- Hỗ trợ tốt việc test đa tab, đa domain, và có cơ chế **auto-wait** (tự động chờ phần tử sẵn sàng) giúp giảm test không ổn định (flaky test).
- Cú pháp (JavaScript):
```javascript
const { test, expect } = require('@playwright/test');

test('login flow', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#username', 'test_user');
  await page.fill('#password', 'password123');
  await page.click('#login-button');
  await expect(page).toHaveURL(/dashboard/);
});
```
- Dù thiên về kiểm thử frontend, Playwright vẫn hữu ích cho vai trò full-stack vì có thể kết hợp kiểm tra cả request/response API trong cùng một test.

---

## 4. Critical User Paths (Luồng người dùng quan trọng)

Vì E2E test có chi phí chạy và bảo trì cao, không nên viết E2E test cho mọi tính năng mà nên **ưu tiên các luồng quan trọng nhất** (critical paths) — những luồng nếu bị lỗi sẽ ảnh hưởng nghiêm trọng đến trải nghiệm người dùng hoặc doanh thu, ví dụ:
- Đăng ký / Đăng nhập
- Tìm kiếm và xem sản phẩm
- Thêm vào giỏ hàng và thanh toán
- Quên mật khẩu / đặt lại mật khẩu

---

## 5. Vấn đề thường gặp: Flaky Test

Flaky test là test đôi khi pass, đôi khi fail dù code không thay đổi, thường do:
- Phần tử giao diện chưa load xong khi test cố gắng tương tác (thiếu wait/timing).
- Dữ liệu test không ổn định giữa các lần chạy.
- Phụ thuộc vào thời gian thực hoặc animation trên giao diện.

Cách khắc phục: dùng cơ chế **explicit wait** hoặc **auto-wait** (Playwright hỗ trợ sẵn), cô lập dữ liệu test cho mỗi lần chạy, tránh phụ thuộc vào thứ tự chạy giữa các test.

---

## 6. Testing Pyramid — Tổng hợp 3 cấp độ

| Cấp độ | Phạm vi kiểm tra | Tốc độ chạy | Số lượng nên có |
|---|---|---|---|
| Unit Testing | Một hàm/class riêng lẻ | Rất nhanh | Nhiều nhất |
| Integration Testing | Nhiều thành phần phối hợp | Trung bình | Vừa phải |
| E2E Testing | Toàn hệ thống, qua giao diện | Chậm nhất | Ít nhất, chỉ critical paths |

Mô hình kim tự tháp kiểm thử khuyến nghị: viết nhiều unit test ở đáy (rẻ, nhanh), giảm dần số lượng integration test ở giữa, và chỉ giữ lại số ít E2E test ở đỉnh cho các luồng quan trọng nhất — nhằm cân bằng giữa độ tin cậy và chi phí bảo trì.