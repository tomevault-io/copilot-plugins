## lappy-medi

> MediExcel Lap- Tài Liệu Kỹ Thuật & Hướng Dẫn Vận Hành


MediExcel Lap- Tài Liệu Kỹ Thuật & Hướng Dẫn Vận Hành
App viết bằng Nextjs với giao diện đơn giản thân thiện dễ thao tác cho phép người dùng nhập liệu bệnh nhân một cách nhanh chóng và chính xác (Giống excel nhưng cách trường dữ liệu nội khoa có data mẫu để lựa chọn nhanh)
1. Cơ Chế Xử Lý Excel (utils/excelUtils.ts)
Quy trình Nhập (Import)
Định dạng hỗ trợ: .xlsx, .xls.
Chọn Sheet: Hệ thống ưu tiên tìm Sheet có tên là "DS". Nếu không tìm thấy, sẽ tự động lấy Sheet đầu tiên (index 0).
Xác định dòng tiêu đề (Header):
Quét 20 dòng đầu tiên của file.
Dòng nào chứa chữ "CODE" hoặc "HỌ VÀ TÊN" (không phân biệt hoa thường) sẽ được chọn làm dòng tiêu đề.
Dữ liệu bệnh nhân được lấy từ dòng ngay sau dòng tiêu đề.
Cột tiêu chuẩn (Standard Mapping):
Hệ thống tự động map hoặc tạo mới các cột sau nếu thiếu:
CODE, HỌ VÀ TÊN, NS (Năm sinh), GT (Giới tính).
Cân nặng, Chiều cao, BMI, THỂ TRẠNG.
KHÁM TỔNG QUÁT, PHÂN LOẠI SỨC KHỎE.
Xquang, Siêu âm, Điện tim.
Quy trình Xuất (Export)
Chỉ xuất các cột đang ở trạng thái Visible (được tick chọn trên giao diện).
Thứ tự cột trong file xuất tuân theo thứ tự hiển thị trên màn hình.
File xuất ra có tên mặc định: Ket_Qua_Kham.xlsx.
Dữ liệu được ghi vào Sheet tên "DS".
2. Chi Tiết Logic Nhập Liệu (components/PatientEditor.tsx)
Giao diện chỉnh sửa (Edit Modal) chia làm 3 Tab chính. Dưới đây là các logic và option được code cứng (hard-coded) trong ứng dụng:
Tab 1: Thể Lực & Phân Loại (Vital & Class)
1. Tính toán BMI & Thể Trạng:
Input: Chiều cao (mét), Cân nặng (kg).
Công thức: BMI = Cân nặng / (Chiều cao * Chiều cao). Làm tròn 2 chữ số thập phân.
Logic tự động điền Thể trạng:
BMI < 18: "Thiếu cân"
18 <= BMI <= 25: "Bình thường" (Màu xanh)
BMI > 25: "Thừa cân" (Màu cam)
2. Phân loại sức khỏe:
Các nút chọn nhanh: Loại I, Loại II, Loại III, Loại IV, Loại V.
Dữ liệu lưu vào cột: PHÂN LOẠI SỨC KHỎE.
Tab 2: Khám Tổng Quát (General Exam)
Hệ thống sẽ ghép dữ liệu từ các trường con thành một đoạn văn bản duy nhất, phân cách bằng xuống dòng (\n), lưu vào cột KHÁM TỔNG QUÁT.
1. Nội khoa:
Huyết áp: Nhập Tâm thu / Tâm trương (VD: 120/80 mmHg).
Option bệnh lý có sẵn:
"Tăng HA có điều trị"
"Tăng HA không điều trị"
"TD tăng HA"
Kết quả: Ghép chuỗi HA {sys}/{dia} mmHg, {bệnh lý}, {ghi chú}. Nếu rỗng -> "Bình thường".
2. Mắt:
Thị lực: Dropdown từ 1/10 đến 10/10.
Checkbox: "Có kính (CK)". Nếu tick -> Thêm tiền tố "CK " vào trước kết quả.
Option bệnh lý:
"Đục thủy tinh thể"
"2 mắt lão thị"
3. Tai Mũi Họng (TMH):
Option bệnh lý:
"Amidan quá phát"
"Viêm họng hạt"
"Viêm mũi dị ứng"
Mặc định: Nếu không chọn gì -> "Bình thường".
4. Răng Hàm Mặt (RHM):
Sức nhai: Thanh trượt (Slider) từ 0% - 100% (bước nhảy 5%).
Option bệnh lý:
"Sâu răng"
"Mất răng"
"Vôi răng"
5. Ngoại khoa & Da liễu:
Nhập văn bản tự do. Mặc định khởi tạo là "Bình thường".
Tab 3: Cận Lâm Sàng (Imaging)
Dữ liệu được lưu vào các cột riêng biệt tương ứng trong Excel.
1. X-Quang (Cột Xquang):
Nút "Đặt mặc định": Tự động điền câu: "Hình ảnh tim, phổi chưa ghi nhận bất thường trên phim xquang".
2. Siêu âm (Cột Siêu âm):
Hệ thống ghép các trường con: Bụng, Tuyến giáp, Vú, Phụ khoa.
Siêu âm bụng (Option nhanh):
Nút "Bình thường" -> Ghi "Chưa ghi nhận bất thường".
Gan: "Gan nhiễm mỡ độ 1", "Gan nhiễm mỡ độ 2", "Gan nhiễm mỡ độ 3".
Thận: "Sỏi thận Trái", "Sỏi thận Phải", "Sỏi thận 2 bên".
Logic ghép: Chỉ dòng nào có dữ liệu khác "Chưa ghi nhận bất thường" (trừ bụng) mới được ghép vào chuỗi kết quả.
Fallback: Nếu tất cả trống -> "Siêu âm bụng: Chưa ghi nhận bất thường".
3. Điện Tâm Đồ (Cột Điện tim):
Input: Nhịp tim (số).
Logic:
Nếu có nhập số -> "Nhịp xoang: {số} l/p".
Nếu không nhập -> "Nhịp xoang đều" (hoặc giữ nguyên văn bản cũ).
3. Cấu Trúc Dữ Liệu (Data Structure)
Ứng dụng sử dụng một mảng đối tượng linh hoạt (PatientData[]), nhưng phụ thuộc vào các key tiêu chuẩn sau để vận hành logic:
Key Code (App)	Header Excel (Hiển thị)	Mô tả
code	CODE	Mã nhân viên/Bệnh nhân
name	HỌ VÀ TÊN	Tên đầy đủ
dob	NS	Ngày/Năm sinh
gender	GT	Giới tính
weight	Cân nặng	Đơn vị kg
height	Chiều cao	Đơn vị mét (m)
bmi	BMI	Tự động tính
physique	THỂ TRẠNG	Tự động phân loại
generalExam	KHÁM TỔNG QUÁT	Tổng hợp từ Tab 2
classification	PHÂN LOẠI SỨC KHỎE	I, II, III...
xray	Xquang	Kết quả X-Quang
ultrasound	Siêu âm	Kết quả Siêu âm
ecg	Điện tim	Kết quả ECG

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zliveze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
