# Bài 3: Tách biệt luồng CI/CD đa môi trường (Staging & Production) với GitHub Environments

## 1. Mục tiêu bài thực hành
- Thiết lập và quản lý hạ tầng triển khai đa môi trường (**Staging** và **Production**) thông qua tính năng **GitHub Environments**.
- Cấu hình quy tắc bảo vệ môi trường (**Environment Protection Rules**) với tính năng **Required reviewers** để bắt buộc phê duyệt thủ công trước khi mã nguồn được triển khai lên Production.
- Tự động hóa phân luồng triển khai:
  - Push nhánh `develop` ➔ Tự động deploy lên môi trường `staging` (không cần phê duyệt).
  - Push nhánh `main` ➔ Kích hoạt deploy lên môi trường `production`, tạm dừng ở trạng thái **Waiting for review** và chỉ tiếp tục sau khi được phê duyệt.

---

## 2. Các bước cấu hình trên GitHub Repository

### Bước 1: Tạo môi trường `staging` và `production`
1. Truy cập vào GitHub Repository của bạn trên trình duyệt.
2. Điều hướng tới tab **Settings** ➔ chọn mục **Environments** ở thanh menu bên trái.
3. Bấm nút **New environment**:
   - Nhập tên: `staging` ➔ Bấm **Configure environment**.
   - Bấm **Save environment** (không cần bật Required reviewers cho staging).
4. Tiếp tục bấm nút **New environment** lần nữa:
   - Nhập tên: `production` ➔ Bấm **Configure environment**.

### Bước 2: Thiết lập Environment Protection Rules cho môi trường `production`
1. Trong màn hình cấu hình của môi trường `production`, tìm mục **Environment protection rules**.
2. Tích chọn vào ô **Required reviewers**.
3. Nhập tên tài khoản GitHub cá nhân của bạn vào ô tìm kiếm và chọn tài khoản đó làm người duyệt.
   *(Tùy chọn: Bỏ tích "Prevent self-review" nếu tài khoản của bạn là người kích hoạt workflow và cũng là người duyệt).*
4. Bấm nút **Save protection rules** để hoàn tất cấu hình.

---

## 3. Cấu trúc kịch bản GitHub Actions Workflow

File kịch bản được lưu tại: [.github/workflows/cd-multi-env.yml](file:///c:/Users/ADMIN/OneDrive/Desktop/IT209/Session12/Lesson3/.github/workflows/cd-multi-env.yml)

### Giải thích các khối cấu hình chính:
- **`on.push.branches`**: Lắng nghe sự kiện push lên 2 nhánh `main` và `develop`.
- **`jobs.deploy-staging`**:
  - Điều kiện: `if: github.ref == 'refs/heads/develop'` đảm bảo chỉ chạy khi có mã nguồn mới trên nhánh `develop`.
  - Cấu hình `environment.name: staging` và gán URL tương ứng.
- **`jobs.deploy-prod`**:
  - Điều kiện: `if: github.ref == 'refs/heads/main'` đảm bảo chỉ chạy khi có mã nguồn mới trên nhánh `main`.
  - Cấu hình `environment.name: production` và `url: https://api.prod.example.com`.
  - Khi Job này bắt đầu, GitHub Actions sẽ tự động kiểm tra quy tắc bảo vệ của `production` và tạm dừng để chờ phê duyệt.

---

## 4. Hướng dẫn kiểm tra luồng hoạt động (Testing)

### Kịch bản 1: Kiểm tra triển khai Staging (nhánh `develop`)
1. Tạo hoặc chuyển sang nhánh `develop`:
   ```bash
   git checkout -b develop
   git add .
   git commit -m "feat: test staging deployment"
   git push origin develop
   ```
2. Vào tab **Actions** trên GitHub:
   - Quan sát workflow chạy Job `Deploy to Staging`.
   - Job này sẽ thực thi ngay lập tức và hoàn thành với tích xanh (Green Checkmark) mà không cần bước phê duyệt.

### Kịch bản 2: Kiểm tra triển khai Production (nhánh `main`)
1. Chuyển sang nhánh `main` (hoặc merge từ `develop` vào `main`):
   ```bash
   git checkout main
   git merge develop
   git push origin main
   ```
2. Vào tab **Actions** trên GitHub:
   - Workflow khởi chạy Job `Deploy to Production`.
   - Luồng chạy sẽ **dừng lại** với trạng thái: **"Waiting for review"** (có thông báo màu vàng/cam yêu cầu xét duyệt).
3. **Thực hiện phê duyệt & Chụp ảnh minh chứng**:
   - Bấm vào nút **Review deployments**.
   - Tích chọn môi trường **production**.
   - Bấm nút **Approve and deploy**.
   - Chụp lại màn hình popup phê duyệt (modal review deployment) và màn hình sau khi job tiếp tục chạy thành công với tích xanh.

---

## 5. Hướng dẫn nộp bài
- **Tệp kịch bản workflow**:
  - Vị trí thực thi: `.github/workflows/cd-multi-env.yml`
  - Vị trí gợi ý theo đề bài: `homework/session_12/ex3/cd-multi-env.yml`
- **Ảnh chụp màn hình**:
  - Ảnh 1: Cấu hình Environment `production` có bật **Required reviewers** trong `Settings -> Environments`.
  - Ảnh 2: Màn hình cửa sổ phê duyệt deployment (**Review deployments** / **Waiting for review**).
  - Ảnh 3: Màn hình kết quả Job `Deploy to Production` chạy thành công sau khi được Approve.
