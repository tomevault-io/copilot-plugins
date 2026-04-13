## thanhnh98-github-io

> Quy tắc bắt buộc khi tạo hoặc chỉnh sửa bài viết tin tức (news.json, tin-tuc/*.html)


# News Publishing Rules

## Nội dung — Không dự đoán / không phán đoán biên tập (Bắt buộc)

- **Không** đăng dưới danh nghĩa trang các ý: dự báo giá, kịch bản tương lai, “triển vọng” do tự suy, “sẽ / có thể” mang tính tiên đoán thị trường nếu không phải **trích thuật có nguồn** (ghi rõ tên cơ quan/báo).
- **Được phép:** tóm tắt sự kiện và số liệu **đã xảy ra**; trích dẫn dự báo của cơ quan có thẩm quyền (vd. khí tượng, thông cáo chính thức) với link; mục “mốc tin công khai đáng theo dõi” **không** kèm kết luận về kết quả tương lai.
- Tiêu đề, `summary` trong `news.json`, meta description phải phản ánh quy tắc này (tránh cụm “dự báo / triển vọng / nhận định tương lai” nếu không phải trích nguồn).

## publishedAt — Ngày phát hành (Bắt buộc)

**Khi tạo bài mới:**
- Gán `publishedAt` = ngày giờ hiện tại lúc gen (ISO-8601, ví dụ `2026-03-16T10:30:00Z`).
- Dùng trong `news.json` và hiển thị tương ứng trong `tin-tuc/<slug>.html` (news-meta "Đăng: DD/MM/YYYY").
- Không dùng ngày tuần tự hoặc giả định (ví dụ: bài 1 → 12/03, bài 2 → 13/03, …).

**Sau khi tạo:**
- **KHÔNG BAO GIỜ** chỉnh sửa `publishedAt` — kể cả khi sửa nội dung, ảnh, affiliate hay metadata khác.
- `publishedAt` là ngày phát hành cố định, không thể thay đổi.

## Đồng bộ hiển thị

- `news.json` → `publishedAt` (ISO-8601)
- `tin-tuc/index.html` → hiển thị từ `news.json` qua `formatDate(post.publishedAt)`
- `tin-tuc/<slug>.html` → news-meta "Đăng: DD/MM/YYYY" phải khớp với `publishedAt`

## URL không đuôi `.html` trên GitHub Pages (Bắt buộc)

Sau khi tạo hoặc đổi slug file `tin-tuc/<slug>.html`, chạy:

- `npm run generate-tin-tuc-clean-urls`

Script tạo `tin-tuc/<slug>/index.html` (redirect nhẹ về `../<slug>.html`) vì **GitHub Pages không hỗ trợ** file `_redirects`.

## Cover image đồng bộ (Bắt buộc)

Khi user nói "ảnh cover cho bài báo", hiểu mặc định là **toàn bộ ảnh đại diện của bài** phải dùng cùng một nguồn ảnh:

- `news.json` → `thumbnailUrl` (ảnh thumbnail ngoài listing)
- `news.json` → `thumbnailSource.name` và `thumbnailSource.url` (nguồn ảnh)
- `tin-tuc/<slug>.html` → ảnh cover trong nội dung bài (`.news-cover`)
- `tin-tuc/<slug>.html` → `og:image`
- `tin-tuc/<slug>.html` → `twitter:image`
- `tin-tuc/<slug>.html` → JSON-LD `image`

### Quy tắc áp dụng

- Khi đổi cover, phải cập nhật **đồng thời tất cả vị trí trên**, không để lệch ảnh giữa listing/detail/social.
- Nếu user chỉ đưa 1 URL ảnh, dùng URL đó cho toàn bộ vị trí cover của bài.
- Không đổi `publishedAt` khi chỉ thay ảnh cover.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thanhnh98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
