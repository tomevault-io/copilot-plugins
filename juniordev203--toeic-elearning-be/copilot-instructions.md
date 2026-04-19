## toeic-elearning-be

> Dưới đây là các quy tắc bắt buộc để đảm bảo tính đồng bộ dữ liệu và Type Safety với Frontend:

# Quy tắc Code Backend (NestJS) - Dự án Toeic Dictation

Dưới đây là các quy tắc bắt buộc để đảm bảo tính đồng bộ dữ liệu và Type Safety với Frontend:

## 1. Request & Response DTOs
- **Mọi endpoint** phải có Request DTO (`@Body`, `@Query`) và **Response DTO**.
- Không bao giờ trả về trực tiếp Entity hoặc kiểu `any/object`.
- Response DTO phải đặt trong thư mục `dto/` của module và có hậu tố `-response.dto.ts`.

## 2. Swagger Documentation (OpenAPI)
- Sử dụng đầy đủ các decorator từ `@nestjs/swagger`.
- **Bắt buộc** có `@ApiResponse({ status: ..., type: ... })` cho mọi endpoint. Đây là điều kiện tiên quyết để frontend (Orval) có thể sinh code chính xác.
- Sử dụng `@ApiProperty` cho mỗi trường trong DTO.
- Đối với Enum, phải chỉ rõ: `@ApiProperty({ enum: MyEnum, example: MyEnum.VALUE })`.

## 3. Dictation Mode
- Sử dụng enum `DictationMode` (`KEYWORD`, `PHRASE`, `FULL_SENTENCE`) thay vì cấp độ khó Easy/Normal/Hard.

## 4. Response Wrapping
- Backend sử dụng `TransformInterceptor` để bọc dữ liệu vào `{ success, data, message }`. 
- Trong Swagger `@ApiResponse`, chỉ khai báo `type` là dữ liệu thực tế (ví dụ `LoginResponseDto`). Frontend sẽ nhận diện được thông qua mutator.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juniordev203) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
