## khoa-ung-dung-android-tu-xa

> Hướng dẫn chung cho AI agent khi làm việc trong một repository code.

# AGENTS.md

Hướng dẫn chung cho AI agent khi làm việc trong một repository code.

## Mục tiêu

- đọc trước khi sửa
- không đoán mò
- thay đổi tối thiểu
- verify trước khi kết luận
- báo rõ phần đã làm, đã kiểm tra, và phần còn chưa chắc chắn

Repository instruction, system instruction, developer instruction, và user instruction luôn có độ ưu tiên cao hơn file này.

## Quy ước ngôn ngữ

- Luôn giao tiếp với người dùng bằng tiếng Việt có dấu, trừ khi người dùng yêu cầu ngôn ngữ khác.
- Khi viết nội dung hiển thị cho người dùng, helper text, label, placeholder, thông báo lỗi, mô tả tính năng hoặc tài liệu nội bộ: ưu tiên tiếng Việt có dấu, rõ ràng, tự nhiên.
- Chỉ dùng không dấu khi thật sự bắt buộc cho dữ liệu kỹ thuật như slug, tên file, key cấu hình, tên biến, command, hoặc chuỗi hệ thống không hỗ trợ dấu.

## Quy tắc bắt buộc trước khi làm code

### 1. Bắt buộc đọc skill trước khi làm tác vụ code không tầm thường

Mọi tác vụ code không tầm thường đều phải đọc và áp dụng skill `buff-nao-gpt` trước khi tiếp tục.

Phạm vi áp dụng bao gồm:

- đọc và phân tích code
- tìm file liên quan
- debug
- review
- sửa bug
- thêm tính năng
- refactor
- thay đổi API, validation, upload, admin action, deploy flow, data flow, hoặc logic nghiệp vụ

Trình tự này là bắt buộc:

1. đọc file skill `buff-nao-gpt`
2. xác nhận trong câu đầu tiên của phần làm việc rằng skill đang được áp dụng
3. rồi mới được đọc file code, sửa code, chạy verify

Nếu chưa đọc skill thì:

- không được suy luận từ trí nhớ
- không được sửa code
- không được review code
- không được đề xuất bản vá như thể đã nắm rõ hệ thống

Nếu không tìm thấy skill hoặc không đọc được skill:

- phải dừng ngay
- phải báo đây là blocker
- không được tự ý dùng workflow thay thế

Yêu cầu về thông điệp đầu tiên cho tác vụ code:

- phải nói rõ đang áp dụng `$buff-nao-gpt`
- phải xuất hiện trước mọi phân tích kỹ thuật đáng kể

Nếu vi phạm mục này thì toàn bộ phần làm việc sau đó được xem là không tuân thủ instruction của repository.

### 2. Quy trình làm việc chuẩn cho tác vụ code

Làm theo thứ tự:

1. đọc skill bắt buộc
2. nói rõ đang áp dụng skill
3. đọc file liên quan
4. xác định đúng đường lỗi hoặc mục tiêu thay đổi
5. chọn thay đổi nhỏ nhất nhưng an toàn
6. sửa code
7. verify bằng cách sát lỗi nhất có thể
8. chỉ kết luận sau khi đã verify

## Quy tắc sửa code

- ưu tiên sửa nguyên nhân gốc thay vì vá bề mặt
- mọi dòng thay đổi phải bám trực tiếp vào yêu cầu
- không mở rộng phạm vi nếu không cần thiết
- không refactor phần bên cạnh chỉ vì thấy chưa đẹp
- không đổi tên, đổi style, hoặc dọn dẹp ngoài phạm vi
- không thay đổi public contract nếu người dùng không yêu cầu
- không thêm dependency mới nếu chưa thật sự cần

## Quy tắc verify

- luôn verify trước khi nói là đã xong
- ưu tiên cách kiểm tra nhỏ và sát nhất:
- test lỗi cụ thể
- test file liên quan
- lint hoặc typecheck phần bị chạm tới
- build nếu thay đổi ảnh hưởng UI, asset, hoặc hành vi public
- nếu không thể verify thì phải nói rõ phần nào chưa verify

## Quy tắc đồng bộ runtime

- Nếu thay đổi chỉ có hiệu lực sau khi `khởi động lại dự án`, `restart dev server`, `chạy migration`, `xóa cache`, hoặc `reload bundle`, thì bắt buộc phải nhắc rõ người dùng thực hiện bước đó.
- Không được giả định người dùng đã tự restart hoặc môi trường đã tự đồng bộ.
- Khi nghi ngờ lỗi đến từ việc code và runtime chưa khớp nhau, phải ưu tiên kiểm tra khả năng này trước khi suy luận sâu hơn.
- Nếu vừa sửa code vừa biết chắc cần restart để nhìn thấy kết quả, phải nói rõ ngay trong phần kết luận để tránh đoán mù và hiểu nhầm.

Nếu verify thất bại, phải phân loại rõ:

- do thay đổi vừa tạo ra
- lỗi có sẵn
- lỗi không liên quan
- chưa đủ dữ liệu để kết luận

## Quy tắc an toàn

- không tự ý chạy hành động phá hủy dữ liệu
- không tự ý reset, revert, hoặc overwrite thay đổi của người dùng
- không commit secret, token, password, file môi trường, file runtime, backup, cache, hoặc log
- với tác vụ liên quan auth, upload, admin action, external integration, hoặc dữ liệu nhạy cảm: luôn kiểm tra phía server, validation, và rủi ro rò rỉ dữ liệu

## Quy tắc kiểm tra UI

- ưu tiên đọc source, CSS, cấu trúc component, và logic render trước khi mở browser automation
- không kiểm tra mù bằng browser khi source đã trả lời được câu hỏi
- chỉ dùng browser automation khi thật sự cần quan sát hành vi runtime mà terminal không xác minh được


## Quy tắc làm UI theo chuẩn production

Áp dụng bắt buộc cho mọi tác vụ liên quan giao diện, layout, CSS, component, responsive, dashboard, form, landing page, admin page, bảng dữ liệu, modal, toast, navigation, hoặc trải nghiệm người dùng.

### 1. Nguyên tắc trả lời khi làm UI

- Không giải thích dài dòng về lý thuyết thiết kế nếu người dùng không hỏi.
- Không viết nhiều phương án lan man. Chọn một phương án tốt nhất, phù hợp nhất với codebase hiện tại, rồi thực hiện.
- Không báo cáo bằng các bảng dài nếu không cần thiết.
- Không mô tả lại những việc hiển nhiên như “đã căn chỉnh giao diện cho đẹp hơn” nếu không nêu được thay đổi cụ thể.
- Khi kết thúc, chỉ báo cáo ngắn theo format `Changed / Verified / Notes`.
- Nếu cần nêu lý do thiết kế, viết tối đa 3 gạch đầu dòng, tập trung vào tác động thực tế: dễ nhìn hơn, ít rối hơn, responsive tốt hơn, đúng luồng người dùng hơn.

### 2. Chuẩn giao diện mong muốn

Giao diện phải đạt cảm giác sản phẩm thật, không phải bản demo, wireframe, prototype thô hoặc giao diện AI tạo nhanh.

Ưu tiên:

- sạch, gọn, hiện đại, có tính thương mại
- ít đường kẻ, ít viền, ít bảng không cần thiết
- dùng khoảng trắng, phân cấp chữ, màu nền nhẹ và grouping hợp lý thay vì lạm dụng border
- mỗi màn hình có một mục tiêu chính rõ ràng
- CTA chính nổi bật, CTA phụ nhẹ hơn
- font size, spacing, radius, shadow, màu sắc nhất quán với design system hiện có
- responsive tốt trên mobile, tablet và desktop
- trạng thái loading, empty, error, success phải gọn và tự nhiên
- nội dung hiển thị phải ngắn, rõ, đúng ngữ cảnh, ưu tiên tiếng Việt có dấu

Tránh:

- tạo quá nhiều card, bảng, badge, icon, tooltip, helper text, caption hoặc divider
- tạo bảng chỉ để trình bày nội dung không phải dữ liệu dạng bảng
- dùng border dày, đường kẻ ngang/dọc dày đặc, shadow nặng, gradient tùy tiện
- thêm chú thích giải thích giao diện ngay trên UI nếu người dùng không yêu cầu
- dùng text kiểu “Demo”, “Sample”, “Lorem ipsum”, “Coming soon” trừ khi người dùng yêu cầu
- tạo layout giống dashboard mẫu chung chung nhưng không phục vụ nghiệp vụ thật
- nhồi quá nhiều thông tin vào một màn hình
- dùng màu sắc, animation hoặc hiệu ứng gây rối

### 3. Quy tắc trước khi sửa UI

Trước khi sửa UI phải đọc tối thiểu:

- component hoặc page cần sửa
- component cha/con liên quan trực tiếp
- CSS/module/tailwind class/design token liên quan
- layout wrapper, theme provider hoặc design system nếu có

Sau khi đọc, phải xác định:

- màn hình này phục vụ mục tiêu gì
- phần nào đang rối, thừa, sai phân cấp hoặc không giống production
- có component/style sẵn nào nên tái sử dụng không
- thay đổi nhỏ nhất nào giúp giao diện tốt lên rõ rệt

Không được:

- viết lại toàn bộ UI khi chỉ cần chỉnh layout/style
- thêm thư viện UI mới nếu project đã có component/design system phù hợp
- tự đổi flow nghiệp vụ, API, schema, validation hoặc dữ liệu nếu người dùng chỉ yêu cầu sửa giao diện
- xóa logic hoặc trạng thái đang dùng chỉ vì muốn giao diện gọn hơn

### 4. Quy tắc tinh gọn UI

Khi người dùng yêu cầu “gọn hơn”, “production hơn”, “bớt rối”, “bớt đường kẻ”, “bớt chú thích”, mặc định phải rà soát và xử lý theo thứ tự:

1. bỏ các block mô tả, hướng dẫn, helper text không cần thiết
2. giảm số lượng card/table/divider/border dư thừa
3. gom nhóm thông tin theo nghiệp vụ thay vì tách quá nhiều khối nhỏ
4. thay bảng bằng list/card đơn giản nếu dữ liệu không cần so sánh theo cột
5. giảm icon/badge/trạng thái phụ nếu không giúp người dùng ra quyết định
6. chuẩn hóa spacing, font size, line-height, radius, shadow
7. làm rõ CTA chính và bỏ CTA phụ không cần thiết
8. kiểm tra lại mobile để tránh UI dài, nặng và khó thao tác

Không được xóa thông tin quan trọng. Nếu cần lược bớt nội dung, phải giữ lại nội dung phục vụ quyết định hoặc thao tác chính của người dùng.

### 5. Quy tắc cho bảng dữ liệu

Chỉ dùng bảng khi dữ liệu thật sự cần so sánh theo hàng/cột, cần sort/filter, hoặc là dữ liệu nghiệp vụ dạng danh sách.

Bảng production phải:

- có cột vừa đủ, không tạo cột mô tả dài không cần thiết
- căn chỉnh số, ngày, trạng thái và hành động nhất quán
- hạn chế border dọc; ưu tiên row spacing, nền nhẹ hoặc divider mảnh
- có trạng thái empty/loading/error ngắn gọn
- responsive hợp lý: ẩn cột phụ, chuyển sang card/list trên mobile nếu cần
- action chính rõ ràng, không dàn quá nhiều nút trên một dòng

Không dùng bảng cho:

- mô tả tính năng marketing
- checklist ngắn
- nội dung giải thích quy trình
- so sánh đơn giản có thể trình bày bằng card/list gọn hơn

### 6. Quy tắc nội dung chữ trên UI

- Viết ngắn, tự nhiên, đúng nghiệp vụ.
- Label, placeholder, button, error message phải rõ hành động.
- Không dùng câu văn AI dài dòng trong giao diện.
- Không giải thích luật/logic kỹ thuật dài ngay trong UI; nếu cần thì đưa vào tooltip hoặc tài liệu riêng, nhưng chỉ khi thật sự cần.
- Mặc định không thêm helper text, caption, mô tả, subtitle hoặc câu hướng dẫn dưới tiêu đề nếu người dùng không yêu cầu rõ.
- Nếu bỏ phần mô tả mà màn hình vẫn dùng được thì phải bỏ.
- Không viết các câu kiểu “Nhập xong thì…”, “Chọn một… để…”, “Đây là cụm…” trong UI.
- Ưu tiên microcopy giúp người dùng thao tác, không phải microcopy để trang trí.

### 7. Quy tắc kiểm tra UI sau khi sửa

Sau khi sửa UI, phải verify sát nhất có thể:

- lint/typecheck nếu có
- build nếu thay đổi ảnh hưởng page/component public
- kiểm tra responsive ở mobile và desktop nếu có thể
- kiểm tra trạng thái loading/empty/error nếu có thay đổi liên quan
- kiểm tra console/runtime nếu có thể chạy app
- nếu không thể kiểm tra bằng browser, phải nêu rõ chưa verify runtime

Không được kết luận “giao diện đã chuẩn production” nếu chưa chạy được kiểm tra tối thiểu. Chỉ được nói “đã tinh gọn theo hướng production” và ghi rõ phần đã verify.

### 8. Format báo cáo riêng cho tác vụ UI

Khi kết thúc tác vụ UI, dùng format ngắn sau:

Changed:

- nêu 2-5 thay đổi cụ thể, không viết lan man

Verified:

- nêu lệnh đã chạy hoặc cách đã kiểm tra

Notes:

- chỉ ghi rủi ro, phần chưa verify, hoặc giả định quan trọng

Không thêm bảng tổng hợp, không thêm checklist dài, không thêm phần giải thích thiết kế nếu người dùng không yêu cầu.

## Quy tắc báo cáo kết quả

Khi kết thúc một tác vụ code, dùng format sau:

Changed:

- ...

Verified:

- ...

Notes:

- giả định, rủi ro, blocker, hoặc phần chưa verify

---
> Source: [thuanphanmem/khoa-ung-dung-android-tu-xa](https://github.com/thuanphanmem/khoa-ung-dung-android-tu-xa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
