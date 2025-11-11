# Hướng Dẫn Chi Tiết: Thiết Lập CI/CD với GitHub Actions và Google Cloud

## Mục Lục
1. [Tổng Quan](#tổng-quan)
2. [Cấu Trúc Pipeline](#cấu-trúc-pipeline)
3. [Giải Thích Chi Tiết Workflow](#giải-thích-chi-tiết-workflow)
4. [Cấu Hình Google Cloud](#cấu-hình-google-cloud)
5. [Cấu Hình Secrets trong GitHub](#cấu-hình-secrets-trong-github)
6. [Kiểm Thử Tự Động với Jest](#kiểm-thử-tự-động-với-jest)
7. [Triển Khai Ứng Dụng](#triển-khai-ứng-dụng)
8. [Xử Lý Lỗi và Debugging](#xử-lý-lỗi-và-debugging)

---

## Tổng Quan

### CI/CD là gì?
- **CI (Continuous Integration)**: Tích hợp liên tục - Tự động build và test code mỗi khi có thay đổi
- **CD (Continuous Deployment)**: Triển khai liên tục - Tự động deploy ứng dụng lên môi trường production

### Lợi ích của CI/CD
- ✅ Phát hiện lỗi sớm
- ✅ Tự động hóa quy trình
- ✅ Giảm thiểu lỗi thủ công
- ✅ Tăng tốc độ phát triển
- ✅ Đảm bảo chất lượng code

---

## Cấu Trúc Pipeline

Pipeline của chúng ta bao gồm 3 jobs chính:

```
┌─────────────────────┐
│  build-and-test     │  ← Chạy song song trên Node.js 16, 18, 20
│  (Matrix Strategy)  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  deploy             │  ← Deploy lên Google App Engine
│  (Chỉ khi pass)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  report             │  ← Tạo báo cáo tổng hợp
│  (Luôn chạy)        │
└─────────────────────┘
```

---

## Giải Thích Chi Tiết Workflow

### 1. Phần Trigger (Kích Hoạt)

```yaml
on:
  push:
    branches:
      - main
      - development
  pull_request:
    branches:
      - main
```

**Giải thích:**
- Pipeline sẽ chạy khi:
  - Có push vào nhánh `main` hoặc `development`
  - Có Pull Request vào nhánh `main`
- Điều này đảm bảo mọi thay đổi đều được kiểm tra trước khi merge

### 2. Job: build-and-test

#### a. Matrix Strategy

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
  fail-fast: false
```

**Giải thích:**
- **Matrix Strategy**: Tạo nhiều job chạy song song với các giá trị khác nhau
- `node-version: [16, 18, 20]`: Chạy test trên 3 phiên bản Node.js khác nhau
- `fail-fast: false`: Tiếp tục chạy các job khác dù một job thất bại
- **Lợi ích**: Đảm bảo ứng dụng tương thích với nhiều phiên bản Node.js

#### b. Các Bước trong Job

**Bước 1: Checkout code**
```yaml
- name: Checkout code
  uses: actions/checkout@v3
```
- Tải mã nguồn từ repository về runner

**Bước 2: Setup Node.js**
```yaml
- name: Set up Node.js ${{ matrix.node-version }}
  uses: actions/setup-node@v3
  with:
    node-version: ${{ matrix.node-version }}
    cache: 'npm'
```
- Cài đặt Node.js với phiên bản từ matrix (16, 18, hoặc 20)
- `cache: 'npm'`: Cache node_modules để tăng tốc độ cài đặt

**Bước 3: Install dependencies**
```yaml
- name: Install dependencies
  run: npm ci
```
- `npm ci`: Cài đặt chính xác từ package-lock.json (nhanh hơn và đảm bảo version chính xác)

**Bước 4: Run tests**
```yaml
- name: Run tests
  run: npm test
  continue-on-error: false
```
- Chạy Jest test suite
- `continue-on-error: false`: Dừng pipeline nếu test thất bại

**Bước 5: Generate test coverage**
```yaml
- name: Generate test coverage
  if: always()
  run: npm run test:coverage || true
  continue-on-error: true
```
- `if: always()`: Chạy dù test pass hay fail
- Tạo báo cáo coverage (tỷ lệ code được test)

**Bước 6: Upload coverage reports**
```yaml
- name: Upload coverage reports
  uses: actions/upload-artifact@v3
  with:
    name: coverage-report-node-${{ matrix.node-version }}
    path: coverage/
```
- Lưu coverage report dưới dạng artifact để tải về sau

**Bước 7: Build application**
```yaml
- name: Build application
  run: npm run build
```
- Build ứng dụng (nếu có bước build)

### 3. Job: deploy

#### Điều kiện chạy:
```yaml
needs: build-and-test
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```

**Giải thích:**
- `needs: build-and-test`: Chỉ chạy khi job build-and-test thành công
- `if: ...`: Chỉ deploy khi push vào nhánh main (không deploy khi PR)

#### Các bước deploy:

**Bước 1-4: Setup và Build** (tương tự job build-and-test)

**Bước 5: Authenticate to Google Cloud**
```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}
```
- Xác thực với Google Cloud bằng Service Account key

**Bước 6: Set up Cloud SDK**
```yaml
- name: Set up Cloud SDK
  uses: google-github-actions/setup-gcloud@v2
  with:
    project_id: ${{ secrets.GCP_PROJECT_ID }}
```
- Cài đặt gcloud CLI và set project ID

**Bước 7: Deploy to App Engine**
```yaml
- name: Deploy to Google App Engine
  uses: google-github-actions/deploy-appengine@v2
  with:
    project_id: ${{ secrets.GCP_PROJECT_ID }}
    deliverables: app.yaml
    version: ${{ github.sha }}
    promote: true
```
- Deploy ứng dụng lên App Engine
- `version: ${{ github.sha }}`: Tạo version mới với tên là commit SHA
- `promote: true`: Tự động promote version mới thành default

**Bước 8: Get deployed URL**
```yaml
- name: Get deployed URL
  id: get-url
  run: |
    URL="https://${{ secrets.GCP_PROJECT_ID }}.appspot.com"
    echo "url=$URL" >> $GITHUB_OUTPUT
```
- Lấy URL ứng dụng đã deploy và lưu vào output

**Bước 9: Send email notification**
```yaml
- name: Send email notification
  if: always()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    ...
```
- Gửi email báo cáo kết quả deploy
- `if: always()`: Gửi dù deploy thành công hay thất bại

### 4. Job: report

```yaml
report:
  name: Generate Summary Report
  needs: [build-and-test, deploy]
  if: always()
```

**Giải thích:**
- Tạo báo cáo tổng hợp kết quả
- `if: always()`: Luôn chạy dù các job khác pass hay fail
- Hiển thị summary trong GitHub Actions UI

---

## Cấu Hình Google Cloud

### Bước 1: Tạo Project trên Google Cloud

1. Truy cập [Google Cloud Console](https://console.cloud.google.com/)
2. Tạo project mới hoặc chọn project có sẵn
3. Ghi nhớ **Project ID**

### Bước 2: Bật App Engine

1. Vào **App Engine** trong Console
2. Chọn region (ví dụ: `us-central`)
3. Bật App Engine API

### Bước 3: Tạo Service Account

1. Vào **IAM & Admin** > **Service Accounts**
2. Click **Create Service Account**
3. Đặt tên: `github-actions-deployer`
4. Cấp quyền:
   - **App Engine Deployer**
   - **Storage Admin** (để upload files)
   - **Service Account User**
5. Click **Create Key** > Chọn **JSON**
6. Tải file JSON về máy

### Bước 4: Cấu hình app.yaml

File `app.yaml` đã có sẵn trong project:

```yaml
runtime: nodejs18
instance_class: F1
env: standard

handlers:
  - url: /.*
    script: auto
```

**Giải thích:**
- `runtime: nodejs18`: Sử dụng Node.js 18
- `instance_class: F1`: Instance class nhỏ nhất (free tier)
- `env: standard`: Môi trường standard (không flexible)
- `handlers`: Xử lý tất cả requests

---

## Cấu Hình Secrets trong GitHub

### Các Secrets cần thiết:

1. **GCP_CREDENTIALS**
   - Nội dung file JSON Service Account (toàn bộ nội dung)
   - Vào: Repository Settings > Secrets and variables > Actions > New repository secret

2. **GCP_PROJECT_ID**
   - Project ID của Google Cloud (ví dụ: `my-project-123456`)

3. **EMAIL_USERNAME** (tùy chọn)
   - Email Gmail để gửi thông báo

4. **EMAIL_PASSWORD** (tùy chọn)
   - App Password của Gmail (không phải mật khẩu thường)
   - Tạo tại: Google Account > Security > 2-Step Verification > App passwords

5. **EMAIL_TO** (tùy chọn)
   - Email nhận thông báo

### Cách thêm Secrets:

1. Vào repository trên GitHub
2. Click **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret**
4. Nhập tên và giá trị
5. Click **Add secret**

---

## Kiểm Thử Tự Động với Jest

### Cấu hình Jest trong package.json

```json
{
  "jest": {
    "testEnvironment": "node",
    "coverageDirectory": "coverage",
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/**/*.test.js"
    ],
    "coverageReporters": ["text", "lcov", "html"]
  }
}
```

**Giải thích:**
- `testEnvironment: "node"`: Môi trường Node.js
- `coverageDirectory`: Thư mục chứa coverage reports
- `collectCoverageFrom`: Thu thập coverage từ các file trong `src/`
- `coverageReporters`: Định dạng báo cáo (text, lcov, html)

### Scripts trong package.json

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

- `npm test`: Chạy test một lần
- `npm run test:watch`: Chạy test ở chế độ watch (tự động chạy lại khi file thay đổi)
- `npm run test:coverage`: Chạy test và tạo coverage report

### Ví dụ Test File

File `tests/sum.test.js`:

```javascript
function sum(a, b) {
  return a + b;
}

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

---

## Triển Khai Ứng Dụng

### Quy trình tự động:

1. **Developer push code** lên GitHub
2. **GitHub Actions trigger** pipeline
3. **Build và Test** chạy trên nhiều phiên bản Node.js
4. **Nếu test pass** → Deploy lên Google App Engine
5. **Gửi email** thông báo kết quả

### Kiểm tra kết quả:

1. Vào tab **Actions** trong repository
2. Click vào workflow run mới nhất
3. Xem logs của từng job
4. Nếu deploy thành công, truy cập: `https://[PROJECT_ID].appspot.com`

---

## Xử Lý Lỗi và Debugging

### Các lỗi thường gặp:

#### 1. Lỗi Authentication
```
Error: Could not authenticate with Google Cloud
```
**Giải pháp:**
- Kiểm tra `GCP_CREDENTIALS` secret có đúng format JSON không
- Đảm bảo Service Account có đủ quyền

#### 2. Lỗi Test thất bại
```
Test suite failed to run
```
**Giải pháp:**
- Xem logs chi tiết trong GitHub Actions
- Chạy `npm test` local để kiểm tra
- Kiểm tra syntax của test files

#### 3. Lỗi Deploy
```
Error deploying to App Engine
```
**Giải pháp:**
- Kiểm tra `app.yaml` có đúng format không
- Đảm bảo App Engine đã được bật
- Kiểm tra billing account đã được kích hoạt

### Debugging Tips:

1. **Xem logs chi tiết**: Click vào từng step trong GitHub Actions
2. **Test local trước**: Chạy `npm test` và `npm run build` local
3. **Kiểm tra secrets**: Đảm bảo tất cả secrets đã được cấu hình
4. **Test từng bước**: Comment out các bước không cần thiết để test từng phần

---

## Tài Liệu Tham Khảo

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Google Cloud App Engine](https://cloud.google.com/appengine/docs)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [GitHub Actions for Google Cloud](https://github.com/google-github-actions)

---

## Kết Luận

Pipeline CI/CD này cung cấp:
- ✅ Kiểm thử tự động trên nhiều phiên bản Node.js
- ✅ Coverage reports
- ✅ Tự động deploy lên Google Cloud
- ✅ Email notifications
- ✅ Báo cáo tổng hợp

Với pipeline này, bạn có thể tự tin deploy code mà không lo về lỗi thủ công!

