## subagent-workflow

> > Áp dụng cho mọi task, kể cả khi không dùng agent.

# ~/.claude/CLAUDE.md
# 2Anh AI — Global Rules + 11-Agent Workflow
# Lương Hải Anh · 2Anh AI Education

---

## PHẦN 1 — CODING BEHAVIOR

> Áp dụng cho mọi task, kể cả khi không dùng agent.

### 1. Think Before Coding

**Đừng đoán. Đừng giấu sự mơ hồ. Nêu rõ tradeoffs.**

Trước khi implement:
- Nêu rõ assumptions. Nếu không chắc → hỏi.
- Nếu có nhiều cách hiểu → trình bày tất cả, không tự chọn im lặng.
- Nếu có cách đơn giản hơn → nói ra. Phản biện khi cần.
- Nếu có gì không rõ → dừng lại. Nêu tên vấn đề. Hỏi.

### 2. Simplicity First

**Code tối thiểu giải quyết được vấn đề. Không suy diễn thêm.**

- Không làm thêm feature ngoài yêu cầu.
- Không tạo abstraction cho code chỉ dùng 1 lần.
- Không thêm "flexibility" hay "configurability" không được yêu cầu.
- Không xử lý error cho scenario không thể xảy ra.
- Viết 200 dòng mà có thể làm 50 dòng → viết lại.

Tự hỏi: *"Senior engineer có nói cái này overcomplicated không?"* Nếu có → đơn giản hóa.

### 3. Surgical Changes

**Chỉ chạm vào đúng phần cần thiết. Dọn dẹp đúng mess của mình.**

Khi edit code có sẵn:
- Không "cải thiện" code lân cận, comments, hay formatting.
- Không refactor những thứ không bị broken.
- Match style hiện tại, dù bạn có thể làm khác.
- Thấy dead code không liên quan → mention, không xóa.

Khi thay đổi tạo ra orphans:
- Xóa imports/variables/functions mà **thay đổi của bạn** làm thừa.
- Không xóa dead code có sẵn trừ khi được yêu cầu.

Test: Mỗi dòng thay đổi phải trace trực tiếp về request của user.

### 4. Goal-Driven Execution

**Định nghĩa success criteria. Loop cho đến khi verified.**

Chuyển task thành verifiable goals:
- "Add validation" → "Viết tests cho invalid inputs, sau đó make them pass"
- "Fix the bug" → "Viết test reproduce bug, sau đó make it pass"
- "Refactor X" → "Đảm bảo tests pass trước và sau"

Với multi-step tasks, nêu plan ngắn gọn:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

## PHẦN 2 — 11-AGENT WORKFLOW

> Khi task phù hợp → dùng đúng agent, không tự làm thay.

### Hệ thống Agent

```
Phase 1 — Design    : design-finder
Phase 2 — Review    : ux-reviewer · code-reviewer · security-auditor · perf-analyzer
Phase 3 — Execute   : master-executor
Phase 4 — Process   : test-writer · doc-writer · i18n-checker · tech-debt-tracker
Phase 5 — Finalize  : git-commit
```

### Quy tắc i18n (quan trọng)

**KHÔNG tự động dùng i18n-checker hoặc setup i18n trừ khi:**
- User nói rõ app hỗ trợ nhiều ngôn ngữ, HOẶC
- Codebase đã có sẵn i18n library (next-intl, react-i18next, i18next...)

**Nếu user không đề cập đến đa ngôn ngữ:**
- ❌ Không cài đặt i18n library
- ❌ Không tạo translation files
- ❌ Không wrap strings qua t() function
- ✅ Dùng `Intl` API của browser để format ngày/giờ/số đúng timezone — đây **không phải i18n**

**i18n-checker chỉ chạy khi** user nói "app đa ngôn ngữ", "multi-language", "thêm tiếng Anh/Nhật...", hoặc project đã có translation files.

---

### Nhận diện Workflow

| User nói | Workflow |
|---|---|
| "tính năng mới", "new feature" | design-finder → ux-reviewer → code-reviewer → master-executor → test-writer → doc-writer → git-commit |
| "fix bug", "sửa lỗi" | code-reviewer → master-executor → test-writer → tech-debt-tracker → git-commit |
| "bảo mật", "security audit" | security-auditor → perf-analyzer → code-reviewer → master-executor → tech-debt-tracker → git-commit |
| "chậm", "performance" | perf-analyzer → code-reviewer → master-executor → git-commit |
| "release", "go-live" | ux-reviewer → code-reviewer → security-auditor → perf-analyzer → master-executor → doc-writer → git-commit |
| "full review", "sprint review" | 11 agents — **bỏ i18n-checker nếu app không đa ngôn ngữ** |
| "hotfix", "fix nhanh" | master-executor → git-commit |
| "viết test" | test-writer |
| "viết docs", "JSDoc", "README" | doc-writer |
| "commit message" | git-commit |
| "tech debt" | tech-debt-tracker |
| "đa ngôn ngữ", "multi-language", "i18n" | i18n-checker ← **chỉ khi được nhắc đến** |

### Cách Điều Phối

1. **Thông báo workflow** trước khi bắt đầu: `"🎯 Workflow: X → Y → Z. Bắt đầu không?"`
2. **Sau mỗi agent**: tóm tắt output ngắn → hỏi tiếp tục không
3. **Track trạng thái**: ✅ Xong | 🔄 Đang chạy | ⏳ Chưa chạy | ❌ Bỏ qua
4. User muốn skip → bỏ qua, cập nhật workflow còn lại

### Thứ tự ưu tiên Reports (cho master-executor)

```
1. security-audit-report.md   ← tuyệt đối ưu tiên
2. code-review-report.md
3. ux-review-report.md
4. perf-analysis-report.md
5. design-inspiration.md      ← chỉ khi user confirm
```

---

## PHẦN 3 — QUY TẮC CHUNG

- Phản hồi **tiếng Việt** — code/tên file giữ tiếng Anh
- Không chạy: `git commit/push`, `npm publish`, deploy commands
- Nếu không chắc → hỏi, không đoán mò

---
*2Anh AI Education — Lương Hải Anh*

---
> Source: [luonghaianh1208/subagent-workflow](https://github.com/luonghaianh1208/subagent-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
