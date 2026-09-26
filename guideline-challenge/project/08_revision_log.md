# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: 1 class `traffic_light` + `state`/`relevance`/`pictogram`/`needs_review`, tag `image_escalate`; rule điểm dừng có đèn gần nhất + ego đi thẳng; ngưỡng 6 px; geometry ban đêm ôm lõi sáng; quy trình từng bước trên CVAT; rule tổng quát cho tình huống ngoài bộ ảnh (đầu 5 ô, LED nhấp nháy, đọc màu theo vị trí ô, ramp meter, đèn buýt/tàu/làn, đèn chắn tàu); thuật ngữ, cài đặt CVAT lần đầu, làm mẫu LISA03, FAQ; 10 hình minh hoạ trong `02_guideline_figures/` + sơ đồ chữ dự phòng | Khởi tạo từ downstream contract (`01_problem_statement.md`); guideline phải dùng được cho ảnh chưa thấy, không chỉ ảnh trong `data/` | Khảo sát 26 ảnh BDD + frame LISA; `pictogram` thêm sau khi so với schema tham khảo, bỏ `pedestrian`/`bicycle`/`red_yellow`/`occluded` (lý do ở `03_ontology_and_cvat_setup.md`) |
| v2 | (1) Vị trí ô đang sáng quyết định `state` (ô trên = red kể cả trông cam/vàng); `unknown` chỉ khi không xác định được ô sáng. (2) Xe mình luôn đi thẳng, cấm tự giả định rẽ. (3) Lõi sáng loá nhưng tròn đều = `circle`. (4) Không ước lượng vỏ khi vỏ tối lẫn nền → ôm lõi sáng. (5) Bước quét ảnh: sát hai mép + dải chân trời. Đưa (1)(2) lên Tóm tắt 30 giây; thêm ví dụ LISA09, FAQ, common mistakes | Calibration cuong vs long: đồng thuận count 57.1%, attribute 20%; long đọc toàn bộ đèn đỏ LISA09/LISA16 thành yellow; hai người lệch relevance mũi tên trái, pictogram/relevance BDD25, geometry đầu vỏ tối | `06_calibration_report.csv` dòng LISA09 state, LISA16 state, LISA09 relevance, BDD25, LISA16 geometry; `06_calibration_measure.csv` |
