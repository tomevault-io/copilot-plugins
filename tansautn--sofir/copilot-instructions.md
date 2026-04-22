## sofir

> You are allowed to be proactive, but only when the user asks you to do something. You should strive to strike a balance between:


## 🧠 Agent LLM Rules for Zuko's Projects

### 1. Pro activeness
You are allowed to be proactive, but only when the user asks you to do something. You should strive to strike a balance between:
1. Doing the right thing when asked, including taking actions and follow-up actions
2. Not surprising the user with actions you take without asking
For example, if the user asks you how to approach something, you should do your best to answer their question first, and not immediately jump into taking actions.
3. Do not add additional code explanation summary unless requested by the user. After working on a file, just stop, rather than providing an explanation of what you did.


### 2. Tone and Style

* ❗ **No preamble or postamble**: Trả lời trực tiếp, không giới thiệu thừa hoặc kết luận trừ khi được yêu cầu.
* ✍️ **Language**:

  * Code, class, method, variable: **English only**. Đại loại là chỉ sử dụng tiếng Anh trong bất kỳ code nào.
  * Response/explanation: Cho phép dùng **Tiếng Việt** khi cần diễn giải kỹ thuật.
* 📦 **Minimal output tokens**: Trả lời ngắn gọn, đúng trọng tâm.

---

### 3. Documentation & Workflow – Document First

* 🧾 Mọi thay đổi phải bắt đầu bằng tài liệu (doc-first), gồm:

  * **Diễn giải mục tiêu** (requirement/why),
  * **Kế hoạch thực hiện (step-by-step)**,
  * **Mermaid diagrams** nếu logic phức tạp hoặc liên quan đến nhiều modules.
* 🔁 Mọi chức năng (Feature, Event, Listener, UI, Policy, v.v...) **phải** có sơ đồ/phác thảo trước khi code.
* 📂 Đặt tài liệu tại thư mục `docs/` trong module tương ứng nếu có.
* ✅ Dùng những thứ sẵn có, hoặc tạo lớp base khi có thể để giảm boilerplate.
* ✅ Luôn listing file in directories. Đọc sơ qua các file có `abstract` hoặc `base` trong filename xem chúng có gì hữu dụng với task đang thực thi không


---

### 4. LLM Behavior Guide

#### Allowed

* ✅ Đề xuất refactor nếu code trùng lặp
* ✅ Tự động tạo docs/mermaid nếu thấy thiếu
* ✅ Gợi ý test nếu chưa có

#### Not Allowed

* ❌ Không generate code nếu chưa có tài liệu sơ bộ
* ❌ Không merge logic controller với logic service
* ❌ Không tạo file ngoài cấu trúc module quy định

---

### 5. Agent Tool Usage

#### Package usage
* Kiểm tra sự tồn tại của packages trước khi gợi ý dùng package mới.
* Không assume thư viện tồn tại — kiểm tra `<root project>/composer.json` trước, và cả các file `composer.json` trong các module.

#### Intergrating Packages to Project
* Luôn luôn kiểm tra  tài liệu của package đang intergrate, Mỗi developer có một hướng thiết kế khác nhau
* Để tra cứu tài liệu, sử dụng `context7` MCP. Nói chung khi nghĩ về tài liệu (đọc) hãy nhớ `context7`

#### Git
* Git: Project always have untracked files, and i want it remain untracked. Only `git add` what you created or changed.
  Nothing else. `git add .` is fobidden.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tansautn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
