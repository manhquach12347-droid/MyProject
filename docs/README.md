# Tài Liệu Hướng Dẫn CI/CD với GitHub Actions và Google Cloud

## Mục Lục

1. [Hướng Dẫn CI/CD Chi Tiết](./HUONG_DAN_CI_CD.md)
   - Tổng quan về CI/CD
   - Giải thích chi tiết workflow
   - Cấu hình và thiết lập
   - Xử lý lỗi và debugging

2. [Trả Lời Câu Hỏi Cuối Bài](./CAU_HOI_CUOI_BAI.md)
   - Giải quyết xung đột Git
   - Thiết lập CI/CD với GitHub Actions
   - Git rerere và lợi ích
   - Xử lý lỗi trong pipeline
   - So sánh merge vs rebase

3. [Cấu Hình Google Cloud](./CAU_HINH_GOOGLE_CLOUD.md)
   - Tạo project trên Google Cloud
   - Bật App Engine
   - Tạo Service Account
   - Cấu hình secrets
   - Test deploy

---

## Tổng Quan Dự Án

Dự án này minh họa cách thiết lập pipeline CI/CD hoàn chỉnh với:

- ✅ **GitHub Actions**: Tự động build, test, và deploy
- ✅ **Jest**: Kiểm thử tự động với coverage reports
- ✅ **Google Cloud App Engine**: Triển khai ứng dụng
- ✅ **Matrix Strategy**: Chạy test trên nhiều phiên bản Node.js
- ✅ **Email Notifications**: Thông báo kết quả deploy

---

## Cấu Trúc Project

```
MyProject/
├── .github/
│   └── workflows/
│       └── main.yml          # Workflow CI/CD
├── docs/
│   ├── README.md             # File này
│   ├── HUONG_DAN_CI_CD.md    # Hướng dẫn chi tiết CI/CD
│   ├── CAU_HOI_CUOI_BAI.md   # Trả lời câu hỏi
│   └── CAU_HINH_GOOGLE_CLOUD.md  # Cấu hình GCP
├── src/
│   └── app.js                # Ứng dụng Node.js
├── tests/
│   └── sum.test.js           # Test file
├── app.yaml                  # Cấu hình App Engine
└── package.json              # Dependencies và scripts
```

---

## Bắt Đầu Nhanh

### 1. Đọc Hướng Dẫn

Bắt đầu với [Hướng Dẫn CI/CD Chi Tiết](./HUONG_DAN_CI_CD.md) để hiểu cách pipeline hoạt động.

### 2. Cấu Hình Google Cloud

Làm theo [Hướng Dẫn Cấu Hình Google Cloud](./CAU_HINH_GOOGLE_CLOUD.md) để thiết lập môi trường deploy.

### 3. Thiết Lập Secrets

Thêm các secrets cần thiết vào GitHub:
- `GCP_CREDENTIALS`
- `GCP_PROJECT_ID`
- `EMAIL_USERNAME` (tùy chọn)
- `EMAIL_PASSWORD` (tùy chọn)
- `EMAIL_TO` (tùy chọn)

### 4. Push Code

```bash
git add .
git commit -m "Setup CI/CD pipeline"
git push origin main
```

### 5. Kiểm Tra Pipeline

Vào tab **Actions** trên GitHub để xem pipeline chạy.

---

## Tính Năng Chính

### 1. Matrix Strategy

Pipeline chạy test trên nhiều phiên bản Node.js (16, 18, 20) song song:

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
```

### 2. Test Coverage

Tự động tạo coverage reports và upload lên artifacts:

```yaml
- name: Generate test coverage
  run: npm run test:coverage
```

### 3. Automatic Deployment

Tự động deploy lên Google App Engine khi test pass:

```yaml
deploy:
  needs: build-and-test
  if: github.ref == 'refs/heads/main'
```

### 4. Email Notifications

Gửi email thông báo kết quả deploy:

```yaml
- name: Send email notification
  uses: dawidd6/action-send-mail@v3
```

---

## Yêu Cầu

- Node.js 16+ (hoặc 18, 20)
- GitHub account
- Google Cloud account
- Repository trên GitHub

---

## Tài Liệu Tham Khảo

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Google Cloud App Engine](https://cloud.google.com/appengine/docs)
- [Jest Documentation](https://jestjs.io/)
- [Git Documentation](https://git-scm.com/doc)

---

## Hỗ Trợ

Nếu gặp vấn đề, hãy:
1. Kiểm tra logs trong GitHub Actions
2. Xem phần "Xử Lý Lỗi" trong [Hướng Dẫn CI/CD](./HUONG_DAN_CI_CD.md)
3. Kiểm tra [Cấu Hình Google Cloud](./CAU_HINH_GOOGLE_CLOUD.md) để đảm bảo đã cấu hình đúng

---

## License

Xem file [LICENSE](../LICENSE) trong thư mục gốc.

