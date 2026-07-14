# Unit Testing 

---

## 1. Unit Testing là gì?

Unit Testing là cấp độ kiểm thử thấp nhất trong kim tự tháp kiểm thử phần mềm, tập trung vào việc kiểm tra tính đúng đắn của **một đơn vị code nhỏ nhất, độc lập** — thường là một hàm (function), một phương thức (method) hoặc một class — tách biệt hoàn toàn khỏi các thành phần khác của hệ thống (database, network, file system, service khác...).

Nguyên tắc cốt lõi: mỗi unit test phải **nhanh, độc lập, và lặp lại được** (fast, isolated, repeatable). Nếu một hàm gọi tới database hoặc API bên ngoài, ta cần "giả lập" (mock) các phần đó để chỉ tập trung kiểm tra logic của riêng hàm đang test.

---

## 2. Các khái niệm cốt lõi

### 2.1 Test Case
Một test case mô tả một kịch bản kiểm thử cụ thể, thường theo cấu trúc **Arrange – Act – Assert (AAA)**:
- **Arrange**: chuẩn bị dữ liệu đầu vào, khởi tạo đối tượng cần test.
- **Act**: gọi hàm/phương thức cần kiểm thử.
- **Assert**: so sánh kết quả trả về với kết quả mong đợi.

Ví dụ (Python, dùng Pytest):
```python
def test_add_numbers():
    # Arrange
    a, b = 2, 3
    # Act
    result = add(a, b)
    # Assert
    assert result == 5
```

Một hàm nên có nhiều test case để bao phủ các trường hợp: input hợp lệ, input biên (edge case), input không hợp lệ (lỗi/ngoại lệ).

### 2.2 Assertions
Assertion là câu lệnh khẳng định kết quả thực tế đúng như kỳ vọng. Các loại assertion phổ biến:
- So sánh giá trị: `assertEqual`, `assertTrue`, `assertFalse`
- Kiểm tra ngoại lệ: `assertRaises` (Python), `expect(() => fn()).toThrow()` (Jest)
- Kiểm tra kiểu dữ liệu, độ dài, thuộc tính của object

### 2.3 Mocking
Mocking là kỹ thuật thay thế các dependency thật (database, API, service ngoài) bằng các đối tượng giả (mock/stub/fake) có hành vi được định nghĩa trước, giúp:
- Test chạy nhanh hơn (không cần kết nối thật).
- Test ổn định hơn (không phụ thuộc vào trạng thái bên ngoài).
- Kiểm soát được các tình huống khó tái hiện (VD: lỗi mạng, timeout).

Các loại test double:
| Loại | Mô tả |
|---|---|
| **Dummy** | Đối tượng chỉ để truyền vào cho đủ tham số, không dùng thật |
| **Stub** | Trả về giá trị cố định khi được gọi |
| **Mock** | Ghi lại cách nó được gọi (bao nhiêu lần, tham số gì) để verify sau |
| **Fake** | Cài đặt đơn giản hoạt động thật nhưng không dùng cho production (VD: in-memory database) |

Ví dụ mocking trong Python:
```python
from unittest.mock import Mock

payment_service = Mock()
payment_service.charge.return_value = True

result = process_order(payment_service, order)
payment_service.charge.assert_called_once_with(order.amount)
```

### 2.4 Test Runner
Test runner là công cụ tự động tìm, thực thi các file test và tổng hợp báo cáo kết quả (pass/fail/skip), thường tích hợp với CI/CD để tự động chạy mỗi khi có commit/push.

---

## 3. Framework theo từng ngôn ngữ

### 3.1 Pytest (Python)
- Cú pháp đơn giản, không cần class, chỉ cần hàm bắt đầu bằng `test_`.
- Hỗ trợ **fixture** để tái sử dụng logic setup/teardown:
```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "An", "age": 25}

def test_user_age(sample_user):
    assert sample_user["age"] == 25
```
- Hỗ trợ **parametrize** để chạy một test với nhiều bộ dữ liệu khác nhau.
- Plugin phổ biến: `pytest-mock`, `pytest-cov` (đo coverage).

### 3.2 JUnit (Java)
- Framework unit test chuẩn của Java, dùng annotation: `@Test`, `@BeforeEach`, `@AfterEach`.
```java
@Test
void testAddNumbers() {
    int result = Calculator.add(2, 3);
    assertEquals(5, result);
}
```
- Thường kết hợp với **Mockito** để mocking.

### 3.3 Jest / Mocha (Node.js)
- **Jest**: framework tất cả trong một (test runner + assertion + mocking), phổ biến với React/Node.
```javascript
test('adds 2 + 3 to equal 5', () => {
  expect(add(2, 3)).toBe(5);
});
```
- **Mocha**: chỉ là test runner, thường kết hợp với thư viện assertion riêng như `Chai` và mocking library `Sinon`.

### 3.4 Go testing package
- Package `testing` có sẵn trong chuẩn thư viện Go, không cần cài thêm framework ngoài.
```go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("expected 5, got %d", result)
    }
}
```
- Chạy bằng lệnh `go test ./...`.

---

## 4. Code Coverage

Code coverage đo tỷ lệ phần trăm code được thực thi khi chạy bộ test, giúp đánh giá mức độ "phủ" của test:
- **Line coverage**: % số dòng code được chạy qua.
- **Branch coverage**: % số nhánh điều kiện (if/else) được kiểm tra.

Lưu ý: coverage cao không đồng nghĩa với chất lượng test tốt — cần đảm bảo assertion thực sự kiểm tra đúng logic, không chỉ chạy qua code mà không assert gì.

---

