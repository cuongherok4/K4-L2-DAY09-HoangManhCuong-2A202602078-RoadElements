# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.


- **Team:** team01
- **Nhóm peer test bài của mình:** team09 (cặp team01 ↔ team09)
- **Nhóm mình test bài của:** team09
- **Problem family:** Traffic light — state + ego relevance
- **Nguồn ảnh:** `bdd100k`, `lisa`

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Hoàng Mạnh Cường | [cuongherok4](https://github.com/cuongherok4) | **Lead** + spec owner; review và merge PR | `01_problem_statement.md`, `02_guideline.md` (+ `02_guideline_figures/`), `03_cvat_labels.json`, `08_revision_log.md` |
| Hồ Minh Trí | [DavidHo1801004](https://github.com/DavidHo1801004) | **Thuyết trình** (debrief owner) + CVAT owner | `03_ontology_and_cvat_setup.md`, `09_cvat_export_or_task_reference.txt` |
| Hoàng Văn Long | [HvLonggg](https://github.com/HvLonggg) | Edge case owner + setup tester | `04_edge_cases/edge_case_cards.md`, `sample_pack.csv` |
| Thái Đức Cường | [maituanv2510](https://github.com/maituanv2510) | Gold owner — **người duy nhất chạy `make freeze`** | `04_edge_cases/gold_decisions.csv` |
| Trịnh Nam Trung | [trinhnamtrung](https://github.com/trinhnamtrung) | QA + calibration + blind handoff owner | `05_qa_plan.md`, `06_calibration_report.csv`, `06_calibration_measure.csv`, `07_blind_handoff/` |

## Phân công theo mốc thời gian

| Phút | Hoàng Mạnh Cường (lead) | Hồ Minh Trí (thuyết trình) | Hoàng Văn Long | Thái Đức Cường | Trịnh Nam Trung |
|---|---|---|---|---|---|
| 0–80 (G1) | chốt topic, `01`, guideline v1, labels JSON — **xong** | đọc hết `01`, `02`, `03` để nắm toàn bộ thiết kế | rà `edge_case_cards.md` (12 card) với ảnh thật | rà `gold_decisions.csv` với 5 ảnh blind ở kích thước gốc | rà `05_qa_plan.md` |
| 80–110 (G2) | duyệt PR, trả lời câu hỏi rule | tạo task calibration mẫu, dán labels + Guide; ghi phần CVAT trong `03_ontology…` | **setup test**: mở task của Trí khi chưa đọc hướng dẫn setup, ghi chỗ vấp | — | — |
| 120–140 | label calibration độc lập | label calibration độc lập | label calibration độc lập | label calibration độc lập | label calibration độc lập; gom 5 file export vào `06_calibration_exports/` |
| 140–160 (G3, G4) | sửa guideline **v2** từ calibration report, dòng v2 trong `08` | ghi lại "guideline vỡ ở đâu" để chuẩn bị debrief | cập nhật edge card từ bất đồng calibration | chốt gold, **chạy `make freeze`**, `git push --follow-tags` | `make calib`, viết `06_calibration_report.csv` (≥ 3 bất đồng) |
| 160–185 (G5) | label bài blind của team09 | label bài blind của team09 | label bài blind của team09 | label bài blind của team09 | `make handoff` gửi team09; ghi `clarification_log.csv`; nhận export + feedback |
| 185–205 | chẩn đoán decision sai | tổng hợp số liệu GTS cho debrief | chẩn đoán decision sai | điền `correct`/`note` trong `transfer_score.csv` | `make score`, `make gts`, `peer_feedback.md` |
| 205–225 (G6) | guideline **v3**, dòng v3 trong `08` | điền `09`; làm 1 trang debrief | edge card ≥ 8, thêm case từ blind test | — | `make check` |
| 225–240 | nộp: `git push --follow-tags` | **thuyết trình debrief** (2 phút owner): guideline vỡ ở đâu, đã sửa gì | hỗ trợ trả lời | hỗ trợ trả lời | hỗ trợ trả lời |

Luật làm chung:
- Mỗi file chỉ một người sửa chính (cột "File phụ trách"). Muốn sửa file của người khác thì nhắn người đó.
- Mọi thay đổi đi qua nhánh riêng và PR để lead duyệt. Luôn `git pull` trước khi sửa.
- **Calibration độc lập:** không xem màn hình, không xem file export của nhau trước khi `make calib`.
- **Chỉ Thái Đức Cường chạy `make freeze`.** Người khác đợi push rồi chạy `git pull && git fetch --tags --force`.
- Người label bài blind của team09 **không xem** trước guideline của team09. Trong blind window, không ai giải thích rule cho team09 bằng miệng.

Gợi ý chia vai (nhóm 2–3 người thì gộp): **spec owner** (`01`, `02`), **CVAT owner** (`03_*`, `sample_pack.csv`,
`09`), **gold owner** (`04_edge_cases/`), **QA owner** (`05`, `06`, `07_blind_handoff/`). Mỗi file một người sửa
chính để tránh xung đột git. Calibration thì mọi người cùng label.
