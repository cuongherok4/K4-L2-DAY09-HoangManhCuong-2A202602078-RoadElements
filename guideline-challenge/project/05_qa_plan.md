# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Self-QC (annotator, trước khi nộp batch):** bật **Attribute annotation**, duyệt từng object: không còn
  `__undefined__`; mọi `unknown` đều có `needs_review`; mọi đèn `arrow_left/arrow_right` + `relevant` đã tự kiểm lại.
- **Ai review, review bao nhiêu:** QA owner review; annotator khác làm reviewer chéo. Mỗi batch: **100%** ảnh có tag
  `low_visibility`/`conflict`/`critical` trong sample pack hoặc có ít nhất một `needs_review`; **20%** ngẫu nhiên phần
  còn lại (tối thiểu 5 ảnh). Annotator mới: 100% ở batch đầu tiên.
- **Chọn sample theo rule nào:** theo rủi ro trước (ảnh đêm/mưa/chạng vạng, ảnh ≥ 4 đầu đèn, ảnh có mũi tên), rồi
  random có seed ghi lại trong issue để tái lập được.
- **Issue được ghi ở đâu, đóng thế nào:** issue CVAT trên đúng object (chuột phải → **Open an issue**), mở đầu bằng
  severity (`[CRITICAL]`, `[MAJOR]`, `[MINOR]`, `[QUESTION]`). Annotator sửa rồi reply; **reviewer** (không phải
  annotator) bấm Resolve sau khi kiểm lại. Batch chỉ qua gate khi không còn issue critical/major mở.
- **Khi phát hiện guideline gap thì update và version ra sao:** issue `[QUESTION]` lặp ≥ 2 lần cùng loại, hoặc bất
  đồng calibration được chẩn đoán `guideline_gap` → spec owner sửa `02_guideline.md`, tăng version (v1 → v2 …), ghi
  `08_revision_log.md` kèm bằng chứng, dán lại Guide trên CVAT. Ảnh đã label theo version cũ bị ảnh hưởng → re-check.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | lỗi làm planning ra STOP/GO sai ở giao lộ gần nhất | bỏ sót đèn relevant đỏ/vàng; đèn relevant đỏ gán `green`; mũi tên rẽ/đèn giao lộ kế gán `relevant`; vẽ đèn đi bộ hoặc phản chiếu | sửa ngay, re-check **toàn bộ** ảnh của annotator đó trong batch |
| Major | sai nhãn không đổi STOP/GO ngay nhưng làm bẩn dữ liệu train | bỏ sót đèn `not_relevant`; sai `pictogram`; gộp 2 vỏ thành 1 box; `__undefined__` còn trong export; `unknown` không có `needs_review` | sửa ảnh đó, đếm vào gate batch |
| Minor | sai geometry vượt tolerance nhưng đúng object và attribute | box gồm cần treo; box đêm gồm quầng; lệch > 2 px ở đèn nhỏ | sửa khi rework, không chặn batch nếu dưới ngưỡng |
| Question | annotator/reviewer không chắc guideline muốn gì | đèn cho đường gom song song có relevant không? | đưa spec owner quyết trong ≤ 1 ngày; lặp lại → sửa guideline |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Critical defect rate | số lỗi critical / số ảnh được review | chỉ số an toàn — mỗi lỗi là một STOP/GO sai |
| Relevant-light recall | đèn relevant annotator vẽ đúng / đèn relevant reviewer xác nhận | bỏ sót đèn relevant là failure (a) |
| Attribute accuracy | object đúng cả `state` + `relevance` + `pictogram` / object được review | ba attribute là đầu ra downstream dùng trực tiếp |
| Geometry pass rate | box trong tolerance / box được review | detector cần box nhất quán, nhất là đèn < 20 px |
| Escalation rate | ảnh có `image_escalate` hoặc ≥ 1 `needs_review` / tổng ảnh | quá cao = guideline chưa đủ; quá thấp trên ảnh đêm/mưa = annotator đang đoán |
| Undefined rate | object còn `__undefined__` / tổng object | lỗi quy trình, phải bằng 0 |

Metric high-risk tách riêng: **critical defect escape rate** = lỗi critical tìm thấy ở vòng audit sau khi batch đã
PASS / số ảnh audit. Audit 10% ảnh của batch đã PASS; escape rate > 0 thì hạ ngưỡng sampling (tăng review lên 40%)
cho batch tiếp theo.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  critical defect rate = 0 trên mẫu review
  AND relevant-light recall >= 98%
  AND attribute accuracy >= 95%
  AND geometry pass rate >= 90%
  AND undefined rate = 0
REWORK if: có 1–2 lỗi critical trong mẫu, hoặc attribute accuracy 85–95%, hoặc geometry pass rate 75–90%
  → annotator sửa, reviewer review lại 100% batch đó
REJECT / ESCALATE if: >= 3 lỗi critical, hoặc attribute accuracy < 85%, hoặc escalation rate > 30% trên ảnh ban ngày
  → trả batch, họp calibration lại; escalation rate cao thì spec owner xem lại guideline trước khi label tiếp
```

Trade-off: critical = 0 là cứng vì một lỗi STOP/GO có thể gây tai nạn, và chi phí review 100% ảnh rủi ro nhỏ (ảnh
đêm/mưa/nhiều đèn chiếm ít). Attribute 95% và geometry 90% nới hơn vì lỗi major/minor chỉ làm giảm chất lượng train,
có thể bù bằng số lượng dữ liệu; đòi 99% sẽ nhân đôi chi phí review mà downstream gần như không hưởng lợi. Geometry
nới nhất vì đèn 6–12 px thì lệch 1–2 px đã là 10–30% kích thước — ép chặt hơn chỉ đo nhiễu tay người vẽ.
