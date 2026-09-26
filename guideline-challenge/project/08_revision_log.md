# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu: đủ 10 mục; rule `relevance` (giao lộ gần nhất, quay về camera, hướng đi của ego xác định từ bằng chứng làn), xác định `state` theo vị trí bóng, bảng LABEL / IGNORE / UNKNOWN / ESCALATE, ví dụ trên LISA01 / LISA30 | Chuẩn bị task calibration (gate G2) | `01_problem_statement.md`, `03_cvat_labels.json` |
