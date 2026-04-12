## smartshop-fontend

> Tài liệu này định nghĩa các quy tắc và quy ước cốt lõi cho dự án. Vui lòng tuân thủ các hướng dẫn này để đảm bảo code nhất quán và dễ bảo trì.

# Quy Tắc và Hướng Dẫn cho Dự Án ReactJS

Tài liệu này định nghĩa các quy tắc và quy ước cốt lõi cho dự án. Vui lòng tuân thủ các hướng dẫn này để đảm bảo code nhất quán và dễ bảo trì.

## 1. Nguyên Tắc Chung

- **Ngôn ngữ**: Toàn bộ code (tên biến, hàm, component, props) và bình luận phải được viết bằng **tiếng Anh**.
- **Định dạng code**: Sử dụng Prettier để tự động định dạng code. Cấu hình được định nghĩa trong `@/.prettierrc`.
- **Tuân thủ mẫu hiện có**: Trước khi viết code mới, hãy tìm kiếm các thành phần hoặc logic tương tự đã có trong dự án và làm theo mẫu đó.

## 2. Cấu trúc Component

- **Quy ước đặt tên**: Tên component dùng `PascalCase`. Tên tệp phải khớp với tên component.
    - Ví dụ đúng: component `UserProfile` nằm trong tệp `UserProfile.jsx`.
- **Functional Components**: Luôn sử dụng functional component với Hooks. Không sử dụng class component.
- **Tái sử dụng**:
    - Các component UI chung, có thể tái sử dụng trên toàn bộ ứng dụng, phải được đặt trong `@/components/common/`.
    - Khi tạo một nút (button) mới, hãy tuân theo cấu trúc và các props của component `@/components/common/Button.jsx`.
- **Props**: Luôn định nghĩa kiểu dữ liệu cho props bằng PropTypes hoặc TypeScript. Tham khảo cách component `@/components/common/Card.jsx` định nghĩa `propTypes` của nó.

## 3. Quản lý State (State Management)

- **State cục bộ**: Sử dụng hook `useState` cho các state đơn giản, chỉ thuộc về một component.
- **State phức tạp**: Đối với các logic state phức tạp hơn (nhiều hành động, state tiếp theo phụ thuộc vào state trước), hãy sử dụng `useReducer`.
- **State toàn cục**:
    - Sử dụng React Context API để quản lý state toàn cục như thông tin người dùng hoặc chủ đề (theme).
    - **Không** sử dụng Redux trừ khi được yêu cầu.
    - Để biết cách thêm một giá trị mới vào context người dùng, hãy xem tệp `@/contexts/AuthContext.js`.

## 4. Styling

- **Phương pháp**: Chúng ta sử dụng **CSS Modules** để styling. Điều này giúp tránh xung đột CSS giữa các component.
- **Quy ước**: Mỗi component phải có một tệp `[ComponentName].module.css` tương ứng.
    - Hãy xem cách `@/components/features/Header/Header.jsx` nhập và sử dụng styles từ `@/components/features/Header/Header.module.css`.
- **Biến màu và Font**: Các giá trị màu sắc, font chữ, và khoảng cách chung được định nghĩa trong `@/styles/globals.css`. Luôn sử dụng các biến CSS này thay vì các giá trị tĩnh (hard-code).

## 5. Custom Hooks

- **Mục đích**: Tách các logic có thể tái sử dụng (ví dụ: gọi API, xử lý form) ra các custom hook.
- **Vị trí**: Tất cả các custom hook phải được đặt trong thư mục `@/hooks/`.
- **Ví dụ**: Khi cần lấy dữ liệu từ API, hãy sử dụng hoặc tham khảo hook `@/hooks/useFetch.js` thay vì viết logic `fetch` trực tiếp trong component.

## 6. Gọi API

- **Vị trí**: Logic gọi API được tập trung trong thư mục `@/services/`.
- **Quy tắc**: Component không được gọi `axios` hoặc `fetch` trực tiếp. Thay vào đó, chúng phải gọi các hàm đã được định nghĩa trong các tệp services.
- **Tham khảo**: Xem cách `@/services/userService.js` định nghĩa các hàm để tương tác với API người dùng.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/An8bit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/An8bit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
