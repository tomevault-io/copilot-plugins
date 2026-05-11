## flash-sale

> > Stack: Next.js 16 + React 19 (frontend) · NestJS 10 + Prisma 6 (backend) · PostgreSQL · Redis · RabbitMQ

# Flash Sale Monorepo — Claude Instructions

> Stack: Next.js 16 + React 19 (frontend) · NestJS 10 + Prisma 6 (backend) · PostgreSQL · Redis · RabbitMQ
> Last updated: 2026-03-28

## Project Context

Flash Sale là monorepo gồm web frontend và API backend cho nền tảng thương mại theo chiến dịch flash sale với các role Customer, Merchant, Admin. Hệ thống tập trung vào luồng đăng ký/duyệt merchant, quản lý campaign, đặt mua, checkout/payment, theo dõi order và dashboard vận hành.

**Workspace layout**
- `flashsale-frontend/`: Next.js App Router, TypeScript, Tailwind
- `flashsale-backend/`: NestJS, Prisma/PostgreSQL, Redis, RabbitMQ
- `docs/`: tài liệu runtime cho workflow .claude ở cấp monorepo
- `.claude/`: operating model, workflows, agents, standards, guardrails, templates

---

## Precedence Order

See `.claude/core/operating-model.md` — canonical authority for precedence, conflict resolution, ownership, and workflow authority.

---

## Critical Rules

1. `PRD.md` là read-only cho agent; chỉ human được sửa khi có chỉ định rõ trong phiên hiện tại.
2. `TODO.md` là backlog sống, do `project-manager` quản lý.
3. Không hardcode secrets; không commit file `.env`.
4. Luôn cập nhật docs liên quan sau thay đổi đáng kể (API/DATABASE/ARCHITECTURE/USER_GUIDE).
5. Không đánh dấu hoàn thành feature FE/BE nếu chưa có báo cáo verify Playwright MCP (PASS + screenshot + error logs nếu fail).
6. Không push trực tiếp vào `main`; tuân thủ branch flow hiện có (`develop`, `staging`, `main`).
7. Dùng lệnh thực tế của repo (`pnpm`, không dùng npm/yarn).

---

## Agents Available

- `project-manager`
- `systems-architect`
- `backend-developer`
- `frontend-developer`
- `database-expert`
- `qa-engineer`
- `documentation-writer`
- `cicd-engineer`
- `docker-expert`
- `ui-ux-designer`
- `copywriter-seo`
- `react-native-developer` (không active cho repo hiện tại, chỉ dùng nếu sau này có mobile app)

---

## Real Commands

### Monorepo
- Frontend lint: `cd flashsale-frontend && pnpm lint`
- Frontend build: `cd flashsale-frontend && pnpm build`
- Backend lint: `cd flashsale-backend && pnpm lint`
- Backend test: `cd flashsale-backend && pnpm test`
- Backend test:e2e: `cd flashsale-backend && pnpm test:e2e`

### Backend notes
- Prisma generate: `cd flashsale-backend && pnpm prisma:generate`
- Prisma migrate dev: `cd flashsale-backend && pnpm prisma:migrate:dev`

### Frontend notes
- Dev server: `cd flashsale-frontend && pnpm dev`

---

## Documentation Map

- `docs/technical/ARCHITECTURE.md`
- `docs/technical/DECISIONS.md`
- `docs/technical/API.md`
- `docs/technical/DATABASE.md`
- `docs/user/USER_GUIDE.md`
- `docs/content/CONTENT_STRATEGY.md`

Nếu tài liệu chưa đầy đủ nội dung, giữ skeleton và gắn rõ marker `TODO(fill-with-project-context)` để điền dần theo từng feature.

---
> Source: [HDuong1803/flash-sale](https://github.com/HDuong1803/flash-sale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
