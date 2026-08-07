## telegram-bot-shop-acc

> File này giúp AI agent hiểu và vận hành dự án **Telegram Shop Bot** từ A đến Z. Đọc file này trước khi làm bất cứ việc gì với repo.

# AGENTS.md — Hướng dẫn cho AI agent

File này giúp AI agent hiểu và vận hành dự án **Telegram Shop Bot** từ A đến Z. Đọc file này trước khi làm bất cứ việc gì với repo.

> Người dùng mới chỉ cần nói "chạy dự án" hoặc "deploy" → agent đọc file này + skill `.agent/skills/run-deploy/SKILL.md` rồi thực hiện đầy đủ, không hỏi lại từng bước (trừ khi thiếu secret/thông tin bí mật).

---

## Dự án là gì

Bot Telegram bán tài khoản số, chạy **trọn vẹn trên một Cloudflare Worker**:
- Webhook Telegram (`/webhook/telegram`) — flow mua hàng, nạp tiền, lịch sử, số dư.
- Webhook SePay (`/webhook/sepay`) — nạp tiền tự động qua VietQR, cộng số dư.
- REST API cho CMS (`/api/admin/*`) — quản trị, bảo vệ bằng JWT.
- Static Vue 3 SPA (`/cms/*`) — giao diện admin.
- Cron (`scheduled`) — hết hạn deposit pending mỗi 15 phút.

Chi tiết tính năng/endpoint/schema: xem `README.md`.

---

## Tech & cấu trúc

- **Backend**: TypeScript + Hono, entry `src/index.ts`.
- **DB**: Cloudflare D1 (binding `DB`, tên `telegram-shop-bot-db`), migrations trong `migrations/` chạy theo thứ tự (`0001`, `0002`, …).
- **CMS**: Vue 3 + Vite trong `cms/`, build ra `dist/cms/`.
- **Config**: `wrangler.toml`. **Local env**: `.dev.vars` (gitignored, mẫu `.dev.vars.example`).
- Thư mục chính: `src/bot` (handlers), `src/routes` (webhook + admin API), `src/services` (transaction/deposit-expiry), `src/middleware` (auth), `src/utils`.

---

## Lệnh chuẩn (package.json)

| Lệnh | Tác dụng |
|------|----------|
| `npm run dev` | Worker local (wrangler dev, port 8787) |
| `npm run build:cms` | Build Vue CMS → `dist/cms` |
| `npm run deploy` | Deploy Worker lên Cloudflare (KHÔNG tự build CMS) |
| `npm run db:migrate:local` | Áp migration vào D1 local |
| `npm run db:migrate:remote` | Áp migration vào D1 production |
| `npm test` | Vitest + fast-check |
| `bash test/e2e_full_flow.sh` | E2E thật qua HTTP (cần Worker local đang chạy) |

---

## Quy tắc khi vận hành (BẮT BUỘC)

1. **Progress realtime**: lệnh chạy lâu (dev server, build, test, deploy, e2e) → chạy bằng **background process** rồi đọc log dần. KHÔNG `cmd 2>&1 | tail` chờ im lặng. Script tự viết phải log từng bước.
2. **Build CMS trước deploy**: `deploy` không build CMS. Nếu sửa `cms/` → `npm run build:cms` rồi mới `npm run deploy`.
3. **Production cần xác nhận**: `db:migrate:remote` và `deploy` ảnh hưởng production → chỉ chạy khi user yêu cầu deploy rõ ràng.
4. **Secrets**: production dùng `wrangler secret put`, KHÔNG dùng `.dev.vars`. Không bịa giá trị secret, không echo secret ra response.
5. **Không sửa `database_id` / secret đã có** trừ khi user yêu cầu hoặc còn placeholder.
6. **Migration**: chỉ thêm file mới (`000N_*.sql`), không sửa migration đã chạy. Sau khi thêm cột DB nhớ cập nhật type trong `src/types/db.ts`.
7. **Verify sau thay đổi code**: chạy `npm test` và/hoặc build CMS để chắc không vỡ. Sau deploy báo Version ID + URL.
8. **CẤM emoji do agent tự thêm**: tuyệt đối KHÔNG chèn icon emoji vào bất cứ thứ gì agent viết — code, chuỗi message bot/Mini App, nhãn nút, text CMS, comment, README/docs, commit message. Dùng chữ thuần (vd `[x]`, `Lỗi:`, `Cảnh báo:`) thay cho emoji. NGOẠI LỆ duy nhất: emoji là **dữ liệu do người/admin nhập** (vd cột `product_types.emoji`, `success_template` admin tự soạn) hoặc user **yêu cầu rõ ràng** giữ/thêm. Khi sửa file đang có sẵn emoji, không tự ý thêm emoji mới; chỉ gỡ khi user yêu cầu.

---

## Quy ước UI/Style (BẮT BUỘC)

- **CMS**: icon dùng component `Icon.vue` (lucide), KHÔNG dùng emoji.
- **Bot/Mini App**: phần emoji hiện có là thiết kế cũ; giữ nguyên, KHÔNG nhân rộng. Tính năng mới do agent viết dùng chữ thuần (xem quy tắc 8 ở trên).
- Muốn đổi toàn bộ emoji có sẵn của bot → phải có yêu cầu rõ ràng của user (thay đổi UX lớn, đụng cả `reply keyboard` matching trong `src/bot/router.ts`).

---

## Khi user nói "chạy dự án" / "deploy"

Kích hoạt skill **`.agent/skills/run-deploy/SKILL.md`** và làm theo:
- "chạy dự án" / "run local" → Workflow A (chạy local).
- Máy mới / lần đầu → Workflow B (setup từ đầu).
- "deploy" / "lên production" → Workflow C (deploy).

Skill chứa từng bước chi tiết, checklist verify, và bảng lỗi thường gặp.

---

## Lưu ý nghiệp vụ quan trọng

- **Giao dịch atomic**: mua hàng/nạp tiền dùng D1 `batch()` + concurrency guard (`WHERE balance >= total`). Khi sửa `src/services/transaction.ts` phải giữ tính atomic và thứ tự: tạo order trước (lấy `orderId`) rồi mới batch các bước phụ thuộc (tránh lỗi FK `last_insert_rowid`).
- **userId trong bot**: các handler bot nhận `telegram_id` (không phải `users.id`). Khi query bảng `users`/`orders` phải JOIN hoặc lọc qua `telegram_id`, không dùng trực tiếp làm `users.id`.
- **Webhook idempotency**: SePay chống trùng theo `sepay_transaction_id`.
- **Template tin nhắn bán hàng**: render qua `src/utils/telegram-template.ts`. Header cố định + body từ `product_types.success_template`, placeholder `[content] [name] [emoji] [quantity] [total] [balance]`. Giá trị động phải escape HTML.
- **Số dư không âm**: CHECK constraint DB + logic. Không bypass.

---

## File tham chiếu nhanh

- `README.md` — setup chi tiết, API, schema, troubleshooting.
- `.agent/skills/run-deploy/SKILL.md` — workflow chạy/deploy từng bước.
- `docs/E2E_REPORT.md` — kết quả test E2E gần nhất.
- `.kiro/specs/telegram-shop-bot/` — requirements, design, tasks gốc.

---
> Source: [6c696e68/Telegram-Bot-Shop-Acc](https://github.com/6c696e68/Telegram-Bot-Shop-Acc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
