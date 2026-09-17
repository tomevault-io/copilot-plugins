## funnel-mkt

> **Bộ 8 Agent chạy theo mô hình bánh đà (flywheel) để xây phễu marketing thu lead cho sản phẩm số và sản phẩm giáo dục.**

# Funnel Marketing — Hệ thống 8 Agent thu lead

**Bộ 8 Agent chạy theo mô hình bánh đà (flywheel) để xây phễu marketing thu lead cho sản phẩm số và sản phẩm giáo dục.**

> Trạng thái hiện tại: **Trọn bộ 8 agent · 45/45 skill · 10/10 template. Đợt 5 xong — hết phần build.**
> ⚠️ **Chưa chạy thử lần nào.** 45 skill đã viết nhưng chưa có dự án thật nào chạy qua. Bài nghiệm thu #1–#14 trong `KE-HOACH.md` vẫn đang chờ.
> Đọc `KE-HOACH.md` để biết sẽ build gì. Đọc `HUONG-DAN-CHO-NGUOI-DUNG.md` nếu là người dùng cuối.

---

## Bánh đà 8 Agent

| # | Lệnh | Agent | Trả lời câu hỏi |
|---|------|-------|-----------------|
| ① | `/tukhoa` | Nghiên cứu Từ khoá | Khách đang tìm gì? Từ nào đáng đánh? |
| ② | `/insight` | Insight khách hàng & Business Model Canvas | Khách là ai? Đau ở đâu? Muốn gì? |
| ③ | `/doithu` | Phân tích Đối thủ | Ai đang làm? Họ bỏ sót chỗ nào? |
| ④ | `/usp` | Chốt USP | Mình khác biệt ở đâu? (USP = Từ khoá × GAP) |
| ⑤ | `/quatang` | Quà tặng | Tặng gì để khách để lại thông tin? |
| ⑥ | `/phanphoi` | Phân phối quà & Thu lead | Giao quà qua đâu? Lead về đâu? |
| ⑦ | `/traffic` | Traffic Free + Paid | Kéo người vào phễu bằng cách nào? |
| ⑧ | `/doluong` | Đo lường & Kỷ luật số liệu | Hiệu quả thật ra sao? Sửa ở đâu? |

**Chiều quay:** ① → ② → ③ → ④ → ⑤ → ⑥ → ⑦ → ⑧ → ① (vòng mới)

Xem sơ đồ trực quan: mở `docs/so-do-flywheel.html` bằng trình duyệt.

---

## Luật bắt buộc — Đọc kho và Bơm vào kho

Đây là thứ phân biệt bánh đà với dây chuyền. **Mọi agent phải theo đúng 3 bước:**

```
1. ĐỌC KHO   →  mở các file liên quan trong kho/ trước khi làm bất cứ việc gì
2. LÀM VIỆC  →  thực thi skill, xuất tài liệu vào du-an/{tên-dự-án}/
3. BƠM KHO   →  ghi lại phát hiện mới vào kho/ — bước này KHÔNG được bỏ
```

**Quy tắc vàng:** nếu trong kho đã có dữ liệu thật mà agent vẫn đưa ra con số ước lượng, đó là **lỗi**. Phải ưu tiên số đã kiểm chứng, và ghi rõ nguồn: "đã kiểm chứng ở vòng 1: CPL 87.000 VND / 34 lead".

### Bản đồ kho

| File trong `kho/` | Ai bơm vào | Ai đọc |
|---|---|---|
| `tu-khoa-da-kiem-chung.csv` | ⑧ | ① |
| `ngon-ngu-khach.md` | ② ⑧ | ② ④ ⑥ |
| `doi-thu-theo-doi.csv` | ③ ⑧ | ③ |
| `usp-da-kiem-chung.md` | ④ ⑧ | ④ |
| `qua-tang/` | ⑤ | ⑤ |
| `hook-thang.csv` | ⑦ ⑧ | ① ⑦ |
| `creative-thang/` | ⑦ ⑧ | ⑦ |
| `benchmark-cua-minh.csv` | ⑧ | ⑥ ⑦ ⑧ |

### Mũi tên hồi tiếp từ ⑧ — bắn về đúng địa chỉ

| ⑧ thấy gì | Bắn về | Sửa cái gì |
|---|---|---|
| Search term thật / CPL theo từ khoá | ① | Bỏ từ khoá đắt, nhân từ khoá rẻ |
| Comment · inbox · trả lời form | ② | Nỗi đau mới, ngôn ngữ khách thật |
| Đối thủ xuất hiện khi mình chạy ads | ③ | Cập nhật hồ sơ, tìm gap mới |
| Lead về sai người (trượt tệp) | ④ | Siết lại USP & câu lọc tệp |
| Tỷ lệ tiêu thụ quà thấp | ⑤ | Đổi định dạng / rút ngắn quà |
| Opt-in thấp · vào nhóm < 60% | ⑥ | Sửa landing, thank-you, kịch bản mess |
| CPL vượt trần · engagement tụt | ⑦ | Kill ad set, đổi hook, đổi creative |

**⑧ không được kết luận chung chung.** Mọi phát hiện phải chỉ đích danh agent nào phải sửa.

---

## Nguyên tắc thiết kế

**Chia theo TỆP khách.** Mọi tài liệu đều song song nhiều cột — mỗi tệp một cột, không gộp chung. Một sản phẩm thường có 2 tệp (ví dụ: người trực tiếp bán hàng / người quản lý đội bán hàng), nỗi đau và ngôn ngữ của hai tệp khác hẳn nhau.

**Kỷ luật số liệu kiểu CFO.** Trước khi tiêu tiền phải có dự toán và ngưỡng CPL trần. Trong khi chạy có đèn vàng / đèn đỏ. Điều kiện scale phải thoả **đồng thời** nhiều chỉ số, không chỉ nhìn CPL.

**Không bịa số.** Thiếu dữ liệu thì báo thiếu và chỉ rõ cần chạy agent nào trước, không tự suy diễn con số.

**Kiểm kê trước khi sản xuất.** Agent ⑤ luôn rà tài nguyên sẵn có (video cũ, bản ghi Zoom, bài viết) trước khi đề xuất làm mới.

---

## Cấu trúc thư mục

```
funnel-marketing/
├── CLAUDE.md                      ← file này
├── KE-HOACH.md                    ← bản chốt kỹ thuật đầy đủ
├── HUONG-DAN-CHO-NGUOI-DUNG.md    ← hướng dẫn cho người dùng cuối
├── docs/
│   ├── so-do-flywheel.html        ← sơ đồ trực quan, mở bằng trình duyệt
│   └── mo-hinh-goc.md             ← nguồn gốc mô hình, lưu vết
├── agents/                        ← 8 file agent          (8/8 — ① đến ⑧)
├── .claude/commands/              ← 8 lệnh                (8/8 — ① đến ⑧)
├── skills/                        ← 45 skill              (**45/45 — trọn bộ ①②③④⑤⑥⑦⑧**)
├── templates/                     ← mẫu CSV/HTML          (**10/10 — đủ** — ma-tran-insight · ho-so-doi-thu · ebook · tai-lieu-can · kich-ban-messenger · landing-optin · thank-you · chuoi-email-7-ngay · du-toan-tai-chinh · scorecard)
├── package.json                   ← puppeteer-core        (đã cài, đã chạy thử)
├── scripts/                       ← html → pdf            (xong — html-to-pdf.js ra PDF thật)
├── kho/                           ← TRỤC BÁNH ĐÀ          (xong, 8 file rỗng có header)
└── du-an/{tên-dự-án}/             ← hồ sơ từng phễu       (sinh khi chạy)
```

**Vì sao `agents/` và `skills/` nằm ở gốc chứ không nằm trong `.claude/`:** Claude Code chỉ tự nhận diện skill ở `.claude/skills/` và không cho lồng thư mục — 45 skill sẽ phải nằm dẹt, mất cách nhóm theo 8 cụm, mô tả của cả 45 cái bị nạp vào mọi phiên làm việc, và skill có thể tự kích hoạt ngoài luồng. Bánh đà cần chạy đúng thứ tự nên ta chọn ngược lại: chỉ `.claude/commands/` là điểm vào, còn `agents/` và `skills/` là file được đọc tường minh theo đúng thứ tự đã định.

**Phân biệt `du-an/` và `kho/`:**
- `du-an/` — hồ sơ của **một** phễu, chết theo chiến dịch. Không commit lên git (có thông tin khách hàng).
- `kho/` — tài sản **xuyên mọi** phễu, chỉ dày lên, không bao giờ xoá. Đây là thứ khiến vòng sau rẻ hơn vòng trước.

---

## Quy tắc ngôn ngữ & trình bày

- **Tiếng Việt đầy đủ dấu.** Xưng hô anh/chị.
- **Tiền tệ VND**, ví dụ từ thị trường Việt Nam.
- **Mỗi output phải dùng được ngay** — có số cụ thể, có ví dụ thật, có bước làm rõ ràng.
- **Không rõ yêu cầu:** hỏi tối đa 2 câu rồi thực thi, không đoán mò.
- **Xuất song song `.md` và `.csv`** khi tài liệu là dạng bảng — `.md` để đọc, `.csv` để dán vào Google Sheets.

---

## Git

Mỗi đợt build là một commit.

Repo đã công khai tại **`thaofvn-coca06/funnel-mkt`**. Từ đây được phép `git push`, nhưng **chỉ push khi anh/chị bảo** — không tự đẩy sau mỗi commit.

`du-an/` nằm trong `.gitignore` và **không bao giờ được commit** — nó chứa dữ liệu khách hàng thật, mà repo thì đang công khai.

---
> Source: [thaofvn-coca06/funnel-mkt](https://github.com/thaofvn-coca06/funnel-mkt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
