# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** team01
- **Nhóm peer test bài của mình:** team09 (cặp team01 ↔ team09)
- **Nhóm mình test bài của:** team09
- **Problem family:** Traffic light — state + ego relevance
- **Nguồn ảnh:** `bdd100k` (example/calibration/blind), `lisa` (example/calibration)

| Thành viên | GitHub / email | Vai trò chính | File phụ trách |
|---|---|---|---|
| Hoàng Mạnh Cường (lead) | cuongherok4 | Lead + spec owner, review PR | `00_team.md`, `01_problem_statement.md`, `03_cvat_labels.json`, `02_guideline.md`, `08_revision_log.md` |
| (chưa có tên) | davidho181004@gmail.com | CVAT owner | `03_ontology_and_cvat_setup.md`, `sample_pack.csv`, `09_cvat_export_or_task_reference.txt` |
| Hoàng Văn Long | hvlongg@gmail.com | Edge case owner | `04_edge_cases/edge_case_cards.md` |
| Thái Đức Cường | elr.iahtc25@gmail.com | Gold owner (người duy nhất chạy `make freeze`) | `04_edge_cases/gold_decisions.csv` |
| (chưa có tên) | namtrinhtrung731@gmail.com | QA + calibration + blind handoff owner | `05_qa_plan.md`, `06_calibration_report.csv`, `06_calibration_measure.csv`, `07_blind_handoff/` |

## Phân công việc

| Người | Việc chính trong buổi | Trạng thái |
|---|---|---|
| Hoàng Mạnh Cường (lead) | Problem statement + downstream contract; CVAT labels JSON; guideline v1 (10 mục bắt buộc) → v2 sau calibration → v3 sau blind test; ghi mỗi version vào `08_revision_log.md`; review và merge PR sau khi duyệt | `01` xong, `03_cvat_labels.json` xong, `02` chưa bắt đầu |
| davidho181004 | Bảng ontology khớp `03_cvat_labels.json` (tên label, attribute, default); chia ảnh vào `sample_pack.csv` (example / calibration / blind); `make pack`; tạo task CVAT, dán labels + guideline; setup test; điền `09` | Chưa bắt đầu |
| Hoàng Văn Long | ≥ 8 edge case card, có case critical + case escalation; mỗi card có expected decision + rationale | Chưa bắt đầu |
| Thái Đức Cường | `gold_decisions.csv` cho từng ảnh blind (≥ 10 decision, ≥ 2 critical, ≥ 1 geometry); soát lại ảnh blind ở kích thước gốc; **người duy nhất chạy `make freeze`** | Chưa bắt đầu |
| namtrinhtrung731 | `make calib` + `06_calibration_report.csv` (≥ 3 bất đồng); `05_qa_plan.md`; `make handoff` gửi team09, nhận bài team09; clarification log; `make score` + `make gts`; `peer_feedback.md` | Chưa bắt đầu |

Calibration (phút 120–140): cả 5 người label độc lập trên CVAT của mình rồi export vào `06_calibration_exports/`.
Mỗi file một người sửa chính; mọi thay đổi đi qua nhánh riêng + PR cho lead duyệt (xem `GIT_WORKFLOW.md`).
