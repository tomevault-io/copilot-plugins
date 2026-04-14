## dat-game-khkt

> Nghiên cứu hành vi và ngôn ngữ của người chơi khi họ trải nghiệm cảm xúc ức chế hoặc thất bại trong trò chơi. Tập trung vào việc phát hiện, phản ứng và hướng dẫn điều chỉnh cảm xúc qua ngôn ngữ mà người chơi nhập vào.

# 🧠 Dự án Khoa học Xã hội & Hành vi: Phản ứng người chơi khi bị ức chế trong game

## Mục tiêu

Nghiên cứu hành vi và ngôn ngữ của người chơi khi họ trải nghiệm cảm xúc ức chế hoặc thất bại trong trò chơi. Tập trung vào việc phát hiện, phản ứng và hướng dẫn điều chỉnh cảm xúc qua ngôn ngữ mà người chơi nhập vào.

## Giả thuyết nghiên cứu

- Người chơi có xu hướng sử dụng ngôn ngữ tiêu cực hoặc xúc phạm nhiều hơn khi bị thua liên tục.
- Một chatbot AI xuất hiện để khuyên nhủ có thể giúp giảm mức độ tiêu cực và tăng khả năng tự điều chỉnh cảm xúc của người chơi.

## Cấu trúc demo (Frontend-only)

1. Trò chơi mini (gây ức chế): game đơn giản, dễ thua. Sau khi người chơi thua → hiển thị màn hình bình luận.
2. Hộp bình luận: nhận diện từ ngữ tiêu cực/ tục tĩu (client-side) và bật chatbot ở góc màn hình với lời nhắn nhẹ nhàng.
3. Chatbot phản hồi: giả lập stream/WebSocket (không cần backend thật) để hiển thị phản hồi theo thời gian thực.
4. Giao diện kết quả (tùy chọn): thống kê số từ, số từ tiêu cực, số lần chatbot can thiệp.

## Công nghệ đề xuất

- Frontend: Next.js/React
- Fake API: WebSocket mock hoặc timer random
- Style: TailwindCSS hoặc CSS thuần
- Data: danh sách từ tiêu cực/regex ở client

## Lưu ý đạo đức

- Thêm thông báo: “Đây là mô phỏng nghiên cứu hành vi, không lưu dữ liệu cá nhân.”
- Game không nên quá khó hoặc gây tổn thương thật.
- Mục tiêu là quan sát phản ứng, không kích động tiêu cực.

Các tệp hiện có liên quan: [app/page.tsx](mdc:app/page.tsx), [app/globals.css](mdc:app/globals.css), [postcss.config.mjs](mdc:postcss.config.mjs), [next.config.ts](mdc:next.config.ts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndyAnh174)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AndyAnh174)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
