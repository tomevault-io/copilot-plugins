## ai-meeting-assistant

> *English / [Tiếng Việt](#hướng-dẫn-dành-cho-nhà-phát-triển)*

# 🤖 AI Assistant & Developer Guide (CLAUDE.md)

*English / [Tiếng Việt](#hướng-dẫn-dành-cho-nhà-phát-triển)*

This file provides architectural guidelines, codebase principles, and setup instructions for both human developers and AI Coding Assistants (like Claude, GitHub Copilot) working on this repository.

## 🏗️ Technical Architecture & Design Decisions

### 1. Backend Migration (Node.js → Python Flask)
- **Decision:** Shifted the primary backend from Express.js to Python Flask.
- **Reason:** Python offers superior, native support for AI libraries (OpenAI, Google Generative AI, Boto3) and handles synchronous AI workflows more gracefully. The old Node.js codebase has been completely deprecated and removed to keep the repository clean.

### 2. AI Model Selection
- **Transcription:** `OpenAI Whisper` (`whisper-1`) is used for highly accurate, multilingual Speech-to-Text.
- **Summarization & Tasks:** `Google Gemini 2.5 Flash` was chosen over `1.5-pro` because it offers faster inference times for meeting summaries while maintaining excellent contextual extraction capabilities.

### 3. Graceful Fallbacks
- **Mock Mode:** If the OpenAI API key hits a rate limit or runs out of credits (`429 Quota Exceeded`), the system automatically injects a `Mock Transcript`. This ensures the UI/UX demonstration (and subsequent Gemini task extraction) never breaks during a live showcase.
- **In-Memory Storage:** If MongoDB is not configured or fails to connect, the system falls back to an in-memory dictionary. Data is ephemeral but allows immediate local testing without Docker or database setups.
- **S3 Bypass:** If AWS S3 credentials are missing, audio uploads silently bypass S3 and process locally.

---

## 💻 Developer Commands

### Environment Setup
```bash
cd backend
python -m venv .venv
# Activate venv:
# Windows: .\.venv\Scripts\activate
# Mac/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

### Running the Server
```bash
cd backend
python app.py
# Server runs on http://localhost:3000
```

### Testing the Endpoints
```bash
# Test AI parsing with a mock text
curl -X POST http://localhost:3000/api/meetings/test-ai \
  -H "Content-Type: application/json" \
  -d '{"text":"Hôm nay họp dự án mới. Minh lo làm Frontend. Lan xử lý API."}'
```

---

<br>

<h1 id="hướng-dẫn-dành-cho-nhà-phát-triển"> Hướng dẫn dành cho Nhà phát triển 🇻🇳 </h1>

Tài liệu này cung cấp cái nhìn tổng quan về kiến trúc, các quyết định kỹ thuật và tiêu chuẩn code dành cho các lập trình viên cũng như các Trợ lý AI khi tham gia phát triển dự án này.

## 🏗️ Các Quyết định Kỹ thuật (Design Decisions)

### 1. Chuyển đổi Backend (Node.js → Python Flask)
- **Quyết định:** Đã loại bỏ hoàn toàn backend Node.js (Express) cũ và chuyển sang dùng Python Flask.
- **Lý do:** Hệ sinh thái Python hỗ trợ native và tối ưu hơn rất nhiều cho các thư viện AI (OpenAI, Google GenAI). Việc xử lý luồng (workflow) Audio -> Text -> AI Summary cũng mạch lạc và dễ bảo trì hơn.

### 2. Lựa chọn Mô hình AI (AI Models)
- **Nhận diện giọng nói:** Dùng `OpenAI Whisper` vì khả năng nhận diện tiếng Việt cực kỳ chính xác.
- **Tóm tắt & Tạo Task:** Chọn `Google Gemini 2.5 Flash` thay vì `1.5-pro` vì tốc độ phản hồi nhanh hơn vượt trội mà vẫn giữ được chất lượng bóc tách công việc (Action Items) xuất sắc.

### 3. Cơ chế Dự phòng (Graceful Fallbacks)
Dự án được thiết kế để không bao giờ "chết" trong lúc Demo:
- **Mock Mode (Dữ liệu giả lập):** Nếu API Key của OpenAI hết tiền (Lỗi 429), hệ thống sẽ tự động bypass và nạp một đoạn văn bản giả lập. Gemini sẽ tóm tắt văn bản đó để người dùng vẫn xem được trọn vẹn luồng UI/UX.
- **In-Memory Database:** Nếu không có kết nối MongoDB, hệ thống tự động lưu trữ trên RAM (In-Memory). Dữ liệu sẽ mất khi tắt server nhưng vô cùng tiện lợi cho việc test nhanh.
- **Bỏ qua S3:** Nếu chưa cấu hình AWS S3, tính năng lưu trữ audio đám mây sẽ tự động bị bỏ qua mà không gây văng app.

---

## 💻 Các Lệnh Hữu Ích

### Cài đặt môi trường
```bash
cd backend
python -m venv .venv
# Kích hoạt venv (Windows): .\.venv\Scripts\activate
pip install -r requirements.txt
```

### Khởi động Server
```bash
cd backend
python app.py
# Server chạy tại http://localhost:3000
```

*Nếu bạn là AI Coding Assistant, hãy luôn tham khảo tài liệu này trước khi thay đổi kiến trúc hoặc thư viện của dự án.*

---
> Source: [nghia2207/AI-Meeting-Assistant](https://github.com/nghia2207/AI-Meeting-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
