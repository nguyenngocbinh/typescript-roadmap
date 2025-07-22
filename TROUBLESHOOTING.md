# GitHub Pages Troubleshooting

## Lỗi 404 - Nguyên nhân và cách khắc phục

### 🔍 Nguyên nhân thường gặp:

1. **GitHub Pages chưa được kích hoạt**
   - Vào Settings > Pages trong repository
   - Chọn Source: Deploy from a branch
   - Chọn Branch: main, Folder: / (root)

2. **Thời gian deploy**
   - GitHub Pages cần 5-10 phút để build và deploy
   - Kiểm tra tab Actions để xem trạng thái deployment

3. **Cấu hình Jekyll**
   - File `_config.yml` phải có đúng `baseurl: "/typescript-roadmap"`
   - Tất cả markdown files phải có frontmatter với `layout: default`

### 🚀 Cách kiểm tra:

1. **Kiểm tra GitHub Actions**: https://github.com/nguyenngocbinh/typescript-roadmap/actions
2. **Kiểm tra Pages Settings**: https://github.com/nguyenngocbinh/typescript-roadmap/settings/pages
3. **Đợi deployment hoàn thành** (có dấu ✅ xanh)

### 📱 Khi website hoạt động:

Website sẽ có sẵn:
- **Desktop và mobile responsive**
- **Dark mode tự động** (theo system preference)
- **Navigation menu** dễ sử dụng
- **Search functionality** (có thể thêm sau)

---

*Website sẽ sớm hoạt động tại: https://nguyenngocbinh.github.io/typescript-roadmap/*
