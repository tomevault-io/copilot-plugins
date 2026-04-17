## hyperguard

> HyperGuard là một giải pháp bảo vệ mã nguồn Android toàn diện, kết hợp giữa việc chuyển đổi Dalvik bytecode sang mã Native (C++) và các lớp bảo vệ nâng cao bằng Rust. Dự án này được thiết kế để chống lại các nỗ lực dịch ngược, gỡ lỗi và chỉnh sửa ứng dụng trái phép.

# HyperGuard: Công cụ bảo mật APK Android (Industrial Grade)

HyperGuard là một giải pháp bảo vệ mã nguồn Android toàn diện, kết hợp giữa việc chuyển đổi Dalvik bytecode sang mã Native (C++) và các lớp bảo vệ nâng cao bằng Rust. Dự án này được thiết kế để chống lại các nỗ lực dịch ngược, gỡ lỗi và chỉnh sửa ứng dụng trái phép.

## 🏗 Kiến trúc dự án

- **DEX-to-C++ (Core):** Chuyển đổi logic Java/Kotlin nhạy cảm sang mã nguồn JNI C++ để ngăn chặn các công cụ như JADX.
- **Rust Native Shield:** Lớp bảo vệ bổ sung bằng Rust (khó dịch ngược hơn C++), xử lý các tác vụ kiểm tra an ninh quan trọng.
- **Virtual Machine Protection (VMP):** Thực thi logic trong một máy ảo tùy chỉnh (Custom Interpreter) với tập lệnh riêng.
- **Anti-Reverse Engineering:** Tích hợp sẵn các kỹ thuật chống Debug (ptrace) và chống Frida (memory scanning).
- **UI Security:** Bảo vệ lớp hiển thị (chống chụp màn hình, chống Tapjacking).

## 📂 Các thành phần chính

- `HyperGuard.py`: Entry point chính của công cụ.
- `HyperGuard/`: Thư viện xử lý bytecode và tạo mã C++.
- `project/rust-shield/`: Module bảo mật viết bằng Rust.
- `project/jni/`: Mã nguồn C++ cho thư viện native.
- `tools/`: Các công cụ hỗ trợ như `apktool`, `apksigner`.
- `androguard/`: Thư viện phân tích DEX được nhúng sẵn.

## 🚀 Cài đặt và Sử dụng

### Tiền đề
- Python 3.8+
- Android NDK (Đã cấu hình trong `HyperGuard.cfg`)
- Rust & `cargo-ndk` (Nếu muốn build lại module Rust)
- JRE/JDK 11

### Các lệnh quan trọng

- **Bảo vệ APK:**
  ```bash
  python3 HyperGuard.py -a input.apk -o output.apk
  ```
- **Biên dịch Module Rust:**
  ```bash
  cd project/rust-shield
  cargo ndk -t arm64-v8a build --release
  ```
- **Làm sạch file tạm:**
  Công cụ tự động xóa thư mục `.tmp` sau khi hoàn tất. Nếu cần xóa thủ công: `rm -rf .tmp`.

## 🛡 Các Module Bảo mật (Industrial Grade)

1.  **Module 1: Native Shielding** (Rust): Ẩn danh hàm JNI bằng `RegisterNatives` và kiểm tra SHA-256 APK.
2.  **Module 2: Virtualization (VMP)**: Trình thông dịch logic bằng Opcode ảo.
3.  **Module 3: Anti-Debug & Anti-Frida**: Chặn debugger và quét sự hiện diện của Frida trong bộ nhớ.
4.  **Module 5: String Encryption**: Giải mã chuỗi "vừa đủ dùng" (Just-in-time) bằng XOR.
5.  **Module 6: UI Security**: Sử dụng `FLAG_SECURE` và phát hiện Overlay.

## 📝 Quy ước Phát triển

- **Native Code:** Ưu tiên viết các logic bảo mật mới bằng Rust thay vì C++.
- **JNI Bridge:** Luôn sử dụng `com.hyperguard.NativeLoader` để giao tiếp giữa Java và Native.
- **Quy trình Build:** Luôn kiểm tra tính toàn vẹn của APK trước khi thực thi các logic quan trọng.
- **Dọn dẹp:** Đảm bảo không để lại dấu vết mã nguồn trung gian trong thư mục `.tmp`.

## ⚙️ Cấu hình (`HyperGuard.cfg`)
Chỉnh sửa file này để thiết lập đường dẫn `apktool`, `ndk_dir` và thông tin chữ ký APK (`keystore`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/plongdevs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
