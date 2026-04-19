## subtitle-draft-gmtpc

> ﻿# Changelog - Subtitle Draft GMTPC

﻿# Changelog - Subtitle Draft GMTPC

Tất cả thay đổi đáng chú ý của project sẽ được ghi lại trong file này.

**⚠️ Lưu ý:** Thứ tự entries từ TRÊN xuống DƯỚI = từ MỚI đến CŨ
- Entry đầu tiên (trên cùng, ngay sau header) = MỚI nhất
- Entry cuối cùng (dưới cùng) = CŨ nhất
- LUÔN chèn entry mới lên ĐẦU file (ngay sau phần header này)

---
## [2026-04-11 04:30:00 PM Saturday]

### Added - Tab Text to Subtitle (✂️ Text to Subtitle)
- **Tab mới**: Tự động chia văn bản dài thành phụ đề ASS format
- **Panel 1**: Input văn bản gốc (Full Text Input)
- **Panel 2**: Settings + Output ASS
- **Max Characters**: Giới hạn ký tự tối đa mỗi subtitle (default: 500), tự tìm dấu chấm câu gần nhất để cắt trọn vẹn
- **CPS (Characters Per Second)**: Tính toán duration mỗi dòng dựa trên CPS (default: 17.0)
- **Punctuation mode**: Count all characters / Ignore punctuation - liên kết với CPS calculation
- **Gap (ms)**: Khoảng cách thời gian giữa các dòng phụ đề (default: 200ms)
- **Thuật toán split**: Ưu tiên cắt tại dấu câu (. ! ? 。) để câu trọn vẹn, fallback khoảng trắng, fallback cứng tại giới hạn
- **Auto-convert**: Debounce 300ms sau khi ngừng typing
- **Stats**: Hiển thị số segments, max chars/segment, tổng chars, CPS trung bình, tổng thời lượng
- **Search support**: Ctrl+F search trong panel input/output
- **Settings persistence**: Lưu MaxChars, CPS, Gap vào Properties.Settings
- **SearchManager**: Thêm `_searchTextToSub` cho tab mới

---
## [2026-04-09 02:00:00 PM Thursday]

### Fixed - Word Split Rules Search Reapply
- **Search tự động reaply** khi rules thay đổi (TextChanged event)
- **Search hoạt động sau Load/Reset**: Thêm `ReapplySearchAfterLoad()`
- Load file .txt xong → tự động tìm lại với search text hiện tại
- Reset rules → tự động tìm lại
- LoadKaraokeEngSplitRules: Set `_isKaraokeEngUpdating = true` để tránh trigger TextChanged loop

---
## [2026-04-09 01:45:00 PM Thursday]

### Fixed - Word Split Rules Search Highlight
- **Highlight hoạt động**: Focus TextBox rules → Select text → Scroll → Restore focus về search box sau 100ms
- **Toast notification**: Hiện số kết quả tìm được (vd: "🔍 3 kết quả cho 'white'")
- **Không tìm thấy**: Hiện "❌ Không tìm thấy '...'"
- Search giờ hoạt động đúng và có highlight rõ ràng

---
## [2026-04-09 01:30:00 PM Thursday]

### Fixed - Word Split Rules Search Issues
- **Không còn jump cursor**: Bỏ `Focus()` trong `SelectRuleSearchMatch()`
- Chỉ `Select()` text để highlight, không focus TextBox rules
- Cursor không bị nhảy lung tung khi search
- Set `_rulesSearchIndex = 0` khi tìm thấy kết quả đầu tiên

---
## [2026-04-09 01:15:00 PM Thursday]

### Added - Word Split Rules Search
- Thêm ô tìm kiếm trong panel Word Split Rules
- TextBox search + 2 nút ◀ Prev / ▶ Next
- **Auto-search**: Tìm ngay khi nhập liệu
- **Enter**: Tìm kết quả tiếp theo (wrap-around)
- **Escape**: Xóa search text và focus về TextBox rules
- Case-insensitive search
- Auto-select và scroll đến kết quả

### Changed - MainWindow.xaml
- Thêm RowDefinition cho search row trong Word Split Rules panel
- Grid chứa: TxtKaraokeEngRulesSearch + BtnSearchPrev + BtnSearchNext

### Changed - MainWindowTabKaraokeEnglish.cs
- Thêm search fields: `_rulesSearchIndex`, `_rulesSearchText`, `_rulesSearchPositions`
- `TxtKaraokeEngRulesSearch_TextChanged`: Auto-search khi nhập
- `TxtKaraokeEngRulesSearch_KeyDown`: Enter/Escape handling
- `BtnKaraokeEngRulesSearchPrev/Next_Click`: Navigation
- `FindAllRuleSearchMatches()`: Tìm tất cả vị trí
- `SelectRuleSearchMatch()`: Select và scroll đến match
- `GetLineFromPosition()`: Tính số dòng từ character position
- Thêm `using System.Windows.Input;`

### Build Result
- **Build**: SUCCESS ✅ (0 errors, 2 warnings - unused fields)

---
## [2026-04-09 01:00:00 PM Thursday]

### Fixed - Word Split Rules Case Sensitivity
- **Input case được bảo toàn** trong output, bất kể rule viết thế nào
- VD: Input="HOWLING", rule="how/ling" → Output="HOW/LING" ✅
- VD: Input="Howling", rule="how/ling" → Output="How/ling" ✅
- VD: Input="howling", rule="HOW/LING" → Output="how/ling" ✅
- Thêm hàm `AdjustSyllableCase()`: Tự động adjust syllables theo input case
- Hỗ trợ: ALL CAPS, lowercase, Title Case
- Thêm `using System.Linq;` vào KaraokeVietnameseService.cs

---
## [2026-04-09 12:45:00 PM Thursday]

### Fixed - Context Menu Placement
- Sửa vị trí context menu Load xuất hiện ngay dưới nút Load
- Thêm HorizontalOffset và VerticalOffset = 0
- Không còn hiện xa tít mù khơi nữa 😄

---
## [2026-04-09 12:30:00 PM Thursday]

### Fixed - Search Behavior
- Bỏ real-time search khi TextChanged
- Enter lần đầu: Tìm từ kết quả đầu tiên
- Enter nhiều lần: Tiếp tục tìm kết quả tiếp theo, không bị reset
- Tự động reset search khi text thay đổi
- F3 vẫn hoạt động để tìm kết quả tiếp theo

### Added - Word Split Rules Buttons (Karaoke English)
- Thêm 3 nút trong panel Word Split Rules: 🔄 Reset, 💾 Save, 📂 Load
- **Reset**: Load lại rules mặc định từ thư mục app
- **Save**: Browse chọn nơi lưu file .txt
- **Load**: Context menu với 2 tùy chọn:
  - 🔄 Load Default: Từ thư mục app
  - 📂 Load from File: Browse chọn file .txt
- Toast notification cho mỗi thao tác

### Changed - MainWindow.xaml
- Thêm RowDefinition cho buttons row trong panel Word Split Rules
- WrapPanel chứa 3 nút Reset/Save/Load

### Changed - MainWindowTabKaraokeEnglish.cs
- `BtnKaraokeEngRulesReset_Click`: Reset về rules mặc định
- `BtnKaraokeEngRulesSave_Click`: SaveFileDialog lưu .txt
- `BtnKaraokeEngRulesLoad_Click`: ContextMenu với Load Default/Load from File
- Xử lý encoding UTF-8 khi save/load

### Changed - MainWindowSystemGlobalEvents.cs
- Thêm field `_lastSearchText` để track text thay đổi
- `PerformSearch()`: Reset SearchManager khi text thay đổi
- `TxtSearchBox_KeyDown()`: Enter luôn tìm tiếp, không reset

### Build Result
- **Build**: SUCCESS ✅ (0 errors, 0 warnings)

---
## [2026-04-09 12:00:00 PM Thursday]

### Added - Global Search Functionality (Ctrl+F / F3)
- Thêm chức năng tìm kiếm toàn cục cho TẤT CẢ các tab
- **Ctrl+F**: Hiện/ẩn thanh tìm kiếm
- **F3**: Tìm kết quả tiếp theo
- **Enter**: Tìm kiếm nhanh từ thanh search
- **Escape**: Đóng thanh tìm kiếm
- Hỗ trợ tìm kiếm trong cả TextBox và ListBox
- Case-insensitive search (không phân biệt hoa thường)

### Added - Search Panel UI
- Thanh tìm kiếm nằm ở trên cùng, ngay phía trên toolbar
- Gồm: Ô nhập liệu + nút Prev/Next + nút đóng + hiển thị kết quả (ví dụ: "Kết quả: 2/5")
- Tự động tìm kiếm khi nhập liệu (real-time search)
- Navigation qua lại giữa các kết quả

### Added - SearchManager Helper
- `Helpers/SearchManager.cs`: Quản lý tìm kiếm riêng cho mỗi tab (13 SearchManagers)
- `SearchInTextBox()`: Tìm kiếm và select text trong TextBox
- `SearchInListBox()`: Tìm kiếm và select item trong ListBox
- Tự động wrap-around (quay lại đầu khi hết kết quả)
- Theo dõi vị trí hiện tại và tổng số kết quả

### Changed - MainWindow.xaml
- Thêm Grid.Row mới cho SearchPanel
- SearchPanel visibility toggle bằng Ctrl+F
- Controls: TxtSearchBox, BtnSearchPrev, BtnSearchNext, BtnSearchClose, TxtSearchStatus

### Changed - MainWindowSystemGlobalEvents.cs
- Thêm 13 SearchManager fields cho từng tab/sub-tab
- `MainWindow_PreviewKeyDown()`: Global key handler cho Ctrl+F và F3
- `ToggleSearchBox()`: Show/hide search panel
- `FindNext()`: Tìm kết quả tiếp theo
- `PerformSearch()`: Logic tìm kiếm thông minh theo tab hiện tại
- Tự động xác định TextBox cần tìm dựa vào tab/sub-tab đang active
- `GetSearchManagerForCurrentTab()`: Lấy SearchManager tương ứng

### Changed - Project Structure
- `Subtitle draft GMTPC.csproj`: Thêm `Helpers/SearchManager.cs`
- Build: SUCCESS ✅ (0 errors, 2 warnings - unused fields)

---
## [2026-04-08 03:00:00 PM Wednesday]

### Added - Full C# Project Conversion
- Chuyển đổi toàn bộ dự án từ VB.NET sang C# (.NET Framework 4.8)
- Tạo `Subtitle draft GMTPC.csproj` - project file C# mới
- Xóa tất cả file `.vb` (36 files) - project giờ thuần C#
- Sửa `MainWindow.xaml`: `x:Class="Subtitle_draft_GMTPC.MainWindow"` (thêm namespace đầy đủ)

### Added - New C# Files
**Models/** (6 files):
- `Models/SubtitleFormat.cs` - Enum SubtitleFormat
- `Models/AssEffectModels.cs` - AssTagType, AssTag, AssTagGroup, EffectPreset, LineEffectInfo
- `Models/PromptItem.cs` - Model prompt dịch thuật
- `Models/SubtitleLine.cs` - Abstract base class cho subtitle lines
- `Models/AssSubtitleLine.cs` - Implement cho định dạng ASS
- `Models/SrtSubtitleLine.cs` - Implement cho định dạng SRT

**Services/** (10 files):
- `Services/SubtitleParser.cs` - Parse SRT/ASS, detect format, sanitize content
- `Services/TimeCodeService.cs` - Shift time, connect gap
- `Services/MergeService.cs` - Gộp 2 file phụ đề
- `Services/TranslateService.cs` - Google Translate API
- `Services/QwenTranslateService.cs` - Qwen AI translation
- `Services/CookieManager.cs` - Cookie management
- `Services/HardwareInfoService.cs` - GPU/CPU/RAM/Mainboard info via WMI
- `Services/KaraokeVietnameseService.cs` - Xử lý karaoke tiếng Việt
- `Services/KaraokeMergeService.cs` - Merge karaoke ASS lines
- `Services/AssEffectBuilder.cs` - Build ASS override tags

**MainWindow Tab Files** (14 files):
- `MainWindowTabSubtitleMerge.cs` - Merge Eng + Viet subtitles
- `MainWindowTabSubtitleDraft.cs` - Time shift, connect gap
- `MainWindowTabDialogueOnly.cs` - Extract dialogues, manual merge
- `MainWindowTabAssFontAdjust.cs` - ASS font size adjustment
- `MainWindowTabTranslate.cs` - Translate with prompts (Google + Qwen)
- `MainWindowTabSearchFonts.cs` - Find missing fonts, install font pack
- `MainWindowTabHardwareInfo.cs` - Display hardware info
- `MainWindowTabKaraokeVietnamese.cs` - Vietnamese karaoke processing
- `MainWindowTabZeroTime.cs` - Zero time code for all lines
- `MainWindowTabKaraokeEnglish.cs` - English karaoke processing
- `MainWindowTabKaraokeMerge.cs` - Merge karaoke lines
- `MainWindowTabKaraokeSync.cs` - Sync time codes
- `MainWindowTabEffect.cs` - ASS effect builder UI
- `MainWindowSystemGlobalEvents.cs` - Global events, settings, Ctrl+Scroll

**System Files**:
- `AppSettings.cs` - Wrapper cho Properties.Settings (thay My.Settings)
- `Properties/Settings.settings` - Application settings
- `Properties/Settings.Designer.cs` - Generated settings class
- `Properties/AssemblyInfo.cs` - Assembly metadata
- `Application.xaml.cs` - Application entry point
- `MainWindow.xaml.cs` - Main window partial class

### Changed - Key Conversions
- `My.Settings.*` → `AppSettings.*` (custom wrapper)
- `vbCrLf`/`vbCr`/`vbLf` → `Environment.NewLine`/`"\r"`/`"\n"`
- `vbTab` → `"\t"`
- `CDbl()`/`CInt()`/`CLng()` → `double.Parse()`/`int.Parse()`/`(long)`
- `Select Case` → `switch`
- `For Each` → `foreach`
- `TryCast(x, T)` → `x as T`
- `If(cond, a, b)` → `cond ? a : b`
- `MustInherit`/`MustOverride` → `abstract`
- `Shared` → `static`
- `Async Sub` → `async void`
- `Handles` keyword → event registration in XAML

### Fixed
- Fix `HardwareInfoService.cs`: `CDbl(string)` → `double.Parse(string)` (4 locations)
- Fix `SubtitleFormat.SRT` → `SubtitleFormat.Srt` (C# enum PascalCase)
- Fix `SubtitleFormat.ASS` → `SubtitleFormat.Ass`
- Fix `DialogResult.OK` → `System.Windows.Forms.DialogResult.OK`
- Fix ambiguous `Clipboard` → `System.Windows.Clipboard`
- Fix ambiguous `Orientation` → `System.Windows.Controls.Orientation`
- Fix ambiguous `TextBox` → `System.Windows.Controls.TextBox`
- Add missing `using System.Linq;` for `.ToList()`, `.OfType()`, `.SelectMany()`
- Add missing `using System.Threading.Tasks;` for `Task.Delay()`
- Add `using System.Windows.Threading;` for `DispatcherTimer`
- Add `Microsoft.VisualBasic` reference to .csproj (for `Interaction.InputBox`)

### Build Result
- **Build**: SUCCESS ✅ (0 errors, 1 warning - unused field)
- **Framework**: .NET Framework 4.8
- **Language**: C# 7.3

---
## [2026-04-08 02:00:00 PM Wednesday]

### Added - Tab Effect (ASS Override Tags Visual Builder)
- Tab Effect mới trong Karaoke, layout 3 panel: Input (trái) | Effect Groups (giữa) | Output (phải)
- 7 nhóm Effect với Expander: Position & Movement, Transform & Rotation, Font & Style, Border & Shadow, Color & Alpha, Fade & Animation, Presets
- Nút 🎨 Chọn màu cạnh label Effect Groups - mở Windows Color Dialog, tự điền hex BGR vào config
- Config panel động với label tiếng Việt, hiện khi chọn effect con
- Nút Apply (dòng tại con trỏ) và Apply All (tất cả dòng)
- Nút Clear Effects (🗑️) xóa toàn bộ tags khỏi input
- 6 Presets: Fade In, Fade Out, Glow, Big Text, Center Top, Center Screen

### Added - Services & Models
- Services/AssEffectBuilder.vb: 378 dòng
  - Build tất cả ASS tags: \pos, \move, \an, \org, \frz, \frx, \fry, \fscx, \fscy, \fax, \fay, \fn, \fs, \b, \i, \u, \s, \bord, \shad, \blur, \be, \1c, \2c, \3c, \4c, \alpha, \1a-\4a, \fad, \k, \kf, \ko, \clip, \iclip
  - Parse tags từ text, extract tags, split tags
  - ApplyTagToLine: chèn tag mới vào phần TEXT của Dialogue (sau dấu , thứ 9), KHÔNG đụng time code
  - Mỗi effect = 1 block {} riêng biệt, cách nhau bởi space. VD: {\bord20} {\1c&H0000FF&} Text
  - RemoveTagByType: xóa chính xác block {} của tag cần xóa, giữ lại tags khác
  - RemoveAllTagsFromAllLines: xóa sạch tags
  - GetPresetEffects: danh sách presets thông dụng
  - ValidateTag: kiểm tra tham số hợp lệ
- Models/AssEffectModels.vb: 119 dòng
  - AssTagType enum: 40+ loại tags
  - AssTag class: đại diện 1 tag với type, raw tag, display name
  - AssTagGroup class: nhóm tags theo chức năng
  - EffectPreset class: tổ hợp tags cấu hình sẵn
  - LineEffectInfo class: thông tin effect áp dụng cho line

### Added - Code-behind
- MainWindowTabEffect.vb: 520 dòng
  - Input/Output sync với debounce 150ms
  - Handlers cho 30+ effect buttons
  - ShowEffectConfig/HideEffectConfig: quản lý config panel động
  - BuildCurrentTag: xây dựng tag từ config values
  - ApplyTagToSelectedLine: apply tag vào dòng tại con trỏ trong Output
  - ApplyTagToAllLines: apply tag vào tất cả dòng trong Output
  - BtnColorPicker_Click: mở Windows Color Dialog, điền hex BGR vào config
  - Preset handlers: Fade In/Out, Glow, Big Text, Center Top/Screen

### Changed - UI/UX
- Input luôn giữ nguyên bản gốc, KHÔNG bị thay đổi bởi Apply
- Output là nơi áp dụng hiệu ứng, tích lũy nhiều lần Apply
- Apply chọn dòng dựa trên vị trí con trỏ trong Input
- Config panel có label tiếng Việt cho mọi effect
- Toast notification thông báo kết quả mỗi thao tác

### Changed - Project Structure
- Subtitle draft GMTPC.vbproj: thêm 3 files mới + reference System.Windows.Forms
- MainWindow.xaml: thêm sub-tab Effect trong KaraokeTabControl
- MainWindowSystemGlobalEvents.vb: thêm InitializeEffectDebounce() vào MainWindow_Loaded
- MainWindow.xaml.vb: thêm Imports System

### Fixed
- Fix VB keyword conflict: đổi parameter "on" → "isEnabled" (on là keyword VB.NET)
- Fix namespace conflict: fully qualify System.Windows.MessageBox, System.Windows.Media
- Fix InsertTag: chỉ chèn vào phần text của Dialogue, không chèn vào block {} có sẵn
- Fix Regex Dialogue: \s*\d+ để bắt tùy biến số space sau "Dialogue:"
- Fix Grid RowDefinitions: ScrollViewer có MaxHeight=350 để config panel luôn hiện
- Fix WrapTag: mỗi tag có block {} riêng, không chồng vào nhau
- Fix color picker: chỉ điền hex, không auto-apply

---

## [2026-04-08 01:00:00 PM Wednesday]

### Fixed
- Sua AssEffectBuilder: Moi effect = 1 block {} rieng, VD: {\bord20} {\1c&H0000FF&} Text
- InsertTag chi chen vao phan TEXT cua Dialogue line (sau dau , thu 9), khong dong vao time code
- Them nut Clear Effects de xoa toan bo tags khoi input
- Regex Dialogue: backslash-s*backslash-d+ de bat tuy bien so space sau Dialogue:
- RemoveTagByType xoa chinh xac block {} cua tag can xoa, giu lai cac tag khac


## [2026-04-08 11:00:00 AM Wednesday]

### Added
- Tab Effect mới trong Karaoke với giao diện mở rộng theo nhóm chức năng
- Panel Input (trái) + Effect Groups (giữa) + Output (phải)
- 7 nhóm Effect với Expanders: Position & Movement, Transform & Rotation, Font & Style, Border & Shadow, Color & Alpha, Fade & Animation, Presets
- Config panel động hiện ra khi chọn effect, cho phép nhập tham số trước khi Apply
- Nút Apply (cho dòng hiện tại) và Apply All (cho tất cả dòng)
- Presets: Fade In, Fade Out, Glow, Big Text, Center Top, Center Screen
- Service AssEffectBuilder.vb: tạo tags (\pos, \move, \an, \frz, \fscx, \fax, \fn, \fs, \b, \i, \u, \s, \bord, \shad, \blur, \be, \1c, \3c, \4c, \alpha, \fad, \k, \kf, \ko)
- Service AssEffectBuilder.vb: parse tags, apply/remove tags từ dòng subtitle
- Models AssEffectModels.vb: AssTagType enum, AssTag class, EffectPreset class
- Hỗ trợ copy output và reset tags

### Changed
- MainWindowTabEffect.vb: 490 dòng với đầy đủ handlers cho từng effect button

---

## [2026-04-08 12:00:00 AM Wednesday]

### Changed
- Đổi tên `workflow.cursorrules` thành `.cursorrules` để Qwen Code tự động đọc
- Sắp xếp lại thứ tự tabs trong MainWindow.xaml
- Main tabs (non-karaoke): Hardware Info → Dialogue → Translate → Search Fonts → ASS Font Adjust → Subtitle Merge → Subtitle Draft
- Karaoke sub-tabs: Zero Time → Karaoke Vietnamese → Karaoke English → Karaoke Merge → Karaoke Sync
- Sửa lỗi thiếu closing `</Border>` tag trong Tab Subtitle Draft
- Xóa duplicate Zero Time sub-tab trong Karaoke

---

## [2026-04-07 06:00:00 PM Tuesday]

### Changed
- Sửa quy tắc changelog: Entry mới nhất ở ĐẦU file (trên cùng)
- Entry cũ nhất ở CUỐI file (dưới cùng)
- Cập nhật workflow rules và changelog.cursorrules

---

## [2026-04-07 05:53:09 PM Tuesday]

### Fixed
- Sửa logic parse Manual Input trong Tab Dialogue
- Format 3 hàng: Sửa từ lấy hàng 2 → hàng 3 (Text gốc tiếng Anh)
- Cập nhật comment trong code cho rõ ràng

---

## [2026-04-07 05:24:34 PM Tuesday]

### Changed
- Tách MainWindow.xaml.vb (2123 dòng) thành 14 files nhỏ theo tab
- Thêm workflow rules mới: Cấm code vào MainWindow.xaml.vb
- Thêm quy tắc nhận diện tab từ user input (@MainWindowTab*.vb)
- Thêm quy tắc bắt buộc ghi changelog sau mỗi lần code

### Added
- MainWindowSystemGlobalEvents.vb (136 dòng) - Window init, global events
- MainWindowTabSubtitleMerge.vb (122 dòng)
- MainWindowTabSubtitleDraft.vb (160 dòng)
- MainWindowTabDialogueOnly.vb (262 dòng)
- MainWindowTabAssFontAdjust.vb (146 dòng)
- MainWindowTabTranslate.vb (404 dòng)
- MainWindowTabSearchFonts.vb (352 dòng)
- MainWindowTabHardwareInfo.vb (93 dòng)
- MainWindowTabKaraokeVietnamese.vb (66 dòng)
- MainWindowTabZeroTime.vb (124 dòng)
- MainWindowTabKaraokeEnglish.vb (67 dòng)
- MainWindowTabKaraokeMerge.vb (65 dòng)
- MainWindowTabKaraokeSync.vb (320 dòng)

### Fixed
- Sửa lỗi imports trong các file tab mới tạo
- Sửa lỗi compile do thiếu file trong .vbproj

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghostminhtoan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
