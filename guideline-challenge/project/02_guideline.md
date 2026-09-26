# Annotation guideline — Traffic light: state + ego relevance

**Version:** v1

<!--
v0 = chưa có bản nháp. Đổi dòng Version ở trên thành v1 khi xong bản nháp đầu, v2 sau calibration, v3 sau blind
handoff; mỗi lần tăng version ghi một dòng vào 08_revision_log.md. `make freeze` đòi v2 trở lên.

File này là thứ nhóm peer nhận nguyên văn trong blind pack và là Guide dán vào CVAT. Peer KHÔNG nhận
edge_case_cards.md, gold_decisions.csv hay sample_pack.csv. Rule nào peer cần biết phải nằm ở đây.
No hidden rules: rule chỉ giải thích bằng miệng thì coi như không tồn tại.
Ví dụ trong guideline chỉ dùng ảnh split example hoặc calibration, không dùng ảnh blind.
-->

## 1. Objective + scope

**Mục đích.** Dữ liệu dùng để huấn luyện và kiểm thử module nhận diện đèn giao thông của xe tự lái / ADAS. Với mỗi đầu
đèn, module cần biết (a) đèn đang hiển thị màu gì (`state`) và (b) đèn đó có điều khiển làn của xe đang quay camera
(**ego vehicle**) hay không (`relevance`). Planner dựa vào đèn `relevant` để quyết định dừng hay đi.

**Lỗi nghiêm trọng nhất (critical):** một đèn **đỏ hoặc vàng đang điều khiển ego lane** bị bỏ sót, bị gán
`state=green`, hoặc bị gán `relevance=not_relevant`. Khi phân vân, luôn chọn phương án không làm mất thông tin này
(xem mục 7).

**Trong scope — bắt buộc vẽ box:**

- Mọi đầu đèn tín hiệu **dành cho xe** (đèn tròn và đèn mũi tên) mà nhìn thấy được vỏ đèn hoặc bóng đèn đang sáng.
- Box cao **≥ 8 px** (đo ở độ phân giải gốc, không đo khi đang thu nhỏ ảnh).
- Kể cả đèn quay lưng / quay ngang so với camera, đèn không sáng, đèn thuộc làn khác hoặc giao lộ khác.

**Ngoài scope — không vẽ:** xem danh sách ở mục 5.

## 2. Annotation unit

- **Đơn vị:** một ảnh tĩnh. Mỗi ảnh được label độc lập.
- **Instance:** một **đầu đèn** = một vỏ đèn vật lý (một khối chứa 3, 4 hoặc 5 bóng). Mỗi đầu đèn là một box
  `traffic_light` riêng.
- Hai đầu đèn gắn cạnh nhau trên cùng cần treo hoặc cùng cột → **hai box**, kể cả khi cùng màu.
- Đầu đèn 5 bóng dạng chữ T hoặc hai cột bóng (đèn tròn + mũi tên chung một vỏ) → **một box**.
- Mỗi ảnh có thể có thêm **một** tag `image_escalate` (xem mục 7). Không dùng tag này cho từng đèn.

## 3. Geometry rule

- **Shape:** rectangle (`traffic_light`).
- **Box ôm phần vỏ đèn nhìn thấy được:** gồm mặt đèn và các bóng. **Không** gồm cột, cần treo, giá đỡ, tấm nền đen
  (backplate) phía sau vỏ đèn, hay quầng loá quanh bóng.
- **Đèn bị che hoặc bị cắt mép ảnh:** chỉ ôm phần nhìn thấy (visible box, không đoán phần bị che) và tick `occluded`.
- **Ban đêm / ngược sáng, không thấy vỏ đèn, chỉ thấy bóng sáng:** box ôm **lõi sáng** của bóng (phần sáng đặc,
  không lấy quầng toả). Chiều cao của box này dùng để xét ngưỡng 8 px.
- **Tolerance:** mỗi cạnh lệch tối đa **2 px** với đèn cao ≥ 20 px; tối đa **1 px** với đèn cao 8–19 px. Phóng to
  ảnh khi vẽ đèn nhỏ.

## 4. Taxonomy

Một class `traffic_light` (rectangle) với 5 attribute, cộng một tag ảnh `image_escalate`. Bảng đầy đủ (kèm
rationale) ở `03_ontology_and_cvat_setup.md` — hai nơi phải khớp nhau.

| Attribute | Giá trị cho phép | Default | Ghi chú |
|---|---|---|---|
| `state` | `red` · `yellow` · `green` · `off` · `unknown` | `__undefined__` | **Bắt buộc chọn** |
| `relevance` | `relevant` · `not_relevant` · `unknown` | `__undefined__` | **Bắt buộc chọn** |
| `pictogram` | `circle` · `arrow_left` · `arrow_right` · `arrow_straight` · `unknown` | `__undefined__` | **Bắt buộc chọn** |
| `occluded` | checkbox | false | Tick khi vỏ đèn bị che hoặc cắt mép ảnh |
| `needs_review` | checkbox | false | Tick khi cần QA xem lại (mục 7) |

`__undefined__` chỉ là giá trị khởi tạo. **Export còn `__undefined__` ở bất kỳ box nào = lỗi chưa gán.** Không có
giá trị nào được tự điền thay annotator.

### 4.1 `state` — đèn đang hiển thị gì

- Chọn màu của **bóng đang sáng**. Khi màu bị biến dạng (đèn đỏ ban đêm trông cam hoặc trắng, đèn xanh trông xanh
  ngọc), xác định theo **vị trí bóng** trong vỏ đèn: đèn dọc — trên cùng `red`, giữa `yellow`, dưới cùng `green`;
  đèn ngang — trái `red`, giữa `yellow`, phải `green`.
- `off`: thấy rõ vỏ đèn và **không bóng nào sáng**, ảnh đủ sáng để chắc chắn (không bị cháy sáng, không loá).
- `unknown`: đèn quay lưng / quay ngang (không thấy mặt đèn); bóng sáng bị che; màu và vị trí đều không kết luận
  được; phân vân giữa `off` và đang sáng.
- Đầu đèn có **nhiều bóng sáng cùng lúc** (ví dụ tròn đỏ + mũi tên xanh rẽ trái): `state` và `pictogram` lấy theo
  **bóng áp dụng cho hướng đi của ego** (quy ước hướng đi ở mục 4.2). Không xác định được bóng nào áp dụng →
  `state=unknown` + `needs_review`.

### 4.2 `relevance` — đèn có điều khiển ego lane không

Quy ước hướng đi của ego (ảnh đơn không cho biết ý định rẽ): **ego đi theo làn đang đứng**. Nếu ego không nằm trong
làn rẽ riêng (không có mũi tên rẽ vẽ trên mặt đường của làn ego, không nằm trong làn rẽ tách riêng), coi như **ego
đi thẳng**.

`relevant` khi **đủ cả ba** điều kiện:

1. Đèn thuộc **giao lộ gần nhất phía trước** ego (giao lộ có vạch dừng mà ego sẽ gặp đầu tiên).
2. Mặt đèn **quay về phía camera**.
3. Đèn điều khiển hướng đi của ego: đèn tròn (`circle`) hoặc mũi tên cùng hướng với ego (ego đi thẳng →
   `arrow_straight`; ego ở làn rẽ trái → `arrow_left`...). Giao lộ có nhiều đầu đèn tròn cùng điều khiển hướng
   đi thẳng → **tất cả** đều `relevant`.

`not_relevant` khi chắc chắn một trong các trường hợp: đèn quay lưng / quay ngang; đèn thuộc giao lộ xa hơn giao lộ
gần nhất; đèn mũi tên cho hướng ego không đi (ví dụ ego đi thẳng, đèn chỉ có mũi tên rẽ trái treo trên làn rẽ trái);
đèn cho đường cắt ngang.

`unknown` khi không đủ bằng chứng để chọn một trong hai giá trị trên — luôn kèm `needs_review` (mục 7).

### 4.3 `pictogram` — hình của bóng đang sáng

- Chọn hình của bóng đang sáng (hoặc bóng áp dụng cho ego, theo mục 4.1). `arrow_left`/`arrow_right`/
  `arrow_straight` là mũi tên trái / phải / thẳng; `circle` là bóng tròn đầy.
- Đèn `off`: chọn theo hình in trên mặt kính nếu nhìn ra; không nhìn ra → `unknown`.
- Đèn quay lưng, bóng bị che, đèn quá nhỏ hoặc loá không phân biệt tròn / mũi tên → `unknown`.

### 4.4 `occluded`

Tick khi **bất kỳ phần nào** của vỏ đèn bị vật khác che (cây, xe, biển báo, cột) hoặc bị cắt ở mép ảnh. Nếu phần bị
che là bóng đang sáng → thêm `state=unknown`.

## 5. Inclusion / exclusion

**LABEL (vẽ box):**

- Đầu đèn tín hiệu cho xe cao ≥ 8 px, dù sáng, tắt, quay lưng hay thuộc làn / giao lộ khác.
- Đèn ở cả hai phía giao lộ (phía gần và phía xa) nếu đủ ngưỡng kích thước.
- Đèn tín hiệu cho xe gắn tạm (trên giá di động ở công trường).
- Vật thể trông như đèn tín hiệu cho xe nhưng không chắc → **vẫn vẽ** và tick `needs_review` (không bỏ qua).

**IGNORE (không vẽ):**

- Đèn cho người đi bộ (hình người, bàn tay, đồng hồ đếm ngược) và đèn cho xe đạp.
- Đèn của phương tiện: đèn phanh, đèn hậu, đèn xi-nhan, đèn ưu tiên.
- Đèn đường, đèn trang trí, đèn biển quảng cáo, biển báo giao thông (kể cả biển có viền phản quang).
- Hình phản chiếu của đèn trên kính, thân xe, vũng nước.
- Đèn cao < 8 px.

## 6. Visibility / occlusion

| Tình huống | Box | `occluded` | `state` / `pictogram` | Escalate |
|---|---|---|---|---|
| Vỏ đèn bị che một phần, bóng sáng vẫn thấy | Ôm phần nhìn thấy | ✓ | Theo bóng sáng | — |
| Bóng sáng bị che, chỉ thấy phần vỏ | Ôm phần nhìn thấy | ✓ | `unknown` / `unknown` | `needs_review` nếu đèn có thể `relevant` |
| Bị cắt ở mép ảnh | Ôm phần trong ảnh | ✓ | Theo phần nhìn thấy | — |
| Nhỏ / xa nhưng ≥ 8 px, nhìn ra màu | Ôm vỏ hoặc lõi sáng | — | Theo màu / vị trí; hình không rõ → `pictogram=unknown` | — |
| Ban đêm chỉ thấy bóng sáng | Ôm lõi sáng | — | Theo màu | `needs_review` nếu không chắc là đèn tín hiệu |
| Loá / cháy sáng, không phân biệt màu | Ôm vỏ hoặc vùng sáng | — | `unknown` | `needs_review` |
| Đèn quay lưng / quay ngang | Ôm vỏ | — | `unknown` / `unknown` | — (`relevance=not_relevant`) |

## 7. Ambiguity / escalation

Nguyên tắc: **không đoán**. Thiếu bằng chứng thì ghi nhận là thiếu, để QA quyết.

| Quyết định | Khi nào | Thể hiện trong CVAT |
|---|---|---|
| **LABEL** | Object trong scope (mục 5) | Box `traffic_light` với đủ `state`, `relevance`, `pictogram` (không còn `__undefined__`) |
| **IGNORE** | Object ngoài scope (mục 5) | Không có box |
| **UNKNOWN** | Đã vẽ box nhưng thiếu bằng chứng cho **một attribute** | Attribute đó = `unknown` |
| **ESCALATE (đèn)** | Một đèn cần QA xem lại | Tick `needs_review` trên box đó |
| **ESCALATE (ảnh)** | Không xác định được đèn nào điều khiển ego lane cho cả ảnh | Thêm tag `image_escalate` cho ảnh |

**Rule bắt buộc:**

1. `relevance=unknown` → **luôn** tick `needs_review`.
2. **Đèn đỏ hoặc vàng mà phân vân `relevant` / `not_relevant` → chọn `unknown` + `needs_review`, không bao giờ chọn
   `not_relevant`.** Đây là rule chặn lỗi critical.
3. Phân vân giữa hai màu (ví dụ đỏ / vàng lúc chạng vạng) và vị trí bóng cũng không giúp được → `state=unknown` +
   `needs_review`. Không chọn màu "gần đúng".
4. Tag `image_escalate` khi ảnh có đèn nhưng **không đèn nào** đạt được `relevant` hay `not_relevant` một cách chắc
   chắn **và** không xác định được ego đang ở làn nào (ví dụ giao lộ nhiều nhánh, không thấy vạch làn). Vẫn vẽ box và
   gán attribute cho từng đèn như bình thường.
5. `unknown` không phải lỗi. Gán `unknown` đúng chỗ tốt hơn đoán sai.

## 8. Temporal rule

Không áp dụng — task ảnh tĩnh. Ảnh từ cùng một clip (LISA) vẫn label **độc lập từng ảnh**: không suy trạng thái đèn
từ ảnh trước / sau, không suy đèn nhấp nháy hay chuyển pha. Mọi attribute chỉ dựa trên những gì thấy trong ảnh
đang label.

## 9. Examples

Ảnh LISA là cảnh giao lộ lúc chạng vạng, camera dừng ở vạch trước giao lộ; xe có đèn pha ở bên trái là xe ngược
chiều → ego ở làn đi thẳng. Trên cần treo phía trước có 3 đầu đèn gần (trái → phải) và một nhóm đèn nhỏ ở xa phía sau
giao lộ.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| LISA01 | Đầu đèn gần bên trái, bóng trên cùng sáng hình mũi tên rẽ trái, màu cam-đỏ; biển cấm quay đầu bên cạnh | Box ôm vỏ đèn (không gồm biển, cần treo) · `red` · `not_relevant` · `arrow_left` | 3, 4.1 (vị trí bóng trên cùng → red), 4.2 (ego đi thẳng, mũi tên rẽ trái không áp dụng) |
| LISA01 | Đầu đèn gần ở giữa, bóng tròn trên cùng sáng, màu cam | Box ôm vỏ · `red` · `relevant` · `circle` | 4.1 (màu lệch → theo vị trí), 4.2 (đèn tròn, giao lộ gần nhất, quay về camera) |
| LISA01 | Đầu đèn gần bên phải, vỏ đèn gần như lẫn vào nền cây tối, chỉ rõ một bóng tròn đỏ | Box ôm phần vỏ nhìn thấy (hoặc lõi sáng nếu không thấy vỏ) · `red` · `relevant` · `circle` | 3 (không thấy vỏ → ôm lõi sáng), 4.2 (nhiều đèn tròn cùng điều khiển → tất cả relevant) |
| LISA01 | Các đèn nhỏ ở xa phía sau giao lộ, bóng đỏ, cao khoảng 10–20 px | Mỗi đầu đèn một box · `red` · `not_relevant` · `circle` (hoặc `unknown` nếu không phân biệt được hình) | 1 (≥ 8 px), 4.2 (giao lộ xa hơn giao lộ gần nhất), 4.3 |
| LISA30 | Cùng cảnh; đèn mũi tên rẽ trái vẫn đỏ, hai đầu đèn tròn gần chuyển xanh (màu xanh ngọc, bóng dưới cùng) | Đèn trái: `red` · `not_relevant` · `arrow_left`. Đèn giữa và phải: `green` · `relevant` · `circle` | 4.1 (bóng dưới cùng → green), 8 (label độc lập, không suy từ LISA01) |
| LISA01 | Đèn phanh / đèn pha của xe trên đường; biển cấm quay đầu | Không có box | 5 (IGNORE) |

## 10. Common mistakes

| Lỗi | Hậu quả | Cách tránh |
|---|---|---|
| Để `__undefined__` ở `state` / `relevance` / `pictogram` | Export không dùng được, tính là lỗi | Kiểm tra từng box trước khi lưu; mỗi box phải đủ 3 attribute |
| Gán `not_relevant` cho đèn đỏ khi không chắc | **Critical** — xe vượt đèn đỏ | Rule 7.2: phân vân → `unknown` + `needs_review` |
| Chỉ vẽ một box cho hai đầu đèn cạnh nhau | Sai số đèn, mất đèn relevant | Mỗi vỏ đèn một box (mục 2) |
| Box gồm cả backplate, cần treo hoặc quầng loá | Sai geometry | Chỉ ôm vỏ đèn nhìn thấy / lõi sáng (mục 3) |
| Đoán màu đèn đỏ ban đêm là `yellow` vì trông cam | Sai `state` | Xác định theo vị trí bóng (mục 4.1) |
| Gán `relevant` cho mũi tên rẽ khi ego đi thẳng | Planner dùng sai đèn | Xét pictogram với hướng đi của ego (mục 4.2) |
| Gán `relevant` cho đèn ở giao lộ xa hơn | Planner phản ứng sớm với đèn không áp dụng | Chỉ giao lộ gần nhất (mục 4.2) |
| Bỏ qua đèn nhỏ ở xa hoặc đèn quay lưng | Thiếu instance | Vẽ mọi đèn ≥ 8 px (mục 1, 5) |
| Vẽ đèn đi bộ hoặc phản chiếu trên kính | Thừa instance | Xem danh sách IGNORE (mục 5) |
| Suy trạng thái từ frame LISA trước / sau | Label không phản ánh ảnh đang xét | Label độc lập từng ảnh (mục 8) |
