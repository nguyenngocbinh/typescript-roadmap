# 🚀 Hướng dẫn Deploy lên GitHub Pages

## 📋 Các bước thiết lập

### 1. Tạo Repository trên GitHub
1. Đăng nhập vào GitHub
2. Tạo repository mới với tên `typescript-roadmap` (hoặc tên khác)
3. Set visibility là **Public** (GitHub Pages free chỉ support public repos)

### 2. Upload code lên GitHub
```bash
# Di chuyển vào thư mục project
cd typescript-roadmap

# Initialize git repository
git init

# Add tất cả files
git add .

# Commit đầu tiên
git commit -m "Initial commit: TypeScript roadmap website"

# Add remote origin (thay YOUR_USERNAME bằng GitHub username của bạn)
git remote add origin https://github.com/nguyenngocbinh/typescript-roadmap.git

# Push lên GitHub
git push -u origin main
```

### 3. Thiết lập GitHub Pages
1. Vào repository trên GitHub
2. Click tab **Settings**
3. Scroll xuống phần **Pages** (bên trái sidebar)
4. Trong **Source**, chọn **GitHub Actions**
5. GitHub sẽ tự động detect và chạy workflow

### 4. Kiểm tra deployment
- Sau vài phút, website sẽ live tại: `https://nguyenngocbinh.github.io/typescript-roadmap`
- Check tab **Actions** để xem trạng thái deployment
- Mỗi lần push code mới, website sẽ tự động update

## 🔧 Tùy chỉnh website

### Thay đổi theme Jekyll
Trong file `_config.yml`, thay đổi:
```yaml
theme: minima  # Có thể đổi thành cayman, slate, architect, etc.
```

### Thêm Google Analytics
Trong `_config.yml`, thêm:
```yaml
google_analytics: YOUR_TRACKING_ID
```

### Custom domain
1. Tạo file `CNAME` với nội dung domain của bạn:
```
your-domain.com
```
2. Trong GitHub Settings > Pages, add custom domain

### Tùy chỉnh CSS
Tạo file `assets/css/style.scss`:
```scss
---
---

@import "{{ site.theme }}";

// Custom styles
.highlight {
  background-color: #f8f9fa;
  border-radius: 6px;
  padding: 1rem;
}

// Dark mode support
@media (prefers-color-scheme: dark) {
  body {
    background-color: #0d1117;
    color: #e6edf3;
  }
}
```

## 📱 Alternative: Sử dụng Docsify

Nếu muốn giao diện đẹp hơn và interactive, có thể dùng Docsify:

### 1. Tạo index.html
```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>TypeScript Roadmap</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Lộ trình học TypeScript cho người biết Python">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'TypeScript Roadmap',
      repo: 'https://github.com/nguyenngocbinh/typescript-roadmap',
      loadSidebar: true,
      loadNavbar: true,
      subMaxLevel: 2,
      search: 'auto',
      copyCode: {
        buttonText: 'Copy',
        errorText: 'Error',
        successText: 'Copied'
      }
    }
  </script>
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <!-- Search plugin -->
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <!-- Copy code plugin -->
  <script src="//cdn.jsdelivr.net/npm/docsify-copy-code@2"></script>
  <!-- Syntax highlighting -->
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-typescript.min.js"></script>
</body>
</html>
```

### 2. Tạo _sidebar.md
```markdown
<!-- docs/_sidebar.md -->

* [🏠 Trang chủ](/)
* [📖 Giới thiệu](index.md)

* **Lộ trình chính**
  * [🗓️ Tuần 1: Cơ bản](week-1.md)
  * [🗓️ Tuần 2: Thực hành](week-2.md)
  * [🗓️ Tuần 3: Nâng cao](week-3.md)
  * [🗓️ Tuần 4: Ứng dụng](week-4.md)

* [🧰 Bonus](bonus.md)
* [📋 Navigation](navigation.md)
```

## 🔍 SEO và Analytics

### Meta tags trong Jekyll
Tạo file `_includes/head.html`:
```html
<meta property="og:title" content="{{ page.title | default: site.title }}">
<meta property="og:description" content="{{ page.description | default: site.description }}">
<meta property="og:image" content="{{ '/assets/images/preview.png' | absolute_url }}">
<meta property="og:url" content="{{ page.url | absolute_url }}">
<meta name="twitter:card" content="summary_large_image">

<!-- Keywords cho search -->
<meta name="keywords" content="TypeScript, JavaScript, Python, Programming, Tutorial, Learning">
```

### Sitemap tự động
Jekyll tự động tạo sitemap tại `/sitemap.xml` nhờ plugin `jekyll-sitemap`

## 🚨 Troubleshooting

### Website không hiển thị
1. Check GitHub Actions tab cho errors
2. Đảm bảo branch name là `main` (không phải `master`)
3. Repository phải là public
4. Đợi 5-10 phút sau khi push

### CSS không load
1. Check paths trong `_config.yml`
2. Đảm bảo `baseurl` đúng
3. Clear browser cache

### 404 errors
1. Check file paths và tên files
2. Đảm bảo links trong markdown đúng format
3. GitHub Pages case-sensitive với file names

## 📊 Monitoring

### GitHub Insights
- **Traffic**: Xem số lượng visitors
- **Popular content**: Trang nào được xem nhiều nhất
- **Referrers**: Visitors đến từ đâu

### Google Search Console
1. Add property với URL website
2. Submit sitemap: `https://your-site.com/sitemap.xml`
3. Monitor search performance

---

## 🎉 Kết quả cuối cùng

Sau khi hoàn thành, bạn sẽ có:

✅ **Website tĩnh professional** với GitHub Pages  
✅ **Tự động deployment** mỗi khi update code  
✅ **SEO-friendly** với sitemap và meta tags  
✅ **Mobile responsive** với Jekyll theme  
✅ **Search functionality** (nếu dùng Docsify)  
✅ **Free hosting** và custom domain support  

**URL ví dụ**: `https://nguyenngocbinh.github.io/typescript-roadmap`

*Chúc bạn thành công với website TypeScript roadmap! 🚀*
