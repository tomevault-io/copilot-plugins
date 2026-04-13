## vinfast-ai-assistant

> > Tài liệu hướng dẫn phát triển giao diện React.js cho chatbot tư vấn xe điện VinFast.

# AGENTS.md — VinFast AI Chatbot (Vivi AI)

> Tài liệu hướng dẫn phát triển giao diện React.js cho chatbot tư vấn xe điện VinFast.  
> Dựa trên: Canvas (02), User Stories & 4 Paths (03), Failure Modes (05), Flow diagram.

---

## 1. Tổng quan hệ thống

**Tên agent:** Vivi AI — Online Assistant  
**Loại:** Augmentation (AI gợi ý, con người quyết định cuối)  
**Scope:** Tư vấn xe điện VinFast + tính toán tài chính trả góp  
**Không thuộc scope:** So sánh hãng khác, tin tức báo chí, thông tin không liên quan VinFast

---

## 2. Kiến trúc luồng (Flow Architecture)

Toàn bộ hội thoại chia làm **3 giai đoạn tuần tự**, mỗi giai đoạn là một "Phase" trong state của React:

```
PHASE 0: GREETING
    ↓
PHASE 1: CAR_DISCOVERY  (vòng lặp cho đến khi khách ưng 1 mẫu)
    ↓
PHASE 2: FINANCE         (mua thẳng hoặc trả góp)
    ↓
PHASE 3: HANDOFF         (thu thập SĐT, chuyển Sale)
```

### State Machine tổng thể

```
type Phase = 
  | 'GREETING'
  | 'CAR_DISCOVERY'
  | 'FINANCE_QUESTION'       // hỏi mua thẳng hay trả góp
  | 'FINANCE_FULL_PAY'       // tư vấn mua thẳng
  | 'FINANCE_INSTALLMENT'    // tư vấn trả góp
  | 'HANDOFF_COLLECT'        // xin SĐT / tên
  | 'HANDOFF_DONE'           // kết thúc, chờ Sale liên hệ

type Confidence = 'HIGH' | 'LOW' | 'FAILURE'
```

---

## 3. Mô tả từng Phase

### PHASE 0 — GREETING

**Trigger:** Khách mở widget chat  
**AI làm gì:** Chào hỏi, giới thiệu bản thân là AI, đặt câu hỏi mở  
**UI component:** `<GreetingBubble />` + `<QuickReplyBar />` với 2–3 gợi ý nhanh

**Quick replies gợi ý:**
- "Tôi muốn tìm hiểu về xe VinFast"
- "Tính trả góp xe điện"
- "So sánh các dòng xe"

**Chuyển sang:** `CAR_DISCOVERY` khi user nhập hoặc chọn quick reply

---

### PHASE 1 — CAR_DISCOVERY

**Trigger:** User mô tả nhu cầu xe (model, tính năng, ngân sách sơ bộ, hoàn cảnh gia đình)  
**AI làm gì:** Gọi tool `get_car_info()` → trả về thông số, giá, ảnh xe phù hợp

#### Tool được gọi:
```
get_car_info(query: string, filters?: { model?, segment?, budget_range? })
→ CarInfo[]
```

#### 4 Paths trong CAR_DISCOVERY:

| Path | Trigger | UI xử lý |
|------|---------|----------|
| **Happy** | AI tìm đúng xe, khách hỏi rõ | Hiển thị `<CarCard />` với thông số + ảnh. Nút "Tìm hiểu thêm" và "Chọn xe này" |
| **Low-confidence** | Câu hỏi quá ngắn / mơ hồ ("VF5 sao?") | AI hỏi làm rõ: hiển thị `<ClarifyPrompt />` với 2–3 lựa chọn có sẵn |
| **Failure** | AI báo thông số sai, user phản hồi | Hiển thị nút `<HandoffButton label="Kết nối tư vấn viên" />` + xin lỗi |
| **Correction** | User nhấn Dislike / gõ "sai rồi" | Capture feedback → log vào dashboard, AI hỏi lại |

**Điều kiện chuyển Phase:** User nhấn "Chọn xe này" hoặc AI detect đủ tín hiệu ưng ý (từ khóa: "ổn", "được", "xe này đi", v.v.) → chuyển sang `FINANCE_QUESTION`

**Vòng lặp:** Nếu khách đổi ý hoặc còn thắc mắc → quay lại `CAR_DISCOVERY`

---

### PHASE 2A — FINANCE_QUESTION

**Trigger:** Khách đã chọn được mẫu xe  
**AI làm gì:** Hỏi phương thức mua  
**UI component:** `<FinanceOptionCard />` với 2 lựa chọn

```
[ 💳 Mua thẳng ]     [ 📅 Trả góp / Vay ngân hàng ]
```

---

### PHASE 2B — FINANCE_FULL_PAY

**Trigger:** Chọn "Mua thẳng"  
**AI làm gì:** Gọi tool `get_promotions()` → hiển thị khuyến mãi, giá lăn bánh  
**UI component:** `<PriceSummaryCard />` + `<PromotionBadge />`

#### Tool được gọi:
```
get_promotions(model: string, region?: string)
→ { base_price, promotions[], road_registration_fee, total_on_road }
```

**Disclaimer bắt buộc:** `"Giá trên mang tính tham khảo, tư vấn viên sẽ xác nhận con số chính thức."`

---

### PHASE 2C — FINANCE_INSTALLMENT

**Trigger:** Chọn "Trả góp"  
**AI làm gì:** Slot-filling 3 tham số bắt buộc trước khi tính:

```
required_slots = {
  down_payment: number | null,   // % hoặc số tiền tuyệt đối
  loan_term_months: number | null,  // 12 | 24 | 36 | 48 | 60
  interest_rate: number | null   // % / năm (default: lãi suất VinFast hiện hành)
}
```

**Chỉ gọi tool khi đủ cả 3 slot.** Nếu thiếu → AI hỏi lần lượt từng slot.

**UI component:** `<SlotFillForm />` hiển thị trạng thái từng slot (filled / pending)

**Xử lý Low-confidence lãi suất:**
> "Bạn muốn vay qua ngân hàng nào, hay để AI áp dụng lãi suất ưu đãi VinFast hiện tại (8%/năm)?"  
> → Hiển thị `<BankSelector />` hoặc nút "Dùng lãi suất VinFast"

#### Tool được gọi:
```
calculate_installment({
  car_price: number,
  down_payment_ratio: number,     // 0.0–1.0
  loan_term_months: number,
  annual_interest_rate: number
})
→ {
  monthly_payment: number,
  total_payment: number,
  total_interest: number,
  schedule: MonthlyEntry[]        // amortization table
}
```

**UI output:** `<InstallmentTable />` — bảng trả góp tháng đầu + tổng quan  
**Disclaimer bắt buộc:** Hiển thị nổi bật dưới bảng:  
> `"⚠️ Bảng tính mang tính chất tham khảo. Tư vấn viên sẽ chốt con số cuối cùng."`

#### 4 Paths trong FINANCE_INSTALLMENT:

| Path | Trigger | UI xử lý |
|------|---------|----------|
| **Happy** | Đủ slot, tool tính xong | Hiển thị `<InstallmentTable />` + nút "Liên hệ để đặt cọc" |
| **Low-confidence** | Thiếu slot (ví dụ chưa chọn ngân hàng) | `<SlotFillForm />` highlight slot còn thiếu + câu hỏi làm rõ |
| **Failure** | Tool tính ra số bất thường (quá cao/thấp) | Hiển thị warning + nút "Kết nối tư vấn viên để kiểm tra" |
| **Correction** | User nói "Ý tôi là 30% giá xe" | AI parse lại, re-call tool. Log lỗi entity extraction để cải thiện |

---

### PHASE 3 — HANDOFF_COLLECT

**Trigger:** User nhấn "Đặt cọc", "Lái thử", "Liên hệ tư vấn viên"  
**AI làm gì:** Xin tên + SĐT  
**UI component:** `<ContactForm />` — input tên + số điện thoại + checkbox đồng ý PDPA

**Validate:** SĐT 10 số VN (regex). Không submit nếu thiếu.

**Sau khi submit:**
1. Hệ thống gọi `save_lead({ name, phone, conversation_context, selected_car, finance_summary })`
2. Trigger CRM notification cho Sales team
3. Chuyển sang `HANDOFF_DONE`

---

### PHASE 3B — HANDOFF_DONE

**UI:** `<HandoffConfirmCard />` — thông báo thành công  
> "Cảm ơn [Tên]! Tư vấn viên VinFast sẽ liên hệ bạn trong vòng 30 phút trong giờ làm việc."

---

## 4. Xử lý Out-of-Scope & Guardrails

Khi user hỏi ngoài phạm vi (so sánh hãng khác, tin báo chí, prompt injection):

```
AI response: "Mình chỉ có thể tư vấn về xe điện VinFast. 
Bạn có muốn mình hỗ trợ tìm hiểu mẫu xe hoặc tính toán tài chính không?"
```

**UI:** Hiển thị `<OutOfScopeCard />` + 2 quick reply để dẫn lại flow chính.  
**Log:** Mọi câu hỏi out-of-scope được log với flag `OUT_OF_SCOPE` để review.

---

## 5. Learning Signal — UI Implementation

### Explicit Feedback
Mỗi bubble tin nhắn AI có: `👍` / `👎` ẩn, hiện khi hover  
Khi nhấn 👎 → hiện `<FeedbackModal />` với lý do: "Sai thông tin" / "Không liên quan" / "Khác"

### Implicit Signals (log tự động)
| Signal | Cách đo |
|--------|---------|
| Handoff rate | Đếm session kết thúc bằng HANDOFF / tổng session |
| Session length | Timestamp từ GREETING → HANDOFF hoặc rời trang |
| Re-ask rate | Detect user hỏi lại cùng chủ đề trong 1 session |
| Slot correction rate | Đếm lần tool bị gọi lại sau correction |

---

## 6. Component Map (React)

```
<ChatWidget>                        // Container chính, quản lý phase state
  <ChatHeader />                    // Logo Vivi AI, status "Online"
  <MessageList>                     // Danh sách bubble
    <AiBubble />                    // Tin nhắn AI
    <UserBubble />                  // Tin nhắn user
    <TypingIndicator />             // 3 chấm khi đang gọi tool
    
    // Phase-specific components
    <QuickReplyBar />               // GREETING: quick replies
    <CarCard />                     // CAR_DISCOVERY: thông số xe
    <ClarifyPrompt />               // LOW_CONFIDENCE: hỏi làm rõ
    <FinanceOptionCard />           // FINANCE_QUESTION: chọn phương án
    <SlotFillForm />                // INSTALLMENT: trạng thái slot
    <BankSelector />                // INSTALLMENT: chọn ngân hàng
    <InstallmentTable />            // INSTALLMENT: bảng trả góp
    <PriceSummaryCard />            // FULL_PAY: tóm tắt giá
    <DisclaimerBanner />            // FINANCE: cảnh báo tham khảo
    <ContactForm />                 // HANDOFF: thu thập SĐT
    <HandoffConfirmCard />          // HANDOFF_DONE: xác nhận
    <OutOfScopeCard />              // GUARDRAIL: out-of-scope
    <HandoffButton />               // FAILURE: nút khẩn kết nối người thật
    <FeedbackModal />               // CORRECTION: dislike flow
  </MessageList>
  <ChatInput />                     // Text input + send button
</ChatWidget>
```

---

## 7. Tools / Function Calling Summary

| Tool | Gọi ở Phase | Input | Output |
|------|------------|-------|--------|
| `get_car_info()` | CAR_DISCOVERY | query, filters | CarInfo[] |
| `get_promotions()` | FINANCE_FULL_PAY | model, region | PriceDetail |
| `calculate_installment()` | FINANCE_INSTALLMENT | price, down%, term, rate | InstallmentResult |
| `save_lead()` | HANDOFF_COLLECT | name, phone, context | LeadID |

**Nguyên tắc:** LLM **không tự tính toán số tài chính**. Mọi con số phải từ tool. Nếu tool lỗi → hiển thị lỗi và chuyển handoff.

---

## 8. Red Flags & Threshold (Monitoring)

| Metric | Target | Red Flag |
|--------|--------|----------|
| Precision tài chính/chính sách | ≥ 95% | < 90% / tuần |
| Re-ask rate (mơ hồ) | ≤ 20% session | > 35% / tuần |
| Out-of-scope accuracy | ≥ 98% | < 95% / tuần |
| Handoff rate sau AI trả lời | 20–40% | > 55% kèm dislike tăng |
| SĐT conversion (Happy Path) | TBD | Theo dõi từ tuần đầu |

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PhamXuanKhang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PhamXuanKhang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
