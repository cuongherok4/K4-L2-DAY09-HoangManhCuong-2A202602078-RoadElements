# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Viết lại đầy đủ 10 mục; chốt một housing/một instance, visible-only geometry, ngưỡng 8 px, `off` so với `unknown`, bằng chứng gán `relevance`, `pictogram` cho direction, LABEL / IGNORE / UNKNOWN / ESCALATE và temporal rule; bổ sung sample pack cùng edge/gold cases LISA + BDD100K | Chuyển downstream contract và các edge case thành hướng dẫn tự đủ để annotator dùng độc lập trong calibration | `01_problem_statement.md`; `03_cvat_labels.json`; `sample_pack.csv`; `04_edge_cases/edge_case_cards.md`; các sample example/calibration LISA/BDD100K |
