# Checklist Thiết Lập CI/CD

## ✅ Checklist Hoàn Thành

Sử dụng checklist này để đảm bảo bạn đã hoàn thành tất cả các bước cần thiết.

---

## 📋 Phần 1: Chuẩn Bị

### Google Cloud Setup
- [ ] Đã tạo project trên Google Cloud Console
- [ ] Đã ghi nhớ Project ID
- [ ] Đã bật App Engine
- [ ] Đã chọn region cho App Engine
- [ ] Đã tạo Service Account
- [ ] Đã cấp quyền cho Service Account:
  - [ ] App Engine Deployer
  - [ ] Storage Admin
  - [ ] Service Account User
- [ ] Đã tải file JSON Service Account Key
- [ ] Đã lưu file JSON an toàn (không commit vào Git)

### GitHub Setup
- [ ] Đã có repository trên GitHub
- [ ] Đã clone repository về máy local
- [ ] Đã cấu hình Git user name và email

### Local Setup
- [ ] Đã cài đặt Node.js (16, 18, hoặc 20)
- [ ] Đã chạy `npm install`
- [ ] Đã test ứng dụng local (`npm start`)
- [ ] Đã chạy tests (`npm test`)

---

## 📋 Phần 2: Cấu Hình GitHub Secrets

Vào **Repository Settings** > **Secrets and variables** > **Actions**

- [ ] Đã thêm secret `GCP_CREDENTIALS` (toàn bộ nội dung file JSON)
- [ ] Đã thêm secret `GCP_PROJECT_ID` (Project ID của Google Cloud)
- [ ] (Tùy chọn) Đã thêm secret `EMAIL_USERNAME`
- [ ] (Tùy chọn) Đã thêm secret `EMAIL_PASSWORD` (App Password)
- [ ] (Tùy chọn) Đã thêm secret `EMAIL_TO`

---

## 📋 Phần 3: Kiểm Tra Files

### Workflow File
- [ ] File `.github/workflows/main.yml` đã tồn tại
- [ ] Workflow đã được cấu hình đúng
- [ ] Đã kiểm tra syntax YAML

### App Configuration
- [ ] File `app.yaml` đã tồn tại
- [ ] Runtime trong `app.yaml` khớp với Node.js version
- [ ] File `package.json` có đầy đủ scripts

### Test Files
- [ ] Có ít nhất một test file trong `tests/`
- [ ] Tests chạy thành công local (`npm test`)

---

## 📋 Phần 4: Test Pipeline

### Push Code
- [ ] Đã commit tất cả thay đổi
- [ ] Đã push code lên GitHub
- [ ] Đã push vào nhánh `main` hoặc `development`

### Kiểm Tra Pipeline
- [ ] Đã vào tab **Actions** trên GitHub
- [ ] Workflow đã được trigger
- [ ] Job `build-and-test` đang chạy
- [ ] Tests đã pass trên tất cả Node.js versions (16, 18, 20)
- [ ] Coverage reports đã được tạo
- [ ] Job `deploy` đã chạy (nếu push vào main)
- [ ] Deploy đã thành công

### Kiểm Tra Deployment
- [ ] Đã truy cập URL: `https://[PROJECT-ID].appspot.com`
- [ ] Ứng dụng đang chạy và phản hồi
- [ ] (Nếu có) Đã nhận email thông báo

---

## 📋 Phần 5: Tài Liệu

- [ ] Đã đọc [Hướng Dẫn CI/CD Chi Tiết](./HUONG_DAN_CI_CD.md)
- [ ] Đã đọc [Cấu Hình Google Cloud](./CAU_HINH_GOOGLE_CLOUD.md)
- [ ] Đã đọc [Trả Lời Câu Hỏi Cuối Bài](./CAU_HOI_CUOI_BAI.md)
- [ ] Đã hiểu cách pipeline hoạt động
- [ ] Đã biết cách xử lý lỗi

---

## 📋 Phần 6: Best Practices

### Git
- [ ] Đã hiểu cách giải quyết xung đột
- [ ] Đã biết khi nào dùng merge vs rebase
- [ ] Đã cấu hình git rerere (nếu muốn)

### CI/CD
- [ ] Đã hiểu cách pipeline xử lý lỗi
- [ ] Đã biết cách xem logs và debug
- [ ] Đã test với Pull Request

### Security
- [ ] Đã đảm bảo không commit secrets vào Git
- [ ] Đã sử dụng `.gitignore` đúng cách
- [ ] Đã review quyền của Service Account

---

## 🎯 Kết Quả Mong Đợi

Sau khi hoàn thành tất cả các bước trên, bạn sẽ có:

✅ Pipeline CI/CD hoạt động tự động
✅ Tests chạy trên nhiều phiên bản Node.js
✅ Coverage reports được tạo tự động
✅ Ứng dụng tự động deploy lên Google Cloud
✅ Email notifications khi deploy
✅ Hiểu rõ cách hoạt động của CI/CD

---

## 🆘 Nếu Gặp Vấn Đề

1. **Pipeline thất bại:**
   - Xem logs trong GitHub Actions
   - Kiểm tra secrets đã đúng chưa
   - Xem phần "Xử Lý Lỗi" trong [Hướng Dẫn CI/CD](./HUONG_DAN_CI_CD.md)

2. **Deploy thất bại:**
   - Kiểm tra Service Account có đủ quyền
   - Kiểm tra App Engine đã được bật
   - Xem [Cấu Hình Google Cloud](./CAU_HINH_GOOGLE_CLOUD.md)

3. **Tests thất bại:**
   - Chạy `npm test` local để kiểm tra
   - Xem logs chi tiết trong GitHub Actions
   - Kiểm tra syntax của test files

---

## 📝 Ghi Chú

- Đánh dấu ✅ khi hoàn thành mỗi bước
- Nếu gặp vấn đề, ghi chú lại để tham khảo sau
- Tham khảo các file tài liệu trong thư mục `docs/` khi cần

---

**Chúc bạn thành công! 🚀**

