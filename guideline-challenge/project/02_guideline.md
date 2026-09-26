# Annotation guideline — Vehicle traffic light: state, ego relevance, direction

**Version:** v2
**Trạng thái:** Bản nháp đầu dùng cho calibration nội bộ

Guide này là toàn bộ thông tin annotator được dùng khi gắn nhãn. Nếu một tình huống không được giải quyết bằng rule
bên dưới, annotator không tự đoán mà dùng cơ chế `unknown` / `needs_review` / `image_escalate` ở mục 7.

## 1. Objective + scope

### Mục tiêu

Gắn nhãn **đèn giao thông dành cho xe** để phục vụ ba bước downstream của hệ thống ADAS/AV:

1. phát hiện từng đầu đèn bằng bounding box;
2. phân loại trạng thái và pictogram của từng đầu đèn;
3. chọn đầu đèn điều khiển làn hiện tại của ego vehicle để planner quyết định dừng hay đi.

Mỗi output gồm class `traffic_light`, box hình chữ nhật và năm attribute:

- `state`: màu/trạng thái đang thể hiện;
- `relevance`: đầu đèn có điều khiển ego lane hay không;
- `pictogram`: hình tròn hoặc hướng mũi tên — đây là trường lưu **direction**;
- `occluded`: housing có bị một vật thể thật che hay không;
- `needs_review`: object cần QA owner xem lại hay không.

Khi không xác định được tín hiệu nào điều khiển ego lane trên toàn ảnh, dùng thêm image tag `image_escalate`.

### Rủi ro downstream

Các lỗi sau là **critical**:

- bỏ sót một đèn đỏ/vàng đang điều khiển ego lane;
- gán đèn đỏ/vàng relevant thành `green` hoặc `not_relevant`;
- dùng state của một head/làn khác cho head điều khiển ego.

Gán đèn xanh của làn khác thành relevant là lỗi **major**, vì planner có thể áp tín hiệu đi của movement khác cho ego.

### Phạm vi ngắn gọn

- **Label:** mọi đầu đèn dành cho xe đã xác nhận, có chiều cao nhìn thấy từ 8 px trở lên, kể cả head không relevant,
  quay lưng, nhỏ/xa, bị che hoặc nằm sát mép ảnh.
- **Ignore:** pedestrian/bicycle signal, đèn phương tiện, đèn đường, biển hiệu, reflection/flare, object dưới 8 px và
  chấm màu không đủ bằng chứng là vehicle signal.
- Dataset là ảnh tĩnh. Không suy state, relevance hoặc direction từ frame trước/sau.

## 2. Annotation unit

### Đơn vị gắn nhãn

- Một **đầu đèn/housing vật lý** là một instance `traffic_light`.
- Dùng **Shape → Rectangle**, không dùng Track, polygon hoặc polyline.
- Hai housing khác nhau phải có hai box, dù cùng gắn trên một cần treo, cùng màu và cùng relevance.
- Nhiều bóng red/yellow/green nằm trong cùng một vỏ đèn dọc vẫn là **một** instance.
- Không gộp cả cụm giao lộ, cột, cần treo hoặc nhiều housing vào một box.

### Khi nào tạo instance mới

Tạo instance mới khi có đường biên/vỏ riêng biệt cho một mặt tín hiệu. Hai mặt tín hiệu quay về hai hướng khác nhau
được xem là hai instance nếu nhìn thấy hai housing riêng. Không tách quầng sáng, reflection hoặc từng bóng màu trong
cùng một housing thành instance riêng.

Nếu một housing hiếm gặp hiển thị đồng thời nhiều pictogram mà schema một giá trị không biểu diễn được, không tách
box giả. Giữ một box, gán `pictogram=unknown`, bật `needs_review=true`; chỉ thêm `image_escalate` nếu việc này làm cho
ego relevance không thể xác định.

## 3. Geometry rule

### Cách vẽ box

1. Vẽ rectangle **tight** quanh phần housing nhìn thấy được.
2. Bao gồm vỏ/head và chụp che sáng gắn trực tiếp với head nếu đường biên của chúng nhìn thấy.
3. Không bao gồm cột, dây, cần treo, biển báo, tấm nền lớn phía sau, bầu trời, cây hoặc vùng nền.
4. Không mở rộng box theo quầng sáng, bloom hoặc reflection.
5. Mỗi housing một box; không vẽ một box dài chứa nhiều head.

Đây là rule **visible-only**, không phải amodal. Không ước lượng phần housing nằm sau xe, cây, cột hoặc ngoài ảnh.

### Khi chỉ thấy bóng sáng

Nếu housing tối nhưng bóng sáng vẫn có đủ bằng chứng là vehicle signal — ví dụ vị trí gắn, cấu trúc mặt đèn và quan hệ
với các head lân cận còn nhận ra — vẽ rectangle nhỏ nhất quanh **phần mặt tín hiệu nhìn thấy**, loại quầng halo. Không
suy rộng tới housing không nhìn thấy. Bật `needs_review=true` nếu biên housing không đủ rõ.

Một đốm đỏ/vàng/xanh đơn lẻ không có cấu trúc hoặc bằng chứng vehicle signal thì IGNORE, không dùng `unknown` để biến
mọi nguồn sáng thành traffic light.

### Ngưỡng kích thước

- Đo trên ảnh ở kích thước gốc, không đo trên ảnh đã phóng đại.
- Chiều cao là chiều cao của tight box quanh **phần signal/housing nhìn thấy**, sau khi bỏ halo và phần bị che.
- `height >= 8 px`: thuộc scope.
- `height < 8 px`: IGNORE.
- Đúng 8 px vẫn phải label.

### Tolerance khi review

- Đèn cao từ 20 px trở lên: mỗi cạnh box được lệch tối đa 2 px so với phần housing nhìn thấy.
- Đèn cao từ 8 đến dưới 20 px: mỗi cạnh được lệch tối đa 1 px.

Nếu object bị che một phần, tolerance áp dụng cho biên **đang nhìn thấy**, không áp dụng cho phần bị suy đoán.

## 4. Taxonomy

Không tạo class riêng cho từng màu hoặc hướng. `state`, `relevance` và `pictogram` là attribute của cùng một object
`traffic_light`, tránh nổ ra nhiều tổ hợp class và giữ được một geometry thống nhất.

| Tên | Loại | Allowed values / default | Cách dùng |
|---|---|---|---|
| `traffic_light` | class, rectangle | một housing vật lý | Class duy nhất cho vehicle traffic-light head. |
| `state` | select, mutable | `red`, `yellow`, `green`, `off`, `unknown`; default `__undefined__` | Màu/trạng thái của head trong chính ảnh hiện tại. |
| `relevance` | select, mutable | `relevant`, `not_relevant`, `unknown`; default `__undefined__` | Quan hệ giữa head và ego lane. |
| `pictogram` | select | `circle`, `arrow_left`, `arrow_right`, `arrow_straight`, `unknown`; default `__undefined__` | Hình của tín hiệu đang thể hiện; đây là direction. |
| `occluded` | checkbox | `false`, `true`; default `false` | Chỉ true khi một vật thể thật che một phần housing. |
| `needs_review` | checkbox | `false`, `true`; default `false` | True khi có attribute/object-level ambiguity cần QA xem. |
| `image_escalate` | image tag | có hoặc không có tag | Dùng cho image-level ambiguity về tín hiệu điều khiển ego. |

`state` và `relevance` có `mutable=true` trong schema để hỗ trợ trường hợp video về sau; task hiện tại dùng ảnh tĩnh nên
annotator vẫn gán độc lập trên từng ảnh. Không có giá trị `red_yellow` và không có attribute tên `direction`.

### `state`

- `red`: bóng đỏ đang sáng.
- `yellow`: bóng vàng/amber đang sáng.
- `green`: bóng xanh đang sáng.
- `off`: thấy rõ mặt trước của housing trong điều kiện đủ sáng và xác nhận không bóng nào đang sáng.
- `unknown`: không đủ bằng chứng để đọc state, ví dụ head quay lưng, quá mờ, bị che hoặc exposure làm mất màu.

Không dùng `off` cho head quay lưng hoặc quá mờ. Nếu thấy màu rõ nhưng các attribute khác không rõ, giữ state cụ thể
và chỉ đặt attribute thiếu bằng chứng thành `unknown`.

### `relevance`

Gán theo thứ tự bằng chứng sau:

1. hướng mặt của head;
2. lane/movement mà head được đặt thẳng hàng hoặc treo phía trên;
3. vạch làn, mũi tên mặt đường, stop line và cấu trúc giao lộ;
4. sự lặp lại của các head cùng điều khiển một movement.

- `relevant`: có bằng chứng head điều khiển ego lane. Có thể có nhiều head relevant cùng lúc nếu chúng là tín hiệu lặp
  cho cùng movement.
- `not_relevant`: head quay lưng, điều khiển cross traffic, hoặc điều khiển làn/movement tách biệt mà ego lane không
  thuộc về.
- `unknown`: thấy vehicle signal nhưng không đủ bằng chứng liên kết head với lane/movement.

Không suy relevance chỉ từ màu, vị trí trái/phải trong ảnh hoặc việc head nằm phía trước camera.

### `pictogram`

- `circle`: bóng tròn thông thường.
- `arrow_left`, `arrow_right`, `arrow_straight`: dùng khi hình mũi tên tương ứng đọc được trực tiếp.
- `unknown`: bloom, blur, occlusion hoặc kích thước làm hình không đọc được.

Không suy pictogram từ hướng đường hoặc lane marking khi hình trên đèn không nhìn thấy. Một head đỏ vẫn có thể là
`arrow_left`; state và pictogram phải được gán độc lập.

### Checkbox `occluded` trong CVAT

Giá trị dùng để chấm là **attribute checkbox `occluded` của label `traffic_light`**. Nếu phiên bản CVAT còn hiển thị
một cờ Occluded tích hợp riêng cho shape, có thể tick cùng trạng thái để giao diện nhất quán, nhưng cờ đó không thay
thế custom attribute; export không được thiếu attribute `occluded`.

Mọi box phải kết thúc với `state`, `relevance`, `pictogram` đã gán. `__undefined__` còn lại trong export là lỗi.

## 5. Inclusion / exclusion

### Bắt buộc LABEL

- Vehicle signal nhìn thấy housing và cao ít nhất 8 px.
- Vehicle signal chỉ thấy bóng sáng nhưng vẫn xác nhận được từ cấu trúc/ngữ cảnh và đạt ngưỡng 8 px theo mục 3.
- Head relevant và not relevant.
- Head quay lưng/quay sang hướng khác: `state=unknown`, `relevance=not_relevant`; pictogram theo bằng chứng, thường là
  `unknown`; bật `needs_review=true` khi có giá trị unknown.
- Head bị che một phần: box phần nhìn thấy và `occluded=true`.
- Head bị cắt ở mép ảnh nhưng phần nhìn thấy vẫn đạt ngưỡng.
- Head nhỏ/xa đủ 8 px, kể cả khi state hoặc relevance phải dùng `unknown`.

### Bắt buộc IGNORE

- Pedestrian signal và bicycle signal, kể cả biểu tượng đỏ/xanh nằm sát vehicle signal.
- Đèn pha, đèn hậu, đèn phanh, đèn xi-nhan và đèn cảnh báo trên phương tiện.
- Đèn đường, biển hiệu, biển báo phát sáng và thiết bị công nghiệp.
- Reflection trên kính, thân xe, mặt đường ướt; flare và quầng halo.
- Chấm màu không có đủ bằng chứng là vehicle signal.
- Candidate có tight visible height dưới 8 px.

Không tạo box rồi gán `unknown` cho object đã biết là ngoài scope. `unknown` chỉ dành cho attribute của một vehicle
signal đã được xác nhận hoặc candidate đủ bằng chứng theo rule.

## 6. Visibility / occlusion

| Tình huống | `occluded` | Cách xử lý |
|---|---:|---|
| Xe, cây, cột hoặc object khác che một phần housing | `true` | Box phần housing nhìn thấy; không đoán phần bị che. |
| Housing bị cắt bởi mép ảnh | `false` | Box phần trong ảnh; crop không phải occlusion. |
| Mưa, giọt nước trên kính, blur chuyển động | `false` | Dùng `unknown` cho attribute không đọc được và bật `needs_review`. |
| Bloom, glare, ngược sáng, tương phản thấp | `false` | Box theo housing, không theo halo; dùng `unknown` khi cần. |
| Nền tối ban đêm nhưng housing vẫn thấy | theo vật thể che | Label bình thường; bóng sáng không làm box phình ra. |
| Head quay lưng | `false` nếu không bị che | `state=unknown`, `relevance=not_relevant`, thường `pictogram=unknown`. |

Occlusion mô tả **vật thể che hình học**, không mô tả độ khó nhìn. Nếu housing vừa bị che vừa bị blur, đặt
`occluded=true` và đồng thời dùng `unknown + needs_review` cho attribute thiếu bằng chứng.

Với ảnh đêm, màu đúng chưa đủ chứng minh class. Chỉ label khi có housing/mặt tín hiệu hoặc bố trí cấu trúc đủ mạnh để
xác nhận đó là vehicle signal; bỏ đèn xe và các điểm sáng rời rạc.

## 7. Ambiguity / escalation

### Decision flow bắt buộc

1. **Có phải vehicle signal không?**
   - Chắc chắn ngoài scope → IGNORE.
   - Có đủ bằng chứng là vehicle signal → sang bước 2.
   - Chỉ là chấm màu/cấu trúc chung chung → IGNORE.
2. **Tight visible height có đạt 8 px không?**
   - Không đạt → IGNORE.
   - Đạt → LABEL.
3. **Vẽ geometry** theo visible housing; xác định `occluded`.
4. **Gán từng attribute độc lập.** Giá trị nào nhìn rõ phải dùng giá trị cụ thể; chỉ giá trị thiếu bằng chứng mới là
   `unknown`.
5. **Có attribute nào là `unknown` hoặc biên object đáng ngờ?**
   - Có → `needs_review=true`.
   - Không → `needs_review=false`.
6. **Có xác định được head nào điều khiển ego lane không?**
   - Có → không thêm image tag, kể cả ảnh có mixed colors hoặc object khác cần review.
   - Không, trong khi ảnh có giao lộ/tín hiệu có khả năng điều khiển ego → thêm tag `image_escalate`.

### Cách thể hiện bốn quyết định trong export

| Decision | Thể hiện trong CVAT |
|---|---|
| LABEL | Có rectangle `traffic_light`, geometry đúng và gán đủ năm attribute. |
| IGNORE | Không có box trên object ngoài scope/dưới ngưỡng. |
| UNKNOWN | Vẫn có box; đúng attribute thiếu bằng chứng mang giá trị `unknown`; `needs_review=true`. |
| ESCALATE cấp object | `needs_review=true` trên box cần QA xem. |
| ESCALATE cấp ảnh | Thêm tag `image_escalate` trên ảnh. Đây là tag, không phải attribute text. |

Không thêm `image_escalate` chỉ vì:

- ảnh có cả đèn đỏ và xanh ở các movement khác nhau;
- một head nhỏ có `relevance=unknown` nhưng head điều khiển ego vẫn xác định được;
- ảnh tối/mưa nhưng semantic ego signal vẫn rõ.

QA owner xử lý mọi box `needs_review=true` và mọi ảnh có `image_escalate`. Annotator không được thay `unknown` bằng
giá trị đoán để tránh review.

## 8. Temporal rule

**Không áp dụng track — task dùng ảnh tĩnh.**

- Annotate mỗi ảnh/frame độc lập bằng Shape.
- Chỉ dùng pixel trong ảnh hiện tại để gán state.
- Không copy state từ frame trước, không dùng frame sau để “sửa” frame hiện tại.
- Không suy một frame ở biên chuyển pha là yellow/unknown chỉ vì frame kế tiếp đổi màu.
- Vị trí housing giống nhau qua nhiều frame không có nghĩa state bất biến.
- Không suy đèn nhấp nháy từ một ảnh đơn; gán theo bằng chứng nhìn thấy trong ảnh đó.

## 9. Examples

Chỉ các sample thuộc split `example` hoặc `calibration` được dùng trong guide.

| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| `BDD21` | Hai vehicle signal xanh ban ngày, housing rõ | Hai box riêng; `state=green`, `relevance=relevant`, `pictogram=circle`, `occluded=false`, `needs_review=false`; không gồm cần treo | Normal baseline; một housing = một instance |
| `BDD02` | Nhiều head xanh ở trái/giữa/phải và ở các độ sâu khác nhau | Label mọi housing đủ 8 px; head theo ego lane là relevant, head nhánh khác là not relevant; attribute của head xa không rõ dùng unknown + review | Multi-head; không suy relevance từ màu/vị trí |
| `BDD25` | Chạng vạng, nhiều đèn xanh và reflection trên mặt đường ướt | Box từng housing thật; ignore vệt xanh trên mặt đường và đèn hậu; head xa không rõ lane dùng `relevance=unknown`, `needs_review=true` | Low visibility; reflection là negative |
| `BDD05` | Cấu trúc giống head quay lưng ở xa gần khu công nghiệp | Chỉ label cấu trúc xác nhận là vehicle signal và đủ 8 px; `state=unknown`, `relevance=not_relevant`, `pictogram=unknown`, `needs_review=true`; ignore thiết bị công nghiệp | Back-facing và class ambiguity |
| `BDD15` | Candidate signal rất nhỏ ở cuối phối cảnh | Label head xác nhận được và đủ 8 px; giữ state nếu đọc được, còn relevance/pictogram thiếu bằng chứng dùng unknown + review; dưới 8 px ignore | Small/far và threshold |
| `BDD17` | Mưa, kính ướt, có xanh và đỏ ở các hướng khác nhau | Box riêng từng housing; head theo ego giữ green + relevant; head movement khác red + not relevant/unknown theo bằng chứng; không gán occluded chỉ vì mưa | Weather artifact; mixed state |
| `BDD24` | Tuyết, tín hiệu xa và bị xe che một phần | Box visible-only; `occluded=true` khi xe che; state/relevance/pictogram không chắc dùng unknown + review; ignore đèn hậu | Occlusion khác low visibility |
| `LISA16` | Hai head tròn chuyển xanh, mũi tên trái vẫn đỏ | Head mũi tên: `red + not_relevant + arrow_left`; hai head tròn: `green + relevant + circle`; ba box riêng; không image-escalate | Mixed movement; state độc lập từng head/frame |

## 10. Common mistakes

| Sai thường gặp | Cách làm đúng |
|---|---|
| Một box dài chứa nhiều đầu đèn | Mỗi housing vật lý một tight box. |
| Box chỉ ôm bóng sáng hoặc mở rộng theo halo | Box theo phần housing/mặt tín hiệu nhìn thấy; loại halo. |
| Box gồm cột, cần treo hoặc tấm nền lớn | Chỉ bao housing và chụp che sáng gắn với head. |
| Vẽ amodal xuyên qua xe/cây | Chỉ box phần nhìn thấy; `occluded=true`. |
| Tick occluded cho mưa, blur, glare hoặc crop | Chỉ tick khi vật thể thật che; dùng unknown/review cho visibility kém. |
| Dùng `off` cho head quay lưng hoặc không đọc được | Dùng `unknown`; `off` chỉ khi thấy rõ mặt trước và xác nhận tất cả bóng tắt. |
| Gán mọi head phía trước là relevant | Dựa vào hướng mặt, lane/movement, vạch đường và cấu trúc giao lộ. |
| Dùng màu để suy relevance | State và relevance là hai quyết định độc lập. |
| Gán direction theo hướng đường | Chỉ gán `pictogram` từ hình nhìn thấy trên đèn. |
| Gộp đỏ và xanh của các movement thành `state=unknown` | Box và gán state riêng từng housing. Mixed colors không tự động là conflict. |
| Label pedestrian signal, đèn hậu hoặc reflection | IGNORE; kiểm housing và loại nguồn sáng ngoài scope. |
| Bỏ head not relevant hoặc quay lưng | Vẫn label nếu là vehicle signal đủ 8 px; gán relevance/state theo rule. |
| Để `__undefined__` | Gán đủ `state`, `relevance`, `pictogram`; giữ checkbox đúng trước khi submit. |
| Gán `unknown` nhưng quên `needs_review` | Mọi object có attribute unknown phải bật `needs_review=true`. |
| Thêm `image_escalate` cho mọi ảnh khó | Chỉ dùng khi không xác định được tín hiệu điều khiển ego trên toàn ảnh. |
| Copy state giữa các frame LISA | Annotate từng ảnh độc lập từ pixel hiện tại. |

### Checklist trước khi submit một ảnh

- [ ] Đã quét toàn ảnh ở kích thước gốc, gồm head nhỏ/xa và sát mép.
- [ ] Mọi vehicle signal đủ 8 px có một box riêng; object ngoài scope không có box.
- [ ] Box ôm phần housing nhìn thấy, không chứa cột/cần/halo và không amodal.
- [ ] `occluded` phản ánh vật thể che, không phản ánh blur/crop/glare.
- [ ] Mọi box có `state`, `relevance`, `pictogram` khác `__undefined__`.
- [ ] Mọi attribute `unknown` đi kèm `needs_review=true`.
- [ ] Không dùng màu hoặc vị trí đơn lẻ để đoán relevance/direction.
- [ ] `image_escalate` chỉ xuất hiện khi ego signal không thể xác định ở cấp ảnh.
