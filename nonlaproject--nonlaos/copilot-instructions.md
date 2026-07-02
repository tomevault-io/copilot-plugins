## nonlaos

> > Luôn trả lời bằng tiếng Việt có dấu khi làm việc trong repo này.

# AGENTS.md - nonlaOS

> Luôn trả lời bằng tiếng Việt có dấu khi làm việc trong repo này.

## Mục Tiêu Dự Án

nonlaOS là distro desktop tiếng Việt dựa trên Debian stable/trixie, dùng KDE
Plasma, hướng tới người chuyển từ Windows sang Linux.

Repo này đi theo hướng package-first:

- Không sửa ISO thủ công.
- Mọi thay đổi hệ thống phải được đóng gói thành Debian package.
- Không thêm phần mềm lậu, crack, keygen hoặc asset không rõ license.
- Không lấy asset từ mạng nếu chưa xác minh nguồn và license.
- Không mở rộng scope sang ISO/live-build nếu task chỉ yêu cầu package hoặc tài
  liệu.

## License

- Source code, packaging metadata và tài liệu dùng `GPL-3.0-or-later`.
- Artwork/asset nội bộ trong `img/` và payload theme dùng `CC-BY-SA-4.0`.
- Khi thêm file mới, cập nhật `debian/copyright`, `NOTICE` hoặc tài liệu liên
  quan nếu license/source thay đổi.

## Workflow GitHub Bắt Buộc

Branch `main` được bảo vệ bằng GitHub ruleset.

Không push trực tiếp lên `main`.

Quy trình đúng:

1. Tạo branch mới từ `main`.
2. Commit thay đổi theo từng logical unit.
3. Push branch lên GitHub.
4. Mở pull request vào `main`.
5. Đợi CI `Package build / build` pass.
6. Cần ít nhất 1 review trước khi merge, trừ khi owner/admin chủ động bypass.

Repo đã bật `delete_branch_on_merge`, nên branch của PR merged sẽ tự xóa. Nếu PR
bị đóng mà không merge, workflow `Cleanup closed PR branches` sẽ tự xóa branch
đó khi branch nằm trong cùng repo.

Ví dụ:

```bash
git switch main
git pull --ff-only
git switch -c fix/package-build

# chỉnh sửa, build, lint
git add <files>
git commit -m "fix: describe the packaging change"
git push -u origin fix/package-build
gh pr create --base main --head fix/package-build
```

## Kiểm Thử Packaging

Trên Debian/WSL:

```bash
sudo apt-get update
sudo apt-get install -y build-essential devscripts dpkg-dev debhelper lintian
```

Build toàn bộ package:

```bash
./tools/build-packages.sh
```

Lint:

```bash
lintian dist/packages/*.deb
```

Hiện một số package skeleton có thể còn warning `empty-binary-package`; xem
`docs/PACKAGING_NOTES.md` trước khi xử lý.

## Quy Tắc Sửa File

- Đọc file liên quan trước khi sửa.
- Ưu tiên patch nhỏ, đúng scope.
- Không revert thay đổi của người khác nếu không được yêu cầu rõ.
- Không commit artifact build trong `dist/`.
- Nếu thay đổi `nonla-look`, chỉ dùng asset có sẵn trong `img/` trừ khi task
  yêu cầu thêm asset và license đã rõ.

## Cấu Trúc Chính

- `packages/`: Debian package source.
- `packages/nonla-desktop`: metapackage desktop stack.
- `packages/nonla-look`: wallpaper, color scheme, SDDM, Plymouth.
- `packages/nonla-repo-keyring`: public archive key cho APT repo.
- `docs/`: roadmap, testing, packaging notes, architecture.
- `tools/build-packages.sh`: build toàn bộ package ra `dist/packages/`.
- `tools/build-iso.sh`: build ISO bằng live-build.
- `tools/verify-iso-boot.sh`: kiểm boot metadata và QEMU BIOS smoke test.
- `.github/workflows/package-build.yml`: CI build và lint package.
- `.github/workflows/build-iso.yml`: CI build/sign/test/upload ISO.

## Chuẩn Tạo GitHub Release

Khi tạo release mới, dùng GitHub CLI và format Markdown chuyên nghiệp, có nút
tải rõ ràng. Không chỉ thả mỗi file/link trần.

Quy trình:

1. Xác nhận workflow ISO pass và artifact/release file đã upload lên
   SourceForge.
2. Xác nhận ISO có `SHA256SUMS` và `SHA256SUMS.gpg`.
3. Tạo tag dạng `v<version>-<channel>`, ví dụ `v0.1-alpha`.
4. Tạo release bằng `gh release create`.
5. Body release phải có nút tải, danh sách file, tính năng, kiểm chứng build và
   hướng dẫn verify.

Template release khuyến nghị:

```markdown
# nonlaOS <VERSION> <CHANNEL>

<p align="center">
  <a href="https://sourceforge.net/projects/nonlaos/files/nonlaOS-<VERSION>-<CHANNEL>-amd64.iso/download">
    <img src="https://img.shields.io/badge/T%E1%BA%A3i%20ISO-SourceForge-2ea44f?style=for-the-badge&logo=sourceforge" alt="Tải ISO từ SourceForge">
  </a>
</p>

> Bản phát hành desktop tiếng Việt dựa trên Debian stable/trixie + KDE Plasma.

## Tải Xuống

| File | Mô tả |
| --- | --- |
| [`nonlaOS-<VERSION>-<CHANNEL>-amd64.iso`](https://sourceforge.net/projects/nonlaos/files/nonlaOS-<VERSION>-<CHANNEL>-amd64.iso/download) | ISO live/install amd64 |
| [`SHA256SUMS`](https://sourceforge.net/projects/nonlaos/files/SHA256SUMS/download) | Checksum SHA-256 |
| [`SHA256SUMS.gpg`](https://sourceforge.net/projects/nonlaos/files/SHA256SUMS.gpg/download) | Chữ ký checksum |

## Điểm Nổi Bật

- Debian stable/trixie base.
- KDE Plasma desktop.
- Giao diện nhận diện nonlaOS đầu tiên: wallpaper, color scheme, SDDM, Plymouth.
- Bộ gõ tiếng Việt FCITX5 + Unikey.
- Firefox ESR, LibreOffice và app desktop cơ bản.
- APT repo metadata được ký bằng nonlaOS Archive Signing Key.
- ISO có `SHA256SUMS` và `SHA256SUMS.gpg`.
- CI kiểm boot metadata và QEMU BIOS smoke test.

## Kiểm Tra File Tải Về

```bash
gpgv --keyring nonla-archive-keyring.gpg SHA256SUMS.gpg SHA256SUMS
sha256sum -c SHA256SUMS
```

## Trạng Thái Build

- Package build: pass.
- ISO build: pass.
- `file`: ISO bootable.
- `isoinfo`: có El Torito boot catalog.
- `xorriso`: có El Torito + MBR isohybrid.
- QEMU BIOS smoke test: pass.

## Lưu Ý

- Đây là bản alpha, ưu tiên test trong VirtualBox/QEMU trước.
- Chưa hỗ trợ Secure Boot.
- Calamares config vẫn đang ở mức MVP.
```

Với release nonlaOS 0.1 alpha hiện tại, URL tải ISO chính là:

```text
https://sourceforge.net/projects/nonlaos/files/nonlaOS-0.1-alpha-amd64.iso/download
```

---
> Source: [NonLaProject/nonlaOS](https://github.com/NonLaProject/nonlaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
