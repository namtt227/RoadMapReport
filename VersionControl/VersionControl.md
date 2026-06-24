# Báo Cáo: Version Control với Git

---

## 1. Giới Thiệu

**Version Control**  là hệ thống ghi lại các thay đổi của tệp theo thời gian, cho phép người dùng quay lại các phiên bản cụ thể sau này. 



---

## 2. Concepts

### 2.1 Repositories

Repository là thư mục chứa toàn bộ mã nguồn và lịch sử thay đổi của dự án.

- **Local Repository**: Kho lưu trữ trên máy tính cá nhân.
- **Remote Repository**: Kho lưu trữ trên máy chủ từ xa (GitHub, GitLab...).

Lệnh khởi tạo remote Respotories:
```bash
git init <tên folder>
git clone https://github.com/username/<tên folder>.git
```

### 2.2 Commits

Commit là chụp lại trạng thái dự án tại một thời điểm cụ thể.

```bash
git add .
git commit -m "comment"
git log --oneline (lịch sử các commit)
```

### 2.3 Branches

Branch cho phép làm việc song song trên bản sao của dự án bằng một nhánh khác
checkout: chuyển nhánh
branch: tạo nhánh 

```bash
git checkout -b feature/login
git branch -a 
```

### 2.4 Merging

Merging là quá trình kết hợp thay đổi từ nhánh này vào nhánh khác.
merge: kết hợp thanh đổi ở nhánh hiện tại -> nhánh merge
merge tạo thêm merge commit.
```bash
git checkout main
git merge feature/login
```

### 2.5 Rebasing

Rebasing tích hợp thay đổi theo cách tạo ra lịch sử commit tuyến tính và sạch hơn.
rebase tái cấu trúc lại lịch sử commit.
```bash
git checkout feature/login
git rebase main
```
Không nên rebase các dự án mã nguồn online

### 2.6 Pull Requests

Pull Request là cơ chế đề xuất thay đổi code và yêu cầu review trước khi merge. Đây là trung tâm của quy trình cộng tác hiện đại.

**Quy trình PR :**

1. Tạo nhánh mới từ `main`
2. Thực hiện thay đổi và commit
3. Push lên remote
4. Mở Pull Request
5. Review và thảo luận
6. Approve và merge

---

## 3. Platforms

### 3.1 GitHub

GitHub là nền tảng lưu trữ code phổ biến nhất thế giới, thuộc sở hữu của Microsoft.

| Tính năng | Mô tả |
|-----------|-------|
| Repository hosting | Lưu trữ code miễn phí và trả phí |
| GitHub Actions | CI/CD tích hợp sẵn |
| GitHub Pages | Deploy website tĩnh miễn phí |
| Issues & Projects | Quản lý công việc, theo dõi lỗi |
| GitHub Copilot | AI hỗ trợ viết code |

### 3.2 GitLab

GitLab là nền tảng DevOps toàn diện, đặc biệt mạnh về CI/CD và hỗ trợ tự host.

| Tính năng | Mô tả |
|-----------|-------|
| GitLab CI/CD | Pipeline tích hợp mạnh mẽ |
| Container Registry | Lưu trữ Docker images |
| Self-hosted | Cài đặt trên server riêng miễn phí |
| Auto DevOps | Tự động hóa toàn bộ DevOps pipeline |



---

## 4. Workflow

### 4.1 Git Flow

Git Flow là mô hình branching phù hợp với dự án có chu kỳ phát hành rõ ràng.

**Các nhánh trong Git Flow:**

| Nhánh | Vai trò |
|-------|---------|
| `main` | Code production ổn định |
| `develop` | Tích hợp các tính năng mới |
| `feature/*` | Phát triển tính năng |
| `release/*` | Chuẩn bị phát hành |
| `hotfix/*` | Sửa lỗi khẩn cấp |

```bash
git flow init
git flow feature start user-authentication
git flow feature finish user-authentication
git flow release start 1.0.0
git flow release finish 1.0.0
```

**Ưu điểm:** Cấu trúc rõ ràng, phù hợp phần mềm có versioning.  
**Nhược điểm:** Phức tạp cho dự án nhỏ, lịch sử commit dễ rối.

### 4.2 GitHub Flow

GitHub Flow đơn giản hơn, phù hợp với nhóm thực hành Continuous Deployment.

**Nguyên tắc:**

1. `main` luôn có thể deploy
2. Tạo nhánh mới cho mỗi tính năng/sửa lỗi
3. Commit thường xuyên
4. Mở Pull Request để review
5. Merge vào `main` sau khi approve
6. Deploy ngay sau khi merge

```bash
git checkout -b feature/new-feature
git add . && git commit -m "feat: thêm tính năng mới"
git push origin feature/new-feature
```

---

## 5. Ví Dụ Thực Tế 

### 5.1 Cộng Tác Nhóm

Khi làm việc nhóm, cần tuân theo các quy ước chung để cộng tác hiệu quả.

**Quy trình :**

```bash
git clone https://github.com/team/project.git

git checkout -b feature/user-dashboard

git commit -m "feat(dashboard): thêm biểu đồ thống kê"
git commit -m "fix(auth): sửa lỗi đăng nhập"

git fetch origin 
git rebase origin/main

git push origin feature/user-dashboard
```

**Chuẩn Conventional Commits:**

| Loại | Ý nghĩa | Ví dụ |
|------|---------|-------|
| `feat` | Tính năng mới | `feat: thêm tìm kiếm` |
| `fix` | Sửa lỗi | `fix: sửa lỗi hiển thị ngày` |
| `docs` | Tài liệu | `docs: cập nhật README` |
| `refactor` | Tái cấu trúc | `refactor: tối ưu thuật toán` |
| `test` | Kiểm thử | `test: thêm unit test` |
| `chore` | Công việc phụ | `chore: cập nhật dependencies` |

**Checklist review Pull Request:**

-  Code đáp ứng yêu cầu tính năng
-  Không có lỗi logic
-  Có unit test đầy đủ
-  Documentation được cập nhật
-  Không có thông tin nhạy cảm (API key, mật khẩu)



