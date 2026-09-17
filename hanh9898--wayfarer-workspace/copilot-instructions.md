## wayfarer-workspace

> <!-- bmad:context -->

<!-- bmad:context -->
<!-- Verified 2026-08-27. Workspace root ở HEAD ff7ac81; van-dao/ đối chiếu ở HEAD a4cb1cb. Managed by bmad-project-context; sửa trong khối này sẽ bị ghi đè ở lần refresh sau. Nội dung ngoài marker được giữ nguyên. -->

## van-dao

Plugin `van-dao`: biến sách PDF/EPUB người dùng đã có thành lộ trình học có người kèm. Mã nguồn ở
`van-dao/` (repo git riêng, **chỉ chứa mã nguồn** — không CLAUDE.md, không tài liệu). Toàn bộ tài liệu,
đặc tả, và điều phối làm việc (kể cả sửa code trong `van-dao/`) đều thực hiện từ workspace BMAD này.

Thiết kế: `docs/VAN-DAO-dac-ta-v1.0.md` — 2.112 dòng, mở theo mục, bắt đầu ở §0 (bảng tra từ vựng), đừng
nạp cả file. Đang ở đâu, làm gì tiếp: `docs/VAN-DAO-trang-thai-du-an.md` §10.

## Policy

- **IMPORTANT: không bao giờ commit nội dung sách hoặc hồ sơ người học vào `van-dao/`.** Chúng ở
  `~/.vandao/` trên máy người dùng; `van-dao/.gitignore` chặn `*.pdf` · `*.epub` · `.vandao/`. Đừng tạo
  file mẫu vi phạm việc này.
- Đặc tả (`docs/VAN-DAO-dac-ta-v1.0.md`) là nguồn sự thật. Mã lệch đặc tả thì sửa đặc tả trước, sửa mã
  sau.
- Không thêm thành phần ngoài §13 đặc tả mà chưa hỏi.
- Không viết thêm design hay data contract trước khi có executable spec — viết just-in-time theo vòng.
  Căn cứ: `docs/VAN-DAO-trang-thai-du-an.md` §4.2 — bốn lần soát tài liệu bắt 0/11 defect, walkthrough
  chạy ca thật bắt 11/11.
- Không làm việc trực tiếp bên trong `van-dao/` (không mở phiên riêng cd vào đó) — mọi thay đổi cho
  `van-dao/` khởi tác từ workspace này.

## Where things are

- Mã nguồn plugin: `van-dao/` — repo git riêng, không có CLAUDE.md hay docs/ nội bộ
- Đặc tả & trạng thái dự án: `docs/`
- Data contract một bí kíp: `van-dao/tham-chieu/bi-kip.schema.md` — đi cặp với
  `van-dao/bin/kiem-bi-kip.py` (§6 của schema là luật bằng lời của đúng những phép kiểm script thực thi,
  §7 là phần script không làm — đổi một bên thì đổi bên kia trong cùng một thay đổi, kèm fixture và test)
- Eval skill/agent: `ARCHITECTURE-SPINE.md` AD-10 (nhẹ lúc viết, đầy đủ trước khi phát hành) + AD-11
  (skill tự đủ, không trích FR/NFR/§) — mẫu chi tiết: `docs/VAN-DAO-setup-du-an.md` Phần 4. Kiểm hai AD
  này có thật sự được tuân thủ trong một `SKILL.md`/`evals/` cụ thể: lens `skill-quality`
  (`_bmad/custom/lenses/skill-quality.md`, đăng ký ở `_bmad/custom/bmad-review.toml`), gọi qua
  `bmad-review lenses=skill-quality`.
- Artifact kế hoạch BMAD: `_bmad-output/planning-artifacts`, `_bmad-output/implementation-artifacts`
- Kiến trúc ràng buộc mọi epic/story: `_bmad-output/planning-artifacts/architecture/architecture-van-dao-workspace-2026-08-25/ARCHITECTURE-SPINE.md`
  — 11 invariant (AD-1..AD-11)

## Running and verifying

- `cd van-dao && pip install -r requirements-dev.txt && python -m pytest tests/ -v` — đúng lệnh CI chạy
  (Python 3.11, cả `ubuntu-latest` và `windows-latest`)
- `python van-dao/bin/kiem-bi-kip.py --kho van-dao/tests/fixtures` — thoát mã 1 là ĐÚNG, fixture
  `sach-loi` cố ý chứa lỗi; `<quyển> --json` cho đầu ra máy đọc (0 đạt · 1 có lỗi · 2 không chạy được)
- `claude plugin validate van-dao --strict` — chạy sau mỗi lần đụng `van-dao/.claude-plugin/` hay thêm
  skill
- Script BMAD ở đây (`_bmad/scripts/*.py` qua `uv run`) trên Windows phải kèm `PYTHONIOENCODING=utf-8`,
  vd: `PYTHONIOENCODING=utf-8 uv run _bmad/scripts/resolve_config.py --project-root .` — thiếu biến này
  sẽ crash `UnicodeEncodeError` vì stdout mặc định code page cp1252, không xử lý được Unicode trong output
- Eval hành vi thật (`claude -p`) cho skill có ghi file ra ngoài thư mục dự án (mọi skill ghi vào
  `~/.vandao/...`) cần CẢ HAI `--add-dir <home cô lập>` và `--permission-mode acceptEdits` ở đúng lượt
  ghi — thiếu `--add-dir` thì bị chặn "ngoài phạm vi thư mục dự án"; có `--add-dir` mà thiếu
  `--permission-mode` thì bị chặn "chưa được cấp quyền ghi" (headless không có TTY để tự duyệt)

## Conventions that differ from defaults

- Định danh trong mã, tên file, tên trường YAML dùng tiếng Việt **không dấu**; kebab-case cho file và
  `id`, snake_case cho trường YAML.
- Mọi script trong `van-dao/bin/` phải kèm luật tương đương bằng lời trong `van-dao/tham-chieu/`, và
  **không được phụ thuộc code page của máy chạy**: mở file khai `encoding="utf-8"`, ép
  `sys.stdout`/`sys.stderr` về UTF-8.
- Mọi SKILL.md trong `van-dao/skills/` phải có "Xong khi", "Khi nào skill này không giúp được", trình
  tự "Trình → xác nhận → ghi → kiểm", và (nếu có field trong `customize.toml`) bước nạp field đó trước
  việc chính (§12.4 đặc tả).
- Mọi ví dụ trong đặc tả và `van-dao/tham-chieu/` phải là ví dụ tổng hợp, không lấy từ bí kíp thật (R24).
- Yêu cầu ở đặc tả §8 gồm **34 mục**: `R1`–`R31` cộng `R13b`, `R32`, `R33`. Sửa hành vi thì trỏ đúng R nào.
- Tham chiếu tương đối từ file trong `van-dao/` sang `docs/` ở workspace này phụ thuộc độ sâu (`../docs/`
  từ gốc `van-dao/`, `../../docs/` từ thư mục con cấp 1). Không áp dụng cho `skills/**`/`agents/**`/
  `commands/**` — AD-11 cấm hẳn các file đó tham chiếu `docs/`/`_bmad-output/`.
- Tên vai hiển thị (Trưởng môn, Nghiệm Công Sứ, Phúc Khảo Sứ, Sơn phong trưởng lão...) và định danh kỹ
  thuật kebab-case (`truong-mon`, `nghiem-cong`, `phuc-khao`, `truong-lao`...) là hai lớp tách biệt — đổi
  tên vai trong văn xuôi/tài liệu không kéo theo đổi file, `id`, hay identifier kỹ thuật.
- Luật mới cần kiểm bằng script (không riêng `bi-kip.schema.md`↔`kiem-bi-kip.py`) phải đi cặp: đoạn luật
  bằng lời trong tài liệu tham chiếu + test dương/test âm trong cùng một thay đổi, và luật đó bắt nguồn
  từ đâu (R-nào của đặc tả, hay quyết định nào) phải nêu rõ — không âm thầm thêm luật không truy được
  nguồn.

## Known pitfalls

- **Rỗng ≠ vắng** (R13). `chi_nhanh: []` hợp lệ; `sai_lam_pho_bien: []` chỉ là cảnh báo. Defect đầu tiên
  của dự án, đã có regression test khoá lại.
- Validator có ba mức — `L()` lỗi chặn nhập kho · `C()` cảnh báo · `G()` ghi chú — và chỉ `loi` quyết
  định mã thoát. Chọn nhầm mức là cách âm thầm nhất làm hỏng luồng thu sách.
- Bí kíp có hai hình dạng rẽ ở `kiem_mot()`: `loai: bi-kip` có `chuong/` và đồ thị phụ thuộc;
  `loai: tan-quyen` không có `chuong/`, trường sư phạm nằm thẳng trong `manifest.yaml`.
- Ba đường truyền tin định nghĩa thiết kế bằng thứ **không** mang theo (đặc tả §4.1): thư linh → Nghiệm
  Công Sứ không mang giáo án; giám khảo → Phúc Khảo Sứ không mang tiêu chí; trưởng môn → trưởng lão không
  mang `thu-linh/**`. Bốn vai chấm là subagent **tươi**, không bao giờ fork.
- `.pham-vi.json` thiếu → **không vai nào** đọc được, không phải mọi vai đọc được (R26). Fail an toàn.
- `van-dao/tests/test_kiem_bi_kip.py` gọi script qua `subprocess`, đọc `--json` — kiểm cả giao diện
  lệnh. Thêm phép kiểm mới thì thêm ca lỗi vào fixture `sach-loi`, assert theo **mã lỗi** (`ma`), không
  theo thông điệp.
- "Bậc" (đặc tả §13.1 — độ hoàn thiện tính năng) và "Vòng" (đặc tả §16 — thứ tự phụ thuộc dựng thật) là
  hai trục khác nhau, từng lệch nhau thật (`phuc-khao` bị gắn Bậc 3 dù đã cần dùng ở Vòng 2). Hỏi "cái gì
  dựng trước" hay "MVP dừng ở đâu" thì dùng Vòng, không dùng Bậc.
- Mermaid `stateDiagram-v2` không cho nối chuỗi `A --> B --> C` trên một dòng như `flowchart` cho phép —
  parse-fail thật đã gặp ở `ARCHITECTURE-SPINE.md` (2026-08-27); viết mỗi cạnh một dòng riêng.
- Edit nối tiếp theo mục đánh số (Epic N, Story N.M...) vào file lớn với anchor text lặp lại đã từng ghi
  nhầm vị trí thật (`epics.md`: thứ tự Epic Details xáo trộn, AC lạc sang story khác, phải Write lại cả
  file để sửa) — đảm bảo anchor đủ duy nhất, hoặc đọc lại file sau mỗi lần nối.
- Tài liệu quy ước (kể cả dạng `.claude/rules/*.md`, dù có cơ chế path-scoped auto-load hợp lệ của
  Claude Code) tuyệt đối không đặt trong `van-dao/` — policy "chỉ chứa mã nguồn" ở trên đã ghi rõ
  nhưng vẫn bị vi phạm thật (thêm `eval.md` + 1 file mới vào đó, phải gỡ ra sau). Quy ước mới cho
  skill/eval luôn vào `ARCHITECTURE-SPINE.md` (qua `bmad-architecture`) hoặc `docs/`, không bao giờ
  vào `van-dao/`.
- Đổi tên bề mặt gọi lệnh (tiền tố plugin, tên slash command...) làm bằng chứng eval CŨ hoá lỗi thời ở
  MỌI `evals.json` có trích transcript cũ, không chỉ file của skill đang đổi tên — bắt được thật: đổi
  `/vd:` → `/van-dao:` đã sửa hết `SKILL.md`/tài liệu nhưng bỏ sót `nhap-mon/evals/evals.json` (đã
  `done`) vẫn trích `/vd:` trong 2 transcript thật, chỉ lộ ra khi lens `skill-quality` chạy trên MỘT
  skill khác rồi lần ngược lại. Đổi tên bề mặt gọi lệnh thì `grep` chuỗi cũ trên toàn bộ
  `van-dao/skills/**/evals/` trước khi coi là xong.

<!-- /bmad:context -->

---
> Source: [hanh9898/wayfarer-workspace](https://github.com/hanh9898/wayfarer-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
