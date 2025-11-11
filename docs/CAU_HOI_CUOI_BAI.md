# Trả Lời Câu Hỏi Bài Tập Cuối Bài

## 1. Làm thế nào để giải quyết xung đột giữa các nhánh một cách hiệu quả?

### Các bước giải quyết xung đột:

#### a. **Chuẩn bị trước khi merge:**
```bash
# 1. Cập nhật nhánh chính trước
git checkout main
git pull origin main

# 2. Chuyển về nhánh feature và merge/rebase từ main
git checkout feature-branch
git merge main  # hoặc git rebase main
```

#### b. **Xử lý xung đột:**

**Khi có xung đột, Git sẽ hiển thị:**
```
Auto-merging file.js
CONFLICT (content): Merge conflict in file.js
```

**Các bước xử lý:**

1. **Mở file có xung đột:**
   - Git sẽ đánh dấu các phần xung đột:
   ```javascript
   <<<<<<< HEAD
   // Code từ nhánh hiện tại
   const version = "1.0.0";
   =======
   // Code từ nhánh đang merge
   const version = "2.0.0";
   >>>>>>> feature-branch
   ```

2. **Giải quyết xung đột:**
   - Xóa các marker (`<<<<<<<`, `=======`, `>>>>>>>`)
   - Giữ lại code đúng hoặc kết hợp cả hai
   - Ví dụ sau khi giải quyết:
   ```javascript
   const version = "2.0.0"; // Giữ version mới hơn
   ```

3. **Sử dụng công cụ hỗ trợ:**
   ```bash
   # Mở merge tool
   git mergetool
   
   # Hoặc sử dụng VS Code
   code .
   # VS Code sẽ highlight các xung đột và cung cấp nút "Accept Current/Incoming/Both"
   ```

4. **Sau khi giải quyết:**
   ```bash
   # Đánh dấu file đã được giải quyết
   git add file.js
   
   # Hoàn tất merge
   git commit
   ```

#### c. **Best Practices:**

✅ **Thường xuyên sync với main:**
```bash
git fetch origin
git rebase origin/main  # Hoặc merge
```

✅ **Sử dụng git rerere (Reuse Recorded Resolution):**
```bash
# Bật rerere
git config rerere.enabled true

# Git sẽ nhớ cách bạn giải quyết xung đột và tự động áp dụng lại
```

✅ **Review code trước khi merge:**
- Sử dụng Pull Request để review
- Yêu cầu code review từ đồng nghiệp

✅ **Test sau khi giải quyết:**
```bash
npm test  # Đảm bảo không có lỗi sau khi merge
```

✅ **Giao tiếp với team:**
- Thông báo khi có xung đột phức tạp
- Thảo luận về cách giải quyết tốt nhất

---

## 2. Giải thích các bước cụ thể để thiết lập CI/CD với GitHub Actions

### Bước 1: Tạo Workflow File

Tạo file `.github/workflows/main.yml` trong repository:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

**Giải thích:**
- File này định nghĩa khi nào pipeline sẽ chạy
- `on: push`: Chạy khi có push code
- `on: pull_request`: Chạy khi có PR

### Bước 2: Định nghĩa Jobs

```yaml
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
```

**Giải thích:**
- `jobs`: Định nghĩa các công việc cần thực hiện
- `runs-on`: Chọn runner (ubuntu-latest, windows-latest, macos-latest)
- `steps`: Các bước thực hiện trong job

### Bước 3: Cấu hình Matrix Strategy (Chạy song song)

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
```

**Giải thích:**
- Chạy test trên nhiều phiên bản Node.js cùng lúc
- Đảm bảo ứng dụng tương thích với nhiều version

### Bước 4: Thêm bước Deploy

```yaml
deploy:
  needs: build-and-test
  if: github.ref == 'refs/heads/main'
  steps:
    - name: Deploy to Google Cloud
      uses: google-github-actions/deploy-appengine@v2
      with:
        credentials_json: ${{ secrets.GCP_CREDENTIALS }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
```

**Giải thích:**
- `needs`: Chỉ chạy khi job trước thành công
- `if`: Điều kiện chạy (chỉ deploy từ main)
- `secrets`: Sử dụng secrets để bảo mật thông tin

### Bước 5: Cấu hình Secrets

1. Vào **Repository Settings** > **Secrets and variables** > **Actions**
2. Click **New repository secret**
3. Thêm các secrets cần thiết:
   - `GCP_CREDENTIALS`: Service Account JSON
   - `GCP_PROJECT_ID`: Project ID

### Bước 6: Test Pipeline

1. Push code lên GitHub
2. Vào tab **Actions** để xem pipeline chạy
3. Kiểm tra logs nếu có lỗi

### Bước 7: Thêm Notifications (Tùy chọn)

```yaml
- name: Send email
  uses: dawidd6/action-send-mail@v3
  with:
    to: ${{ secrets.EMAIL_TO }}
    subject: "Deployment Status"
```

---

## 3. Lợi ích của việc sử dụng git rerere trong dự án lớn là gì?

### Git Rerere là gì?

**Rerere** = **Reuse Recorded Resolution** - Tái sử dụng cách giải quyết xung đột đã ghi nhớ

### Cách bật:

```bash
git config rerere.enabled true
# Hoặc cho toàn bộ hệ thống
git config --global rerere.enabled true
```

### Lợi ích trong dự án lớn:

#### 1. **Tiết kiệm thời gian:**
- Git tự động giải quyết xung đột tương tự mà bạn đã xử lý trước đó
- Không cần giải quyết lại cùng một xung đột nhiều lần

**Ví dụ:**
```bash
# Lần đầu: Bạn giải quyết xung đột trong file config.js
# Lần sau: Git tự động áp dụng cách giải quyết đó
```

#### 2. **Giảm lỗi con người:**
- Đảm bảo tính nhất quán trong cách giải quyết xung đột
- Tránh giải quyết sai do quên cách đã làm trước đó

#### 3. **Hữu ích với long-running branches:**
- Trong dự án lớn, thường có nhiều feature branches chạy song song
- Khi rebase/merge nhiều lần, rerere giúp tự động hóa

**Ví dụ workflow:**
```bash
# Feature branch tồn tại lâu, cần rebase nhiều lần
git checkout feature-branch
git rebase main  # Lần 1: Giải quyết xung đột thủ công
git rebase main  # Lần 2: Git tự động giải quyết (nhờ rerere)
```

#### 4. **Hỗ trợ team collaboration:**
- Mỗi developer có thể có rerere riêng
- Giảm thời gian giải quyết xung đột cho cả team

#### 5. **Tích hợp với CI/CD:**
- Có thể sử dụng trong automated testing
- Giảm số lần manual intervention

### Ví dụ thực tế:

```bash
# Scenario: Feature branch cần merge vào main nhiều lần

# Lần 1: Merge và giải quyết xung đột
git checkout feature-branch
git merge main
# Giải quyết xung đột trong utils.js
git add utils.js
git commit

# Lần 2: Merge lại (có xung đột tương tự)
git merge main
# Git tự động giải quyết nhờ rerere! ✅
```

### Lưu ý:

⚠️ **Rerere chỉ hoạt động với xung đột giống hệt nhau**
- Nếu code thay đổi, vẫn cần giải quyết thủ công

⚠️ **Cần review lại:**
- Dù rerere tự động giải quyết, vẫn nên review để đảm bảo đúng

---

## 4. Cách bạn xử lý lỗi trong pipeline khi một bước kiểm thử thất bại?

### Các chiến lược xử lý:

#### a. **Ngăn chặn Deploy (Fail Fast):**

```yaml
- name: Run tests
  run: npm test
  continue-on-error: false  # Dừng pipeline nếu test fail
```

**Kết quả:**
- Pipeline dừng ngay khi test thất bại
- Không deploy code có lỗi
- Developer nhận thông báo và sửa lỗi

#### b. **Thu thập thông tin lỗi:**

```yaml
- name: Run tests
  run: npm test
  continue-on-error: true  # Tiếp tục dù test fail

- name: Upload test results
  if: failure()  # Chỉ chạy khi test fail
  uses: actions/upload-artifact@v3
  with:
    name: test-results
    path: test-results/
```

**Lợi ích:**
- Lưu lại logs và artifacts để phân tích
- Không mất thông tin debug

#### c. **Gửi thông báo lỗi:**

```yaml
- name: Notify on failure
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    to: ${{ secrets.EMAIL_TO }}
    subject: "❌ Test Failed: ${{ github.repository }}"
    body: |
      Test suite failed!
      Branch: ${{ github.ref_name }}
      Commit: ${{ github.sha }}
      View logs: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

#### d. **Tạo Issue tự động:**

```yaml
- name: Create issue on test failure
  if: failure()
  uses: actions/github-script@v6
  with:
    script: |
      github.rest.issues.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        title: `Test failed in ${context.ref}`,
        body: `Tests failed in commit ${context.sha}`
      })
```

#### e. **Retry mechanism (Thử lại):**

```yaml
- name: Run tests with retry
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm test
```

**Áp dụng khi:**
- Test thất bại do lỗi tạm thời (network, timeout)
- Không phải lỗi code

#### f. **Conditional deployment:**

```yaml
deploy:
  needs: build-and-test
  if: success()  # Chỉ deploy khi test pass
  steps:
    - name: Deploy
      ...
```

### Workflow xử lý lỗi hoàn chỉnh:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      
      - name: Run tests
        id: test
        run: npm test
        continue-on-error: false
      
      - name: Upload logs on failure
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: test-logs
          path: |
            coverage/
            test-results/
      
      - name: Notify team
        if: failure()
        run: |
          echo "Tests failed! Notifying team..."
          # Gửi Slack/Email notification

  deploy:
    needs: test
    if: success()  # Chỉ chạy khi test pass
    steps:
      - name: Deploy
        ...
```

### Best Practices:

✅ **Fail fast**: Dừng ngay khi có lỗi nghiêm trọng
✅ **Collect artifacts**: Lưu logs để debug
✅ **Notify team**: Thông báo ngay khi có lỗi
✅ **Retry flaky tests**: Thử lại với test không ổn định
✅ **Separate concerns**: Tách test và deploy thành jobs riêng

---

## 5. So sánh giữa merge và rebase, khi nào nên sử dụng từng phương pháp?

### Merge vs Rebase

#### **Merge:**

```bash
git checkout main
git merge feature-branch
```

**Cách hoạt động:**
- Tạo một **merge commit** mới
- Giữ nguyên lịch sử commit của cả hai nhánh
- Tạo cấu trúc phân nhánh (branching) trong lịch sử

**Lịch sử sau merge:**
```
*   Merge commit (main)
|\
| * Commit C (feature-branch)
| * Commit B
* | Commit A (main)
|/
* Base commit
```

**Ưu điểm:**
- ✅ Giữ nguyên lịch sử gốc
- ✅ An toàn, không thay đổi commit history
- ✅ Dễ hiểu flow của dự án
- ✅ Phù hợp khi làm việc nhóm

**Nhược điểm:**
- ❌ Lịch sử phức tạp hơn (nhiều merge commits)
- ❌ Khó theo dõi timeline

#### **Rebase:**

```bash
git checkout feature-branch
git rebase main
```

**Cách hoạt động:**
- "Di chuyển" commits từ feature branch lên đầu main
- Tạo lại commits với hash mới
- Lịch sử tuyến tính (linear)

**Lịch sử sau rebase:**
```
* Commit C' (feature-branch, rebased)
* Commit B' (rebased)
* Commit A (main)
* Base commit
```

**Ưu điểm:**
- ✅ Lịch sử sạch, tuyến tính
- ✅ Dễ đọc và theo dõi
- ✅ Không có merge commits

**Nhược điểm:**
- ❌ Thay đổi commit history (hash mới)
- ❌ Có thể gây xung đột khi nhiều người cùng làm việc
- ❌ Mất thông tin về thời điểm thực tế merge

### Khi nào dùng Merge?

✅ **Khi làm việc nhóm:**
- Nhiều người cùng làm việc trên branch
- Cần giữ nguyên lịch sử để dễ debug

✅ **Khi merge vào main/master:**
- Main branch nên giữ lịch sử đầy đủ
- Dễ theo dõi các feature đã merge

✅ **Khi branch đã được push public:**
- Không nên rebase branch đã public
- Tránh làm rối lịch sử của người khác

**Ví dụ:**
```bash
# Feature branch đã được push và review
git checkout main
git merge feature-branch  # ✅ Dùng merge
```

### Khi nào dùng Rebase?

✅ **Khi làm việc local:**
- Branch chưa được push
- Muốn giữ lịch sử sạch trước khi push

✅ **Khi cập nhật feature branch:**
- Sync với main trước khi tạo PR
- Giữ PR sạch và dễ review

✅ **Khi muốn lịch sử tuyến tính:**
- Dự án nhỏ, ít người
- Muốn lịch sử dễ đọc

**Ví dụ:**
```bash
# Đang làm việc trên feature branch local
git checkout feature-branch
git rebase main  # ✅ Cập nhật từ main
git push --force-with-lease  # Push với force (cẩn thận!)
```

### So sánh trực quan:

| Tiêu chí | Merge | Rebase |
|----------|-------|--------|
| Lịch sử | Phân nhánh | Tuyến tính |
| Commit history | Giữ nguyên | Thay đổi |
| An toàn | ✅ Rất an toàn | ⚠️ Cần cẩn thận |
| Phù hợp nhóm | ✅ Có | ❌ Không |
| Dễ đọc | ❌ Phức tạp | ✅ Dễ đọc |
| Merge commits | Có | Không |

### Best Practice Workflow:

```bash
# 1. Tạo feature branch
git checkout -b feature/new-feature

# 2. Làm việc và commit
git add .
git commit -m "Add new feature"

# 3. Cập nhật từ main (dùng rebase)
git fetch origin
git rebase origin/main

# 4. Push và tạo PR
git push origin feature/new-feature

# 5. Sau khi review, merge vào main (dùng merge)
# (Trên GitHub: Click "Merge pull request")
```

### Golden Rule:

> **"Never rebase a public branch"**
> 
> - Chỉ rebase branch của chính bạn
> - Không rebase branch đã được share với người khác
> - Khi đã push, dùng merge

### Kết luận:

- **Merge**: An toàn, phù hợp nhóm, giữ nguyên lịch sử
- **Rebase**: Sạch, tuyến tính, nhưng cần cẩn thận

**Khuyến nghị:**
- Rebase cho feature branches local
- Merge cho main/master và branches đã public

---

## Tổng Kết

Các câu hỏi trên đã bao quát:
1. ✅ Giải quyết xung đột hiệu quả
2. ✅ Thiết lập CI/CD với GitHub Actions
3. ✅ Lợi ích của git rerere
4. ✅ Xử lý lỗi trong pipeline
5. ✅ So sánh merge vs rebase

Với kiến thức này, bạn đã sẵn sàng làm việc với Git và CI/CD trong môi trường thực tế!

