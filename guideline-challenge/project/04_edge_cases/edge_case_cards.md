# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

Toạ độ `(x1,y1)-(x2,y2)` tính bằng pixel ở ảnh gốc, gốc toạ độ góc trên trái; chỉ để tìm đúng đèn, không phải box gold.

---

CASE ID: EC01
Sample: LISA24 (example)
Scene: giao lộ lúc chạng vạng, xe ego dừng ở làn đi thẳng
Observation: trên cần treo có đầu đèn mũi tên rẽ trái (~747,162) đang **đỏ** ngay cạnh đầu đèn đi thẳng (~940,162) đang **xanh**; cột phải (~1155,200) xanh
Decision: LABEL
Expected: 3 box. Đi thẳng + cột phải: `green`, `circle`, `relevant`. Mũi tên: `red`, `arrow_left`, `not_relevant`. Không tag `conflicting_lights`
Rationale: contract quy ước ego đi thẳng; gán mũi tên đỏ là relevant → xe phanh khi đèn xanh (failure b)
Common mistake: coi đỏ + xanh là mâu thuẫn rồi escalate; hoặc chọn `pictogram = circle` cho mũi tên
Diversity: conflict · ambiguity

---

CASE ID: EC02
Sample: BDD12 (example)
Scene: phố thương mại ban ngày, trạm xăng bên phải
Observation: hộp đèn **đi bộ** hình bàn tay đỏ trên cột phải (~1090,140); không có đầu đèn xe cơ giới nào quay mặt về camera
Decision: IGNORE
Expected: ảnh không có box `traffic_light` nào, không tag
Rationale: đèn đi bộ đỏ label thành đèn xe relevant → false STOP (failure c)
Common mistake: vẽ box `red` cho bàn tay đỏ
Diversity: negative · critical

---

CASE ID: EC03
Sample: BDD18 (calibration)
Scene: đêm, phố có xe đỗ hai bên, ego đang ở vạch sang đường
Observation: 2 bóng xanh rất xa giữa đường (~587,307 và ~660,307), không thấy vỏ; hộp đèn đi bộ sáng (bàn tay đỏ + người trắng) ở cột phải (~1160-1200,262); không thấy đèn xe nào của giao lộ ngay trước
Decision: LABEL + ESCALATE (object)
Expected: 2 box ôm bóng sáng, `green`, `circle`, `relevance = unknown`, `needs_review` tick; đèn đi bộ không vẽ
Rationale: không chứng minh được 2 đèn thuộc giao lộ gần nhất; đoán `relevant` hay `not_relevant` đều có thể sai theo hướng critical → escalation path của contract
Common mistake: gán `relevant` vì "chỉ có mỗi đèn đó"; box ôm cả quầng sáng
Diversity: low_visibility · escalation · small_far

---

CASE ID: EC04
Sample: BDD22 (calibration)
Scene: đường cao tốc lúc hoàng hôn, giá long môn rất xa
Observation: dãy 3–4 bóng xanh trên giá long môn ở tâm ảnh (~640-680,360-375), mỗi bóng rộng khoảng 5–7 px, nhoè
Decision: LABEL nếu box ≥ 6 px, IGNORE nếu < 6 px
Expected: đèn rộng ≥ 6 px: box, `green`, pictogram theo hình nhìn thấy (không rõ → `unknown`), relevance theo làn (không rõ → `unknown` + `needs_review`); đèn < 6 px không vẽ
Rationale: detector không học được từ đèn < 6 px; ngưỡng cứng giúp hai annotator ra cùng số lượng
Common mistake: ước lượng bằng mắt thay vì đo box; bỏ sót cả dãy vì không phóng to
Diversity: small_far

---

CASE ID: EC05
Sample: BDD25 (calibration)
Scene: chạng vạng, mưa, đại lộ nhiều làn
Observation: nhiều bóng xanh trên cột và cần treo (~580,254; ~832,242; ~865,262); vệt đỏ/xanh kéo dài trên mặt đường ướt; đèn đỏ nhỏ ở cột phải (~987,283)
Decision: LABEL đèn thật, IGNORE phản chiếu
Expected: box cho các đầu đèn; vệt màu trên mặt đường không vẽ; đèn đỏ cột phải: nếu thấy là đèn xe quay mặt thì vẽ, relevance theo làn, không chắc → `unknown` + `needs_review`
Rationale: phản chiếu label thành đèn → false detection (failure c)
Common mistake: vẽ vệt đỏ trên mặt đường thành đèn đỏ
Diversity: low_visibility · ambiguity

---

CASE ID: EC06
Sample: BDD02 (calibration)
Scene: đại lộ Manhattan, giao lộ ngay trước có vạch sang đường
Observation: đầu đèn xanh quay mặt ở cột trái (~255,100), cần treo giữa (~790,140), cột phải (~1010,150; ~1218,120); đầu đèn vàng góc trên phải (~1240,45) chỉ thấy **cạnh vỏ**; vài đèn nhỏ ở giao lộ xa cuối phố
Decision: LABEL (nhiều relevant) + IGNORE đầu quay ngang
Expected: các đầu quay mặt ở giao lộ gần: `green`, `circle`, `relevant`; đèn giao lộ xa ≥ 6 px: `not_relevant`; đầu đèn góc trên phải: không vẽ
Rationale: một giao lộ có nhiều đầu đèn lặp cho cùng một hướng — downstream cần tất cả để bỏ phiếu STOP/GO
Common mistake: chỉ gán `relevant` cho một đèn "gần tâm nhất"; vẽ đầu quay ngang vì thấy vỏ vàng
Diversity: conflict · small_far · ambiguity

---

CASE ID: EC07
Sample: BDD15 (example)
Scene: phố ban ngày nắng gắt, kính xe mờ
Observation: đầu đèn ở cột trái xa (~280,190) không phân biệt được ô nào sáng; vài chấm đỏ nhỏ phía xa (~392,193)
Decision: LABEL + UNKNOWN
Expected: đầu đèn rộng ≥ 6 px: box, `state = unknown`, `pictogram = unknown`, `needs_review` tick
Rationale: đoán màu trên ảnh nhoè tạo nhãn sai "tự tin"; `unknown` giữ được tín hiệu "có đèn ở đây" cho detector
Common mistake: đoán `red` theo màu nền; bỏ qua không vẽ
Diversity: low_visibility · small_far

---

CASE ID: EC08
Sample: BDD07 (blind)
Scene: phố dân cư Brooklyn, ban ngày trời trong, giao lộ ngay trước
Observation: 2 đầu đèn xanh **quay mặt**: cột trái (~229,124)-(242,157) và cần treo phải (~680,150); cạnh mỗi đầu có một đầu đèn **quay ngang** (chỉ thấy cạnh vỏ vàng); nắp capo phản chiếu 2 đèn xanh (~y 600-680); đèn nhỏ xa ở giao lộ kế (~494,227; ~602,228)
Decision: LABEL 2 đầu quay mặt, IGNORE đầu quay ngang + phản chiếu capo
Expected: 2 box `green`, `circle`, `relevant`; không box nào ở vùng capo; box đèn trái không gồm đầu quay ngang bên cạnh
Rationale: phản chiếu capo label thành đèn → false detection ngay trước xe (critical, failure c); đầu quay ngang điều khiển đường cắt ngang
Common mistake: gộp 2 vỏ sát nhau vào một box; vẽ phản chiếu
Diversity: critical · conflict · small_far

---

CASE ID: EC09
Sample: BDD26 (blind)
Scene: đêm, đại lộ có dải phân cách, loá mạnh
Observation: đèn xanh lớn loá trên cột trái (~487,70); hộp đèn đi bộ bàn tay cam ngay dưới (~437,120); ở giao lộ kế tiếp phía xa có đèn đỏ nhỏ (~667,217) và vài đèn xanh
Decision: LABEL + IGNORE đèn đi bộ + `not_relevant` cho giao lộ kế
Expected: đèn xanh gần: box ôm bóng sáng (không gồm quầng), `green`, `relevant`; đèn đi bộ: không vẽ; đèn đỏ xa (nếu ≥ 6 px): `red`, `not_relevant`
Rationale: gán đèn đỏ giao lộ kế là relevant → xe phanh gấp khi đang được đi (failure b); bàn tay cam → false STOP (failure c)
Common mistake: thấy đèn đỏ nổi bật giữa đường liền gán relevant; box ôm cả quầng loá
Diversity: critical · low_visibility

---

CASE ID: EC10
Sample: BDD17 (blind)
Scene: mưa, nhiều giọt nước trên kính, phố Manhattan
Observation: bóng đỏ trên cột trái (~402,190) và bóng đỏ thấp hơn (~428,212); bóng xanh phía trước (~515,185) và bên phải (~578,195); mọi thứ nhoè, không thấy vỏ, không phân định được làn
Decision: ESCALATE
Expected: tag `image_escalate` (`reason = conflicting_lights` hoặc `low_visibility`); các đèn vẽ được thì `relevance = unknown` + `needs_review`; không đèn nào `relevant` mà thiếu `needs_review`
Rationale: chọn đỏ hoặc xanh đều có thể là failure (a) hoặc (b); contract yêu cầu escalate thay vì đoán
Common mistake: chọn đèn xanh giữa làm relevant vì nằm gần tâm ảnh
Diversity: escalation · ambiguity · conflict · low_visibility

---

CASE ID: EC11
Sample: BDD11 (blind)
Scene: phố dân cư Queens, ban ngày u ám, có vạch sang đường
Observation: chỉ có hộp đèn đi bộ bàn tay đỏ trên cột phải (~1205,260); không có đầu đèn xe cơ giới nào quay mặt về camera
Decision: IGNORE
Expected: ảnh không có box `traffic_light` nào
Rationale: bàn tay đỏ label thành đèn đỏ relevant → xe dừng giữa đường vô cớ (failure c)
Common mistake: vẽ đèn đi bộ; vẽ chấm đỏ đèn hậu xe phía xa
Diversity: negative · critical

---

CASE ID: EC12
Sample: BDD21 (blind)
Scene: đại lộ ven công viên, ban ngày
Observation: đầu đèn xanh trên cần treo (~332,282)-(342,308) ngay phía trước; bóng xanh nhỏ lẫn trong tán cây (~225,318); vật tối bị cắt ở góc trên trái; cặp hộp vàng nhỏ ở cột trái (~60,345)
Decision: LABEL
Expected: đầu đèn cần treo: `green`, `circle`, `relevant`, box ôm vỏ không gồm cần treo; các vật còn lại chỉ vẽ nếu thấy rõ mặt đèn xe và ≥ 6 px
Rationale: case normal — kiểm peer làm đúng quy trình cơ bản và geometry trên đèn nhỏ (~10 px)
Common mistake: box gồm cả cần treo; gán `not_relevant` vì đèn nhỏ
Diversity: normal · small_far · truncation
