# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** team01
- **Nhóm peer test bài của mình:** team09 (cặp team01 ↔ team09)
- **Nhóm mình test bài của:** team09
- **Problem family:** Traffic light — state + ego relevance
- **Nguồn ảnh:** `bdd100k` (example/calibration/blind), `lisa` (example/calibration)

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Hoàng Mạnh Cường (lead) | cuongherok4 | Lead: problem + ontology/CVAT schema, blind handoff với team09, review PR | `00_team.md`, `01_problem_statement.md`, `03_cvat_labels.json`, `03_ontology_and_cvat_setup.md`, `07_blind_handoff/` |
| TODO | TODO | Spec owner | `02_guideline.md`, `08_revision_log.md` |
| TODO | TODO | Data + CVAT task owner | `sample_pack.csv`, `09_cvat_export_or_task_reference.txt` |
| TODO | TODO | Gold owner (người duy nhất chạy `make freeze`) | `04_edge_cases/edge_case_cards.md`, `04_edge_cases/gold_decisions.csv` |
| TODO | TODO | QA + calibration owner (đo đồng thuận, chấm điểm) | `05_qa_plan.md`, `06_calibration_report.csv`, `06_calibration_measure.csv` |

## Phân công việc

| # | Vai | Việc chính trong buổi | Trạng thái |
|---|---|---|---|
| 1 | Lead (Hoàng Mạnh Cường) | `00` team; `01` problem statement + downstream contract; `03` CVAT labels JSON + bảng ontology khớp JSON; review và merge PR sau khi duyệt | **Xong** (`00`, `01`, `03`) |
| 1b | Lead (việc thêm) | Blind handoff: `make handoff` gửi gói cho team09, nhận bài team09 về `07_blind_handoff/peer_output/`; trả lời câu hỏi và ghi `clarification_log.csv`; viết `peer_feedback.md`; điều phối timeline + gate `make status` | Chưa bắt đầu |
| 2 | Spec owner | Guideline v1 (10 mục bắt buộc) → v2 sau calibration → v3 sau blind test; ghi mỗi version vào `08_revision_log.md` | Chưa bắt đầu |
| 3 | Data + CVAT task owner | Chia ảnh vào `sample_pack.csv` (example / calibration / blind); `make pack`; tạo task CVAT, dán labels + guideline; nhờ 1 người chưa setup test task; điền `09` | Chưa bắt đầu |
| 4 | Gold owner | ≥ 8 edge case card (có critical + escalation); `gold_decisions.csv` (≥ 10 decision, ≥ 2 critical, ≥ 1 geometry); **người duy nhất chạy `make freeze`** | Chưa bắt đầu |
| 5 | QA + calibration owner | `05_qa_plan.md`; gom export vào `06_calibration_exports/`, `make calib` + `06_calibration_report.csv` (≥ 3 bất đồng) đưa lại cho Spec owner; `make score` + `make gts` trên bài blind | Chưa bắt đầu |

Calibration (phút 120–140): cả 5 người label độc lập trên CVAT của mình rồi export vào `06_calibration_exports/`.
Mỗi file một người sửa chính; mọi thay đổi đi qua nhánh riêng + PR cho lead duyệt (xem `GIT_WORKFLOW.md`).
