## gpt-upi-tool

> Hướng dẫn cho coding agent (Kiro / Codex / Claude / khác) khi làm việc trong repo này.

# AGENTS.md — gpt_signup_hybrid

Hướng dẫn cho coding agent (Kiro / Codex / Claude / khác) khi làm việc trong repo này.

## Ngôn ngữ + style

- Trả lời tiếng Việt, ngắn gọn, đi thẳng vấn đề.
- Không tổng kết dài dòng. Không tạo doc/markdown khi user không yêu cầu.

## File layout

- File test/debug → `test/`
- Tài liệu .md user yêu cầu → `docs/`
- Không tạo file tạm ngoài 2 chỗ trên.

## Verify / Debug / Run

- **Cấm** dùng inline `python3 -c "..."`, `node -e "..."`, `bash -c "..."`, `eval` để verify hay debug.
- Mọi check (syntax, import, smoke, repro bug) phải nằm trong file `.py`/`.js`/`.sh` thật ở `test/`.
- Đặt tên rõ ràng:
  - `test/syntax_check.py` — parse AST mọi file Python.
  - `test/check_<scope>.py` — check chức năng cụ thể.
  - `test/smoke_<scope>.py` — smoke test integration.
  - `test/test_<scope>.py` — unit test.
- Chỉ chạy file vừa viết: `python3 test/<file>.py`. Không chạy script ad-hoc rồi xóa.
- cấm chạy python -c mà hãy tạo file test mà chạy
## Code rules

- Kiến trúc tổng thể, không feature rời rạc. SOLID, DRY, Fail-Fast.
- Không hardcode default insecure (TLS verify off, CORS *, auth bypass) — phải opt-in qua flag/env.
- Không fallback che lỗi. Không code chết sau `return`/`raise`.
- Không viết test khi user không yêu cầu (trừ test verify ở mục trên).

## Quy ước

- User nói "tiếp tục" → làm tiếp theo best practice, không hỏi lại.
- Yêu cầu mơ hồ → dừng, hỏi rõ với options cụ thể.
- Thấy rủi ro hoặc cách tốt hơn → nói thẳng.

## Tool ưu tiên

- Tra docs → Context7 MCP
- Test web UI → Playwright MCP

## Settings Store (BẮT BUỘC)

Dự án dùng SQLite Settings Store (`SettingsRepository` trong `db/repositories.py`) làm single source of truth cho mọi runtime configuration. **KHÔNG được bỏ qua.**

- **Backend**: Đọc config từ DB tại startup (`apply_settings`), ghi qua write-through khi user thay đổi.
- **Frontend**: Dùng `Settings.get(key)` / `Settings.save(key, value, token)`. KHÔNG dùng `localStorage` cho config.
- **Thêm key mới**: Phải thêm vào `_EXACT_KEYS` + `_validate_type_constraint()` trước khi sử dụng.
- **Không file config riêng**: Không JSON/YAML config file — tất cả vào bảng `settings`.
- **Namespace**: `namespace.field` format (dot-separated lowercase). VD: `reg.headless`, `hotmail.concurrency`.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **gpt_signup_hybrid-main** (10633 symbols, 26425 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> Index stale? Run `node .gitnexus/run.cjs analyze` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? `npx gitnexus analyze` (npm 11 crash → `npm i -g gitnexus`; #1939).

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows. For regression review, compare against the default branch: `detect_changes({scope: "compare", base_ref: "main"})`.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit changes without running `detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/gpt_signup_hybrid-main/context` | Codebase overview, check index freshness |
| `gitnexus://repo/gpt_signup_hybrid-main/clusters` | All functional areas |
| `gitnexus://repo/gpt_signup_hybrid-main/processes` | All execution flows |
| `gitnexus://repo/gpt_signup_hybrid-main/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Source: [bunnies3011/gpt-upi-tool](https://github.com/bunnies3011/gpt-upi-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
