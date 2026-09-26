# QA plan + quality gates

Không được viết "reviewer kiểm tra lại". Phải có sampling, metric, threshold và action khi fail. Thay mọi placeholder
mới là xong (gate G6).

> Bản nháp v1 (QA owner, trước calibration). Ngưỡng ở mục Quality gate là đề xuất ban đầu, sẽ chỉnh theo mức bất đồng
> đo được ở `06_calibration_measure.csv`. Đơn vị kiểm: **một đầu đèn** (box `traffic_light`) và **một ảnh** (tag
> `image_escalate`). Một batch = một job CVAT.

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate. Ghi cụ thể cho project của nhóm:

- **Ai review, review bao nhiêu:** review chéo — reviewer không bao giờ là người đã label ảnh đó. Tầng 1: một
  annotator khác review theo tỉ lệ bên dưới. Tầng 2: QA owner re-check **100%** mọi đèn mà tầng 1 đánh dấu lỗi
  critical. Annotator mới (chưa qua calibration) bị review 100% ở 2 batch đầu.
- **Chọn sample theo rule nào** (random, theo tag rủi ro, theo annotator mới…): chọn theo quy tắc cố định để ai chạy
  cũng ra cùng một tập:
  1. **100%** ảnh tầm nhìn kém theo metadata của ảnh: `timeofday` = `night` / `dawn/dusk` hoặc `weather` = `rainy` /
     `snowy` / `foggy` (đèn nhòe, lóa, chỉ thấy đốm sáng); và ảnh mà annotator đã gán ít nhất một đèn `occluded` = true
     hoặc cao dưới 20 px (bị che, nhỏ/xa).
  2. **100%** ảnh có ít nhất một đèn `relevance=relevant` với `state` = `red` hoặc `yellow` (vùng lỗi critical).
  3. **100%** đèn có `needs_review` = true và ảnh có tag `image_escalate`.
  4. Phần còn lại: **20%** ngẫu nhiên mỗi batch, tối thiểu 5 ảnh.
- **Issue được ghi ở đâu, đóng thế nào:** reviewer mở Issue ngay trên object trong chế độ Review của CVAT, tiêu đề bắt
  đầu bằng mức severity (`[CRITICAL]`, `[MAJOR]`, `[MINOR]`, `[QUESTION]`). Issue đóng khi annotator đã sửa **và**
  reviewer kiểm lại rồi bấm Resolve; issue `[CRITICAL]` chỉ QA owner được đóng. Hàng đợi `needs_review` /
  `image_escalate` do QA owner cùng spec owner xử lý trong cùng batch: mỗi mục kết thúc bằng một quyết định (giữ
  `unknown`, sửa nhãn, hoặc thêm rule) — không trả lời bằng miệng.
- **Khi phát hiện guideline gap thì update và version ra sao:** issue nào được chẩn đoán `guideline_gap` chuyển cho
  spec owner. Spec owner sửa `02_guideline.md`, tăng `Version` (v1 → v2 sau calibration, v2 → v3 sau blind test), ghi
  một dòng vào `08_revision_log.md` kèm bằng chứng (sample_id, dòng calibration report hoặc câu hỏi trong clarification
  log), dán lại vào Guide của task CVAT và báo cả nhóm. Các ảnh đã label theo rule cũ bị lọc theo attribute liên quan
  để review lại — không label lại toàn bộ.

## Defect severity

Nhóm được đổi mapping nếu downstream contract khác, nhưng phải giải thích và chốt trước khi QA.

Nguyên tắc xếp mức: nhìn **hậu quả với planner**, không nhìn độ khó của lỗi. Khớp câu 3 của `01_problem_statement.md`
và cột `severity` trong `gold_decisions.csv`.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Lỗi khiến xe **đi khi đáng lẽ phải dừng** | Đèn đỏ/vàng điều khiển làn ego bị gán `state=green`, hoặc `relevance=not_relevant`, hoặc không được vẽ; đèn xanh của làn/hướng khác bị gán `relevant` trong khi đèn của ego đang đỏ *(dòng này chờ lead chốt R2)* | Chặn cả batch; sửa ngay; QA owner re-check 100% batch cho cùng loại lỗi; ghi nguyên nhân (guideline gap hay execution error) |
| Major | Sai làm model học lệch hoặc khiến xe **dừng khi được đi**, nhưng không làm xe vượt đèn đỏ | Bỏ sót đèn `not_relevant` cao ≥ 8 px; sai `state` của đèn `not_relevant`; sai `pictogram` (mũi tên ghi thành `circle`); vẽ tín hiệu đi bộ hoặc vệt phản chiếu thành `traffic_light` (phanh ma) | Rework trong batch; reviewer kiểm lại đúng các object đó |
| Minor | Sai không đổi quyết định dừng/đi | Box lệch quá dung sai ở `01_problem_statement.md` nhưng vẫn ôm đèn; box ôm cả cột/cần treo; quên tick `occluded` | Sửa khi rework; không chặn batch nếu tỉ lệ dưới ngưỡng |
| Question | Không phải lỗi — guideline chưa có rule cho tình huống này | Đèn bị che không rõ thuộc làn nào; ego đang vắt giữa hai làn; không đọc được mũi tên | Annotator tick `needs_review` / tag `image_escalate`; spec owner trả lời bằng rule hoặc ví dụ mới trong guideline |

## Metrics

Chuẩn so sánh: **bản đã review** (reviewer sửa xong trên các ảnh được lấy mẫu) — mỗi metric tính trên tập ảnh được
review của batch. Hai box "khớp" khi IoU ≥ 0.5.

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Box recall | Số đèn (cao ≥ 8 px) trong bản đã review có box khớp của annotator ÷ tổng số đèn trong bản đã review | Bỏ sót đèn của ego là critical; đèn nhỏ/xa dễ bị bỏ sót nhất |
| Box precision | Số box khớp một đèn xe thật ÷ tổng số box annotator vẽ | Box vẽ vào tín hiệu đi bộ hoặc vệt phản chiếu gây phanh ma |
| State accuracy | Số đèn khớp có `state` đúng ÷ số đèn khớp; báo **tách riêng** ngày và đêm/chạng vạng | State là sự thật vật lý nhưng ban đêm chỉ thấy đốm sáng — số chung che mất slice khó |
| Relevance accuracy | Số đèn khớp có `relevance` đúng ÷ số đèn khớp; báo tách riêng ảnh có ≥ 3 đầu đèn | Relevance là suy luận ngữ cảnh — phần khó nhất và quyết định dừng/đi |
| Pictogram accuracy (đèn mũi tên) | Số đèn mũi tên có `pictogram` đúng ÷ số đèn mũi tên | Đèn tròn chiếm đa số nên tính chung sẽ thổi phồng độ chính xác |
| Tỉ lệ attribute chưa gán | Số đèn còn `__undefined__` ở bất kỳ attribute nào ÷ tổng số đèn | Schema cố ý dùng `__undefined__` làm default; còn sót nghĩa là annotator quên |
| Tỉ lệ escalate | (số đèn `needs_review` + số ảnh `image_escalate`) ÷ số đèn | Quá cao → guideline thiếu rule; bằng 0 ở ảnh đêm → có thể annotator đang đoán |
| Geometry compliance | Số box khớp nằm trong dung sai ÷ số box khớp | Bám rule "ôm vỏ đèn nhìn thấy" thay vì cảm giác |

Metric high-risk tách riêng (ví dụ critical defect escape rate): **Critical escape rate** = số lỗi critical còn lại
sau review (tìm thấy ở gate cuối hoặc trong blind test) ÷ số đèn `relevant` có `state` đỏ/vàng. Mục tiêu **0**. Báo
riêng, không gộp vào accuracy chung, vì một lỗi critical đã đủ để xe vượt đèn đỏ.

## Quality gate

Threshold là đề xuất của nhóm, không phải chuẩn ngành. Giải thích trade-off cost/risk.

```text
PASS if:
  critical escape = 0
  AND tỉ lệ attribute chưa gán = 0
  AND box recall ≥ 95% AND box precision ≥ 95%
  AND state accuracy ≥ 95% (cả slice ngày và slice đêm/chạng vạng)
  AND relevance accuracy ≥ 90%
  AND geometry compliance ≥ 90%
  AND mọi issue [MAJOR] và mục needs_review / image_escalate của batch đã đóng
REWORK if: không có lỗi critical nhưng có metric dưới ngưỡng PASS và chưa chạm ngưỡng REJECT, hoặc còn issue [MAJOR] mở
REJECT / ESCALATE if: có ≥ 1 lỗi critical ở gate cuối (reject batch, review lại 100%); hoặc state accuracy < 85% /
  relevance accuracy < 80% (nghi guideline gap → escalate spec owner, không rework mù); hoặc tỉ lệ escalate > 20%
  (dừng batch, sửa guideline trước)
```

Trade-off: critical đặt ngưỡng **0** vì một đèn đỏ bị đọc sai là đủ gây tai nạn — chấp nhận tốn công review 100% vùng
rủi ro. Relevance đặt thấp hơn state (90% so với 95%) vì relevance là suy luận ngữ cảnh, hai người hợp lý có thể bất
đồng; phần bất đồng đó phải đi qua `unknown` + escalate chứ không ép đúng. Geometry chỉ cần 90% vì lệch box vài pixel
không đổi quyết định dừng/đi. Ngưỡng chặt hơn sẽ làm rework phình to với lỗi minor; ngưỡng lỏng hơn cho lỗi đỏ/xanh lọt
xuống xe.
