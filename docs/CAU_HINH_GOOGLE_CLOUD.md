# Hướng Dẫn Cấu Hình Google Cloud cho CI/CD

## Mục Lục
1. [Tạo Project trên Google Cloud](#1-tạo-project-trên-google-cloud)
2. [Bật App Engine](#2-bật-app-engine)
3. [Tạo Service Account](#3-tạo-service-account)
4. [Cấp Quyền cho Service Account](#4-cấp-quyền-cho-service-account)
5. [Tạo và Tải Service Account Key](#5-tạo-và-tải-service-account-key)
6. [Cấu hình app.yaml](#6-cấu-hình-appyaml)
7. [Thêm Secrets vào GitHub](#7-thêm-secrets-vào-github)
8. [Test Deploy](#8-test-deploy)

---

## 1. Tạo Project trên Google Cloud

### Bước 1: Truy cập Google Cloud Console

1. Mở trình duyệt và truy cập: https://console.cloud.google.com/
2. Đăng nhập bằng tài khoản Google của bạn

### Bước 2: Tạo Project mới

1. Click vào dropdown **Project** ở đầu trang (góc trên bên trái)
2. Click **New Project**
3. Điền thông tin:
   - **Project name**: `myproject-cicd` (hoặc tên bạn muốn)
   - **Organization**: Chọn organization (nếu có)
   - **Location**: Chọn location (nếu có)
4. Click **Create**
5. Đợi vài giây để project được tạo

### Bước 3: Ghi nhớ Project ID

- Project ID sẽ hiển thị sau khi tạo (ví dụ: `myproject-cicd-123456`)
- **Lưu ý**: Project ID khác với Project Name
- Bạn sẽ cần Project ID này để cấu hình trong GitHub Secrets

---

## 2. Bật App Engine

### Bước 1: Mở App Engine

1. Trong Google Cloud Console, vào menu **☰** (góc trên bên trái)
2. Tìm và click **App Engine**
3. Nếu chưa bật, bạn sẽ thấy trang "Get started with App Engine"

### Bước 2: Chọn Region

1. Chọn **Region** cho ứng dụng:
   - **us-central** (Iowa) - Khuyến nghị cho Bắc Mỹ
   - **us-east** (South Carolina)
   - **europe-west** (Belgium)
   - **asia-northeast** (Tokyo) - Khuyến nghị cho châu Á
2. Click **Create Application**

### Bước 3: Chọn Environment

- Chọn **Standard environment** (khuyến nghị cho Node.js)
- Click **Next**

### Bước 4: Hoàn tất

- App Engine sẽ được bật trong vài phút
- Bạn sẽ thấy dashboard của App Engine

---

## 3. Tạo Service Account

### Bước 1: Mở IAM & Admin

1. Vào menu **☰** > **IAM & Admin** > **Service Accounts**
2. Hoặc truy cập trực tiếp: https://console.cloud.google.com/iam-admin/serviceaccounts

### Bước 2: Tạo Service Account mới

1. Click **+ CREATE SERVICE ACCOUNT** (hoặc **Create Service Account**)
2. Điền thông tin:
   - **Service account name**: `github-actions-deployer`
   - **Service account ID**: Sẽ tự động tạo (có thể giữ nguyên)
   - **Description**: `Service account for GitHub Actions CI/CD deployment`
3. Click **CREATE AND CONTINUE**

### Bước 3: Ghi nhớ Service Account Email

- Service Account sẽ có email dạng: `github-actions-deployer@[PROJECT-ID].iam.gserviceaccount.com`
- Email này sẽ được dùng để cấp quyền

---

## 4. Cấp Quyền cho Service Account

### Bước 1: Cấp quyền cơ bản

Trong bước "Grant this service account access to project":

1. Tìm và chọn các role sau:
   - **App Engine Deployer** - Quyền deploy lên App Engine
   - **Storage Admin** - Quyền upload files lên Cloud Storage (cần cho deploy)
   - **Service Account User** - Quyền sử dụng service account

2. Click **CONTINUE**

### Bước 2: Hoàn tất

1. Bước "Grant users access" có thể bỏ qua
2. Click **DONE**

### Danh sách quyền cần thiết:

| Role | Mục đích |
|------|----------|
| **App Engine Deployer** | Deploy ứng dụng lên App Engine |
| **Storage Admin** | Upload files lên Cloud Storage (cần cho deploy process) |
| **Service Account User** | Sử dụng service account |

**Lưu ý**: Nếu thiếu quyền, deploy sẽ thất bại với lỗi "Permission denied"

---

## 5. Tạo và Tải Service Account Key

### Bước 1: Mở Service Account

1. Trong danh sách Service Accounts, click vào service account vừa tạo
2. Vào tab **KEYS**

### Bước 2: Tạo Key mới

1. Click **ADD KEY** > **Create new key**
2. Chọn **JSON** (không chọn P12)
3. Click **CREATE**

### Bước 3: Tải file JSON

- File JSON sẽ tự động tải về máy
- Tên file: `[PROJECT-ID]-[RANDOM].json`
- **Lưu ý**: Giữ file này an toàn, không commit vào Git!

### Bước 4: Xem nội dung file JSON

Mở file JSON, bạn sẽ thấy cấu trúc như sau:

```json
{
  "type": "service_account",
  "project_id": "myproject-cicd-123456",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "github-actions-deployer@myproject-cicd-123456.iam.gserviceaccount.com",
  "client_id": "...",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "..."
}
```

**Toàn bộ nội dung file này** sẽ được dùng làm `GCP_CREDENTIALS` secret trong GitHub.

---

## 6. Cấu hình app.yaml

### File app.yaml hiện tại:

```yaml
runtime: nodejs18
instance_class: F1
env: standard

handlers:
  - url: /.*
    script: auto
```

### Giải thích từng dòng:

#### `runtime: nodejs18`
- Chỉ định phiên bản Node.js
- Có thể thay đổi: `nodejs16`, `nodejs18`, `nodejs20`
- Phải khớp với version trong `package.json`

#### `instance_class: F1`
- Loại instance (tài nguyên máy chủ)
- **F1**: Free tier, phù hợp cho ứng dụng nhỏ
- Các option khác: `F2`, `F4`, `F4_1G` (tốn phí)

#### `env: standard`
- Môi trường App Engine
- **standard**: Môi trường tiêu chuẩn (nhanh, rẻ)
- **flexible**: Môi trường linh hoạt (nhiều tùy chọn hơn, đắt hơn)

#### `handlers:`
- Định nghĩa cách xử lý requests
- `url: /.*`: Xử lý tất cả URLs
- `script: auto`: Tự động detect entry point từ `package.json`

### Cấu hình nâng cao (Tùy chọn):

```yaml
runtime: nodejs18
instance_class: F1
env: standard

# Tự động scale
automatic_scaling:
  min_instances: 0
  max_instances: 10
  target_cpu_utilization: 0.6

# Environment variables
env_variables:
  NODE_ENV: production
  PORT: 8080

# Handlers
handlers:
  - url: /.*
    script: auto
    secure: always  # Chỉ cho phép HTTPS
```

---

## 7. Thêm Secrets vào GitHub

### Bước 1: Mở Repository Settings

1. Vào repository trên GitHub
2. Click **Settings** (tab trên cùng)
3. Trong menu bên trái, click **Secrets and variables** > **Actions**

### Bước 2: Thêm Secret GCP_CREDENTIALS

1. Click **New repository secret**
2. **Name**: `GCP_CREDENTIALS`
3. **Secret**: 
   - Mở file JSON đã tải về
   - Copy **toàn bộ nội dung** (từ `{` đến `}`)
   - Paste vào ô Secret
4. Click **Add secret**

**Lưu ý**: 
- Phải copy toàn bộ file JSON, không chỉ một phần
- Không có khoảng trắng thừa ở đầu/cuối

### Bước 3: Thêm Secret GCP_PROJECT_ID

1. Click **New repository secret** lần nữa
2. **Name**: `GCP_PROJECT_ID`
3. **Secret**: Project ID của bạn (ví dụ: `myproject-cicd-123456`)
4. Click **Add secret**

### Bước 4: Thêm Secrets Email (Tùy chọn)

Nếu muốn nhận email thông báo:

1. **EMAIL_USERNAME**: Email Gmail của bạn
2. **EMAIL_PASSWORD**: App Password (xem hướng dẫn bên dưới)
3. **EMAIL_TO**: Email nhận thông báo

#### Tạo App Password cho Gmail:

1. Vào https://myaccount.google.com/
2. **Security** > **2-Step Verification** (phải bật trước)
3. Cuối trang, click **App passwords**
4. Chọn app: **Mail**
5. Chọn device: **Other (Custom name)**
6. Nhập tên: `GitHub Actions`
7. Click **Generate**
8. Copy password (16 ký tự, không có khoảng trắng)
9. Dùng password này làm `EMAIL_PASSWORD`

### Danh sách Secrets cần thiết:

| Secret Name | Mô tả | Bắt buộc |
|-------------|-------|----------|
| `GCP_CREDENTIALS` | Toàn bộ nội dung file JSON Service Account | ✅ Có |
| `GCP_PROJECT_ID` | Project ID của Google Cloud | ✅ Có |
| `EMAIL_USERNAME` | Email Gmail để gửi thông báo | ❌ Không |
| `EMAIL_PASSWORD` | App Password của Gmail | ❌ Không |
| `EMAIL_TO` | Email nhận thông báo | ❌ Không |

---

## 8. Test Deploy

### Bước 1: Kiểm tra Workflow File

Đảm bảo file `.github/workflows/main.yml` đã được cấu hình đúng:

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}

- name: Deploy to Google App Engine
  uses: google-github-actions/deploy-appengine@v2
  with:
    project_id: ${{ secrets.GCP_PROJECT_ID }}
    deliverables: app.yaml
```

### Bước 2: Push code lên GitHub

```bash
git add .
git commit -m "Setup CI/CD with Google Cloud"
git push origin main
```

### Bước 3: Kiểm tra GitHub Actions

1. Vào repository trên GitHub
2. Click tab **Actions**
3. Bạn sẽ thấy workflow đang chạy
4. Click vào workflow run để xem chi tiết

### Bước 4: Xem Logs

1. Click vào job **Deploy to Google App Engine**
2. Xem logs từng bước
3. Nếu có lỗi, logs sẽ hiển thị chi tiết

### Bước 5: Kiểm tra App đã Deploy

1. Sau khi deploy thành công, truy cập:
   ```
   https://[PROJECT-ID].appspot.com
   ```
2. Bạn sẽ thấy ứng dụng Node.js đang chạy

### Các lỗi thường gặp:

#### Lỗi 1: Authentication failed
```
Error: Could not authenticate with Google Cloud
```
**Giải pháp:**
- Kiểm tra `GCP_CREDENTIALS` có đúng format JSON không
- Đảm bảo copy toàn bộ nội dung file JSON

#### Lỗi 2: Permission denied
```
Error: Permission denied
```
**Giải pháp:**
- Kiểm tra Service Account có đủ quyền:
  - App Engine Deployer
  - Storage Admin
  - Service Account User

#### Lỗi 3: Project not found
```
Error: Project [PROJECT-ID] not found
```
**Giải pháp:**
- Kiểm tra `GCP_PROJECT_ID` có đúng không
- Đảm bảo project đã được tạo và bật billing (nếu cần)

#### Lỗi 4: App Engine not enabled
```
Error: App Engine application not found
```
**Giải pháp:**
- Đảm bảo App Engine đã được bật trong project
- Chọn đúng region khi bật App Engine

---

## Checklist Hoàn Thành

Trước khi deploy, đảm bảo:

- [ ] Đã tạo project trên Google Cloud
- [ ] Đã bật App Engine
- [ ] Đã tạo Service Account
- [ ] Đã cấp đủ quyền cho Service Account
- [ ] Đã tải file JSON key
- [ ] Đã thêm `GCP_CREDENTIALS` vào GitHub Secrets
- [ ] Đã thêm `GCP_PROJECT_ID` vào GitHub Secrets
- [ ] File `app.yaml` đã được cấu hình đúng
- [ ] Workflow file đã được cấu hình đúng
- [ ] Đã push code lên GitHub

---

## Tài Liệu Tham Khảo

- [Google Cloud App Engine Documentation](https://cloud.google.com/appengine/docs)
- [GitHub Actions for Google Cloud](https://github.com/google-github-actions)
- [App Engine Node.js Runtime](https://cloud.google.com/appengine/docs/standard/nodejs/runtime)
- [Service Accounts Best Practices](https://cloud.google.com/iam/docs/best-practices-service-accounts)

---

## Kết Luận

Sau khi hoàn thành các bước trên, bạn đã sẵn sàng:
- ✅ Deploy ứng dụng lên Google Cloud App Engine
- ✅ Tự động hóa quy trình CI/CD
- ✅ Nhận thông báo qua email khi deploy

Chúc bạn thành công! 🚀

