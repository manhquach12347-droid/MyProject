# MyProject - CI/CD với GitHub Actions và Google Cloud

Dự án mẫu minh họa cách thiết lập pipeline CI/CD hoàn chỉnh với GitHub Actions và triển khai tự động lên Google Cloud App Engine.

## 🚀 Tính Năng

- ✅ **CI/CD Pipeline** với GitHub Actions
- ✅ **Kiểm thử tự động** với Jest (chạy trên Node.js 16, 18, 20)
- ✅ **Test Coverage Reports** tự động
- ✅ **Tự động deploy** lên Google Cloud App Engine
- ✅ **Email Notifications** khi deploy thành công/thất bại
- ✅ **Matrix Strategy** - Chạy song song nhiều môi trường

## 📋 Yêu Cầu

- Node.js 16+ (hoặc 18, 20)
- GitHub account
- Google Cloud account với App Engine đã bật
- Repository trên GitHub

## 🏗️ Cấu Trúc Project

```
MyProject/
├── .github/
│   └── workflows/
│       └── main.yml          # Workflow CI/CD
├── docs/
│   ├── README.md             # Tài liệu tổng hợp
│   ├── HUONG_DAN_CI_CD.md    # Hướng dẫn chi tiết CI/CD
│   ├── CAU_HOI_CUOI_BAI.md   # Trả lời câu hỏi cuối bài
│   └── CAU_HINH_GOOGLE_CLOUD.md  # Cấu hình Google Cloud
├── src/
│   └── app.js                # Ứng dụng Node.js
├── tests/
│   └── sum.test.js           # Test file
├── app.yaml                  # Cấu hình App Engine
└── package.json              # Dependencies và scripts
```

## 🚦 Bắt Đầu

### 1. Clone Repository

```bash
git clone https://github.com/your-username/MyProject.git
cd MyProject
```

### 2. Cài Đặt Dependencies

```bash
npm install
```

### 3. Chạy Tests

```bash
npm test
```

### 4. Chạy Tests với Coverage

```bash
npm run test:coverage
```

### 5. Chạy Ứng Dụng Local

```bash
npm start
```

Ứng dụng sẽ chạy tại: `http://localhost:8080`

## 📚 Tài Liệu

Xem thư mục [`docs/`](./docs/) để biết hướng dẫn chi tiết:

- **[Hướng Dẫn CI/CD Chi Tiết](./docs/HUONG_DAN_CI_CD.md)** - Giải thích đầy đủ về pipeline và cách hoạt động
- **[Cấu Hình Google Cloud](./docs/CAU_HINH_GOOGLE_CLOUD.md)** - Hướng dẫn từng bước cấu hình Google Cloud
- **[Trả Lời Câu Hỏi Cuối Bài](./docs/CAU_HOI_CUOI_BAI.md)** - Giải đáp các câu hỏi về Git, CI/CD, và best practices

## ⚙️ Cấu Hình CI/CD

### 1. Cấu Hình Google Cloud

Làm theo [Hướng Dẫn Cấu Hình Google Cloud](./docs/CAU_HINH_GOOGLE_CLOUD.md) để:
- Tạo project trên Google Cloud
- Bật App Engine
- Tạo Service Account
- Tải Service Account Key

### 2. Thêm Secrets vào GitHub

Vào **Repository Settings** > **Secrets and variables** > **Actions**, thêm:

| Secret | Mô tả |
|--------|-------|
| `GCP_CREDENTIALS` | Toàn bộ nội dung file JSON Service Account |
| `GCP_PROJECT_ID` | Project ID của Google Cloud |
| `EMAIL_USERNAME` | Email Gmail (tùy chọn) |
| `EMAIL_PASSWORD` | App Password của Gmail (tùy chọn) |
| `EMAIL_TO` | Email nhận thông báo (tùy chọn) |

### 3. Push Code

```bash
git add .
git commit -m "Setup CI/CD pipeline"
git push origin main
```

Pipeline sẽ tự động chạy khi bạn push code lên nhánh `main` hoặc `development`.

## 🔄 Workflow Pipeline

Pipeline bao gồm 3 jobs chính:

1. **build-and-test**: 
   - Chạy test trên Node.js 16, 18, 20 (song song)
   - Tạo coverage reports
   - Build ứng dụng

2. **deploy**:
   - Chỉ chạy khi test pass
   - Deploy lên Google App Engine
   - Gửi email thông báo

3. **report**:
   - Tạo báo cáo tổng hợp
   - Hiển thị trong GitHub Actions UI

## 📊 Test Coverage

Pipeline tự động tạo coverage reports và upload lên artifacts. Xem coverage reports trong:
- GitHub Actions artifacts
- Thư mục `coverage/` sau khi chạy `npm run test:coverage`

## 🐛 Xử Lý Lỗi

Nếu pipeline thất bại:

1. Xem logs chi tiết trong tab **Actions** trên GitHub
2. Kiểm tra các secrets đã được cấu hình đúng chưa
3. Xem phần "Xử Lý Lỗi" trong [Hướng Dẫn CI/CD](./docs/HUONG_DAN_CI_CD.md)

## 📝 Scripts

```json
{
  "start": "node src/app.js",
  "test": "jest",
  "test:watch": "jest --watch",
  "test:coverage": "jest --coverage",
  "build": "echo 'Build completed successfully'"
}
```

## 🔗 Liên Kết Hữu Ích

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Google Cloud App Engine](https://cloud.google.com/appengine/docs)
- [Jest Documentation](https://jestjs.io/)
- [Git Documentation](https://git-scm.com/doc)

## 📄 License

Xem file [LICENSE](./LICENSE) để biết thêm chi tiết.

## 👥 Đóng Góp

Mọi đóng góp đều được chào đón! Vui lòng tạo Pull Request hoặc Issue.

---

**Lưu ý**: Đây là dự án mẫu cho mục đích học tập. Đảm bảo cấu hình đúng các secrets và quyền truy cập trước khi sử dụng trong môi trường production.

