# Edge-case library — LISA traffic light

Các card dưới đây được rút ra sau khi xem ở kích thước gốc toàn bộ 30 frame `LISA01`–`LISA30` của `dayClip5`
(frame 1606–1635). Trong schema hiện tại, **direction** được lưu bằng attribute `pictogram`:
`circle`, `arrow_left`, `arrow_right`, `arrow_straight`, `unknown`.

Chuỗi có hai pha quan sát được:

- `LISA01`–`LISA15`: hai đầu đèn tròn phía trên ở trạng thái `red`; đầu đèn mũi tên trái cũng `red`.
- `LISA16`–`LISA30`: hai đầu đèn tròn phía trên chuyển sang `green`; đầu đèn mũi tên trái vẫn `red`.

LISA là một clip liên tiếp nên các card này dùng cho **example/calibration**, không dùng làm blind. Gold decisions chỉ
được tạo cho năm ảnh BDD100K thuộc split `blind` trong `sample_pack.csv`.

Quy ước vị trí dùng trong card:

- **A — mũi tên trái phía trên:** housing bên trái trên cần ngang.
- **B — đèn tròn giữa phía trên:** housing lớn gần giữa ảnh.
- **C — đèn tròn phải phía trên:** housing lớn sát vùng cây tối bên phải.
- **D — cụm nhỏ phía xa:** các housing nhỏ gần tòa nhà/bên phải đường.

---

CASE ID: LISA-EC-01-MULTI-HEAD
Sample: LISA01
Scene: Giao lộ ngược sáng có nhiều đầu đèn cùng xuất hiện trên cần ngang và bên phải đường.
Observation: Ba housing A, B, C nhìn thấy rõ. A hiển thị mũi tên trái đỏ; B và C hiển thị đèn tròn đỏ.
Decision: LABEL
Expected: Vẽ ba box `traffic_light` độc lập, mỗi box ôm một housing nhìn thấy; không gộp cả cần ngang thành một box. A: `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`. B và C: `state=red`, `relevance=relevant`, `pictogram=circle`. Cả ba `occluded=false`, `needs_review=false`.
Rationale: Mỗi đầu đèn có state/direction/relevance riêng. Gộp housing làm mất tín hiệu áp dụng cho ego lane và gây lỗi cho detector.
Common mistake: Vẽ một box dài chứa A+B+C hoặc chỉ vẽ đầu đèn sáng nhất.
Diversity: multi_instance / geometry / state / relevance / direction

---

CASE ID: LISA-EC-02-LEFT-ARROW
Sample: LISA01
Scene: Đầu đèn A là mũi tên trái đỏ nằm cạnh hai đầu đèn tròn đỏ.
Observation: Hình mũi tên trái đọc được rõ; camera/ego ở làn đi thẳng, không ở làn rẽ trái được đầu đèn A điều khiển.
Decision: LABEL
Expected: A: `traffic_light`, `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`, `occluded=false`, `needs_review=false`; box chỉ ôm housing A, không gồm biển cấm quay đầu và cần treo.
Rationale: State đỏ không tự động làm tín hiệu relevant. Relevance phụ thuộc làn ego; direction phải được giữ riêng để planner không áp lệnh rẽ trái cho chuyển động đi thẳng.
Common mistake: Gán `pictogram=circle`, hoặc gán `relevance=relevant` chỉ vì đèn nằm phía trước camera.
Diversity: ambiguous_semantics / relevance / direction / geometry

---

CASE ID: LISA-EC-03-DUPLICATE-RELEVANT-RED
Sample: LISA01
Scene: Hai đầu đèn tròn B và C cùng báo đỏ cho hướng đi thẳng của ego.
Observation: B và C là hai instance vật lý riêng nhưng cùng state và cùng relevance.
Decision: LABEL
Expected: Vẽ riêng B và C. Cả hai: `state=red`, `relevance=relevant`, `pictogram=circle`, `occluded=false`, `needs_review=false`.
Rationale: Bỏ sót một đầu đèn relevant làm giảm recall; gán nhầm đỏ relevant thành `not_relevant` có thể khiến planner không dừng.
Common mistake: Chỉ giữ một đầu đèn vì cho rằng C là bản sao không cần label, hoặc gán C `not_relevant` vì nằm lệch phải.
Diversity: duplicate_signal / critical / relevance / recall

---

CASE ID: LISA-EC-04-SMALL-FAR-CANDIDATE
Sample: LISA01
Scene: Cụm D gần tòa nhà có các housing rất nhỏ, nền tối và chi tiết direction không đủ rõ.
Observation: Có ánh sáng đỏ gắn với cấu trúc giống đầu đèn xe; housing nhìn thấy cao từ 8 px trở lên nhưng quan hệ với ego lane không chắc chắn.
Decision: ESCALATE
Expected: Với từng housing D xác nhận được là đèn giao thông xe và cao ≥ 8 px, vẽ box phần housing nhìn thấy; gán `state=red` nếu bóng đỏ phân biệt được, `relevance=unknown`, `pictogram=unknown`, `needs_review=true`. Không gắn `image_escalate` vì B/C vẫn xác định được là tín hiệu relevant cho ego.
Rationale: Rule bắt buộc label vehicle signal đủ ngưỡng kích thước, nhưng không được suy đoán direction hoặc relevance từ vị trí xa/mờ.
Common mistake: Bỏ toàn bộ D vì nhỏ, hoặc tự đoán `relevance=not_relevant` và `pictogram=circle`.
Diversity: small_far / ambiguity / unknown / escalation

---

CASE ID: LISA-EC-05-NON-SIGNAL-LIGHTS
Sample: LISA01
Scene: Nhiều đèn pha xe đối diện, đèn hậu và điểm phản sáng xuất hiện trên mặt đường/tán cây.
Observation: Các điểm sáng không có housing traffic-light ổn định; một số sáng và tròn giống bóng đèn tín hiệu khi nhìn nhanh.
Decision: IGNORE
Expected: Không vẽ `traffic_light` cho đèn pha, đèn hậu, phản chiếu hoặc flare. Chỉ label khi thấy đủ bằng chứng là đầu đèn giao thông dành cho xe.
Rationale: False positive từ nguồn sáng giao thông khác làm hỏng detector và có thể tạo state giả cho planner.
Common mistake: Vẽ box quanh đèn pha xe đối diện hoặc đốm đỏ xa chỉ dựa vào màu.
Diversity: negative / conflicting_road_elements / reflection

---

CASE ID: LISA-EC-06-BACKLIGHT-HOUSING
Sample: LISA08
Scene: Bầu trời sáng phía sau làm housing tối, trong khi bóng đỏ bị bloom.
Observation: Biên housing B/C vẫn nhìn thấy; quầng sáng vượt ra ngoài vỏ đèn không phải là một phần geometry.
Decision: LABEL
Expected: B và C: `state=red`, `relevance=relevant`, `pictogram=circle`; box ôm vỏ đèn nhìn thấy, không ôm quầng đỏ, bầu trời hoặc tay cần. `occluded=false` vì tương phản thấp/bloom không phải che khuất.
Rationale: Geometry phải ổn định theo vỏ đèn, không thay đổi theo độ lớn của quầng sáng.
Common mistake: Box chỉ ôm bóng sáng, box phình theo halo, hoặc tick `occluded=true` vì housing tối.
Diversity: low_visibility / glare / geometry / occlusion_boundary

---

CASE ID: LISA-EC-07-LAST-RED-FRAME
Sample: LISA15
Scene: Frame cuối ngay trước khi hai đèn tròn đổi sang xanh.
Observation: Trong chính LISA15, B và C vẫn đỏ; không có bằng chứng màu vàng và không được dùng frame kế tiếp để sửa state của ảnh hiện tại.
Decision: LABEL
Expected: B và C: `state=red`, `relevance=relevant`, `pictogram=circle`. A: `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`. Không gán `yellow` hoặc `unknown` chỉ vì đây là biên chuyển pha.
Rationale: Task là ảnh tĩnh; state phải theo pixel của frame hiện tại. Suy diễn theo thứ tự thời gian tạo nhãn sai ở boundary.
Common mistake: Gán LISA15 là `yellow`, `green` hoặc `unknown` vì biết LISA16 đã xanh.
Diversity: temporal_boundary / state / static_frame_rule

---

CASE ID: LISA-EC-08-FIRST-GREEN-FRAME
Sample: LISA16
Scene: Frame đầu mà hai đầu đèn tròn B/C chuyển từ đỏ sang xanh.
Observation: B/C có bóng xanh rõ; A vẫn là mũi tên trái đỏ.
Decision: LABEL
Expected: B và C: `state=green`, `relevance=relevant`, `pictogram=circle`. A: `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`. Vẽ ba box riêng, không sao chép state từ LISA15.
Rationale: State là attribute theo từng head và từng frame. Việc giữ nhãn đỏ cũ làm sai trực tiếp input của planner.
Common mistake: Copy/paste annotation từ frame trước rồi quên đổi B/C sang `green`.
Diversity: temporal_transition / state_change / critical

---

CASE ID: LISA-EC-09-MIXED-STATE-DIRECTION
Sample: LISA16
Scene: Cùng một vùng giao lộ có mũi tên trái đỏ và đèn tròn xanh đồng thời.
Observation: Tín hiệu không mâu thuẫn: chúng điều khiển các movement/lane khác nhau.
Decision: LABEL
Expected: A giữ `red + not_relevant + arrow_left`; B/C là `green + relevant + circle`. Không gán `state=unknown`, không gắn `needs_review`, và không gắn `image_escalate` chỉ vì màu khác nhau.
Rationale: Gộp state theo toàn giao lộ có thể biến đèn đỏ rẽ trái thành xanh hoặc biến đèn xanh ego thành đỏ. Đây là lỗi state/relevance có hậu quả downstream lớn.
Common mistake: Chọn một màu chung cho cả cụm, hoặc coi mixed color là ảnh lỗi cần escalate.
Diversity: conflict / direction / relevance / critical

---

CASE ID: LISA-EC-10-SMALL-FAR-MIXED-COLOR
Sample: LISA16
Scene: Cụm D nhỏ phía xa xuất hiện điểm xanh cạnh điểm đỏ trong vùng nền tối.
Observation: State màu có thể phân biệt ở một số housing, nhưng direction và quan hệ lane không đủ chắc chắn ở kích thước này.
Decision: UNKNOWN
Expected: Box riêng từng housing D đủ ngưỡng ≥ 8 px và xác nhận là vehicle signal. Gán `state=green` hoặc `state=red` theo bóng đang sáng của từng housing; gán `relevance=unknown`, `pictogram=unknown`, `needs_review=true`. Không gộp hai màu vào một box/housing.
Rationale: Không được dùng màu của head lân cận để suy ra direction/relevance. Object-level review giữ lại bằng chứng mà không chặn toàn ảnh.
Common mistake: Gộp điểm xanh và đỏ thành một đầu đèn `unknown`, hoặc bỏ qua vì khó xác định relevance.
Diversity: small_far / mixed_state / ambiguity / escalation

---

CASE ID: LISA-EC-11-GREEN-BLOOM-GEOMETRY
Sample: LISA24
Scene: Hai bóng xanh B/C sáng mạnh trên nền cây và tòa nhà tối.
Observation: Quầng cyan lớn hơn bóng thật; vỏ đèn vẫn là giới hạn instance.
Decision: LABEL
Expected: B/C: `state=green`, `relevance=relevant`, `pictogram=circle`; bbox ôm housing, không ôm halo. A: `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`. Tất cả `occluded=false` nếu biên housing còn nhìn thấy.
Rationale: Box theo halo thay đổi theo exposure và làm geometry không nhất quán giữa pha đỏ/xanh.
Common mistake: Vẽ box chỉ quanh đốm xanh hoặc mở rộng box đến toàn quầng sáng.
Diversity: glare / geometry / state / consistency

---

CASE ID: LISA-EC-12-DO-NOT-CARRY-STATE
Sample: LISA30
Scene: Cuối đoạn clip, bố cục gần giống các frame đầu nhưng phase đã khác.
Observation: B/C vẫn xanh; A vẫn là mũi tên trái đỏ. Vị trí gần như không đổi khiến thao tác copy dễ giữ nhầm attribute.
Decision: LABEL
Expected: A: `red + not_relevant + arrow_left`. B/C: `green + relevant + circle`. Vì task là ảnh tĩnh, annotate từng ảnh độc lập; không tạo track và không giả định state bất biến theo identity.
Rationale: Hình học ổn định không đồng nghĩa state ổn định. Sai state trên relevant head là lỗi critical theo downstream contract.
Common mistake: Copy nguyên annotation của LISA01 sang LISA30, giữ B/C là `red`.
Diversity: temporal_consistency / state / critical / execution_risk

---

CASE ID: LISA-EC-13-SUBTHRESHOLD-DOTS
Sample: LISA30
Scene: Cuối đường có nhiều chấm đỏ/cam rất nhỏ lẫn với đèn xe và biển hiệu.
Observation: Không thấy housing vehicle signal đủ bằng chứng hoặc chiều cao object dưới 8 px.
Decision: IGNORE
Expected: Không tạo box cho chấm sáng dưới ngưỡng 8 px hoặc không xác nhận được là đèn giao thông xe. Không dùng `unknown` để label mọi đốm màu. Nếu một candidate đạt ≥ 8 px và có housing rõ thì xử lý theo LISA-EC-04/LISA-EC-10.
Rationale: Ngưỡng kích thước giữ consistency và tránh false positive từ đèn xe/biển hiệu, trong khi vẫn có rule riêng cho signal nhỏ nhưng đủ ngưỡng.
Common mistake: Vẽ hàng loạt box lên mọi chấm đỏ/xanh ở xa, hoặc ngược lại bỏ cả housing đủ 8 px.
Diversity: small_far / threshold / negative / inclusion_exclusion

---

# Edge-case library — BDD100K traffic light

Đã mở ở kích thước gốc toàn bộ `BDD01`–`BDD26`. Mười hai ảnh có tín hiệu giao thông xe hoặc candidate đủ để
kiểm thử rule là: `BDD02`, `BDD05`, `BDD07`, `BDD12`, `BDD15`, `BDD17`, `BDD18`, `BDD20`, `BDD21`, `BDD24`,
`BDD25`, `BDD26`. Các ảnh còn lại không đưa vào pack traffic-light vì không có vehicle signal đủ điều kiện; riêng
pedestrian/bicycle signal không được tính là traffic light trong scope này.

---

CASE ID: BDD-EC-01-MULTI-GREEN
Sample: BDD02
Scene: Giao lộ nội đô ban ngày có nhiều đầu đèn xanh ở trái, giữa, phải và xa dần theo trục đường.
Observation: Nhiều housing vật lý cùng màu nhưng ở các nhánh/làn khác nhau; một housing sát mép ảnh.
Decision: LABEL
Expected: Vẽ box riêng cho từng vehicle-signal housing cao ≥ 8 px. Head điều khiển làn ego: `state=green`, `relevance=relevant`, `pictogram=circle`; head của nhánh khác: `relevance=not_relevant`. Nếu pictogram/relevance của head xa không đọc được, dùng `unknown` và `needs_review=true`.
Rationale: Cùng màu xanh không đồng nghĩa cùng relevance; mỗi housing phải giữ state và relation riêng.
Common mistake: Gộp nhiều đèn vào một box, cắt mất housing sát mép, hoặc gán tất cả `relevance=relevant`.
Diversity: multi_instance / edge_crop / state / relevance / direction

---

CASE ID: BDD-EC-02-BACK-FACING-FAR
Sample: BDD05
Scene: Đường cao tốc cạnh khu công nghiệp có các cấu trúc giống đầu đèn rất xa và quay lệch/quay lưng.
Observation: Hình dạng housing có thể nhận ra nhưng bóng sáng, direction và quan hệ với ego lane không chắc chắn.
Decision: UNKNOWN
Expected: Chỉ label candidate xác nhận là vehicle signal và cao ≥ 8 px. Nếu thấy mặt sau/housing nhưng không thấy bóng sáng: `state=unknown`, `relevance=not_relevant`, `pictogram=unknown`, `needs_review=true`; không label thiết bị công nghiệp chỉ vì có ba khoang dọc.
Rationale: Scope có head quay lưng, nhưng không được biến cấu trúc công nghiệp tương tự thành false positive.
Common mistake: Bỏ mọi head quay lưng, hoặc gán `off` cho object không đủ bằng chứng là traffic light.
Diversity: small_far / back_facing / negative / ambiguity

---

CASE ID: BDD-EC-03-CROSS-LANE-RELEVANCE
Sample: BDD07
Scene: Giao lộ ban ngày có hai đầu đèn xanh rõ ở hai bên ảnh.
Observation: Cả hai là vehicle signal, nhưng đèn trái điều khiển movement/làn khác với ego lane.
Decision: LABEL
Expected: Vẽ `traffic_light` x2. Cả hai `state=green`, `pictogram=circle`; đèn phải/đầu đèn theo hướng ego là `relevance=relevant`, đèn trái là `relevance=not_relevant`. Box riêng từng housing.
Rationale: Relevance là quan hệ với ego lane, không suy trực tiếp từ màu hoặc việc đèn nằm phía trước camera.
Common mistake: Gán cả hai relevant, hoặc bỏ đèn trái dù scope vẫn yêu cầu label head không relevant.
Diversity: normal / critical / relevance / multi_instance

---

CASE ID: BDD-EC-04-OCCLUDED-INTERSECTION
Sample: BDD12
Scene: Xe van phía trước che phần lớn vùng giao lộ; chỉ còn các candidate/tín hiệu phụ rất khó liên hệ với lane.
Observation: Không đủ bằng chứng ảnh tĩnh để xác định chắc candidate vehicle signal nào điều khiển ego lane; pedestrian signal không thuộc scope.
Decision: ESCALATE
Expected: Gắn `image_escalate=true`. Không tự đoán state/relevance/direction cho vùng bị xe che; bỏ qua pedestrian signal. Nếu một vehicle housing riêng lẻ vẫn xác nhận được và cao ≥ 8 px, box phần nhìn thấy với `occluded=true`, các attribute không chắc dùng `unknown` và `needs_review=true`.
Rationale: Đây là image-level ambiguity: uncertainty ảnh hưởng trực tiếp quyết định tín hiệu nào áp dụng cho ego.
Common mistake: Dùng vị trí tương đối để đoán relevance, hoặc label pedestrian signal như vehicle signal.
Diversity: occlusion / ambiguity / escalation / critical

---

CASE ID: BDD-EC-05-DISTANT-DAYLIGHT
Sample: BDD15
Scene: Đường nội đô rộng ban ngày; các đầu đèn nằm rất xa gần cuối phối cảnh.
Observation: Housing nhỏ, một phần lẫn với cột/biển và phương tiện; màu có thể thấy nhưng pictogram không đủ rõ.
Decision: UNKNOWN
Expected: Label mọi vehicle housing cao ≥ 8 px; gán state chỉ khi bóng sáng phân biệt được. `relevance=unknown`, `pictogram=unknown`, `needs_review=true` cho các head không truy được movement. Bỏ candidate dưới 8 px.
Rationale: Giữ recall cho object đủ ngưỡng nhưng không bịa semantic từ vài pixel.
Common mistake: Bỏ toàn bộ vì xa, hoặc gán mặc định `circle + relevant`.
Diversity: small_far / threshold / ambiguity / direction

---

CASE ID: BDD-EC-06-RAIN-AND-MIXED-SIGNALS
Sample: BDD17
Scene: Phố nội đô trong mưa; giọt nước trên kính làm mờ biên, đồng thời có tín hiệu xanh và đỏ ở các hướng khác nhau.
Observation: Các bóng xanh hướng theo trục ego vẫn đọc được; điểm đỏ bên nhánh/ở xa không được dùng để đổi state của head xanh.
Decision: LABEL
Expected: Vẽ riêng từng housing đủ ngưỡng. Head hướng ego đọc được: `state=green`, `relevance=relevant`, `pictogram=circle`; head đỏ của movement khác: `state=red`, `relevance=not_relevant` hoặc `unknown` theo bằng chứng, không gộp state. Giọt nước không phải `occluded=true`.
Rationale: Weather artifact làm giảm visibility nhưng không phải vật thể che; mixed color thường là nhiều movement khác nhau.
Common mistake: Gán toàn ảnh một state, tick occluded cho blur do mưa, hoặc box giọt nước/phản chiếu.
Diversity: rain / low_visibility / conflict / state / relevance

---

CASE ID: BDD-EC-07-NIGHT-GREEN-VS-LIGHTS
Sample: BDD18
Scene: Phố đêm có hai đầu đèn xanh chính giữa, xen dày đặc đèn hậu, đèn đường và biển sáng.
Observation: Hai green vehicle-signal có housing; nhiều đốm đỏ/trắng khác không có hình học traffic-light.
Decision: LABEL
Expected: Hai head chính: `state=green`, `relevance=relevant`, `pictogram=circle`, box theo housing không theo halo. Không label đèn hậu/đèn pha/biển sáng/pedestrian signal. Head xa khác chỉ label nếu có housing và cao ≥ 8 px.
Rationale: Màu sáng đơn lẻ không đủ xác nhận class; housing là bằng chứng phân biệt quan trọng ở ban đêm.
Common mistake: Box quầng sáng xanh, label đèn hậu đỏ, hoặc bỏ housing vì nền tối.
Diversity: night / low_visibility / glare / negative / geometry

---

CASE ID: BDD-EC-08-TINY-MIXED-COLOR
Sample: BDD20
Scene: Đường dân cư có candidate signal rất xa, lẫn với cây, cột và các xe đỗ.
Observation: Có bằng chứng housing đủ ngưỡng nhưng state, pictogram và lane relation không thể đọc chắc; các điểm đỏ/xanh sát nhau dễ bị gộp nhầm.
Decision: ESCALATE
Expected: Candidate vehicle housing cao ≥ 8 px: `state=unknown`, `relevance=unknown`, `pictogram=unknown`, `needs_review=true`; gắn `image_escalate=true` vì không xác định được tín hiệu điều khiển ego lane. Không gộp các điểm màu khác housing.
Rationale: Unknown giữ lại object-level evidence; escalation báo semantic ambiguity có thể ảnh hưởng downstream.
Common mistake: Đoán state từ màu lân cận, bỏ object vì nhỏ, hoặc box cả cụm cây/cột.
Diversity: small_far / ambiguity / escalation / geometry

---

CASE ID: BDD-EC-09-CLEAR-DAY-GREEN
Sample: BDD21
Scene: Đường nhiều cây ban ngày có hai đầu đèn xanh nhìn rõ ở trước xe.
Observation: Housing và bóng xanh tròn rõ, không bị vật thể che; đây là baseline normal cho so sánh với ảnh khó.
Decision: LABEL
Expected: Vẽ hai instance riêng; `state=green`, `relevance=relevant`, `pictogram=circle`, `occluded=false`, `needs_review=false`. Box không gồm cần cong/cột treo.
Rationale: Case normal neo cách vẽ và attribute trước khi kiểm các case low-visibility.
Common mistake: Một box chứa cả head lẫn tay cần, hoặc chỉ label head gần hơn.
Diversity: normal / multi_instance / geometry / state

---

CASE ID: BDD-EC-10-SNOW-VEHICLE-OCCLUSION
Sample: BDD24
Scene: Phố có tuyết, xe dày và tín hiệu rất xa giữa nền tòa nhà sáng.
Observation: Các candidate nhỏ bị phương tiện che một phần, độ tương phản thấp; state/direction không đủ chắc ở một số head.
Decision: UNKNOWN
Expected: Head xác nhận là vehicle signal và cao ≥ 8 px được box phần housing nhìn thấy với `occluded=true`; state đọc được thì giữ màu, còn không thì `state=unknown`; `relevance=unknown`, `pictogram=unknown`, `needs_review=true`. Không box đèn hậu xe.
Rationale: Occlusion được xác định bởi vật thể che, khác với blur/low contrast do thời tiết.
Common mistake: Box xuyên qua xe để đoán toàn housing, hoặc dùng đèn hậu làm bằng chứng state đỏ.
Diversity: snow / occlusion / small_far / low_visibility

---

CASE ID: BDD-EC-11-DUSK-WET-REFLECTION
Sample: BDD25
Scene: Chạng vạng trên phố đông, nhiều đầu đèn xanh liên tiếp và mặt đường ướt phản chiếu màu.
Observation: Vehicle-signal housing thật nằm trên cột/cần; vệt xanh dưới mặt đường và đèn hậu đỏ không có housing.
Decision: LABEL
Expected: Label từng housing đủ ngưỡng. Head điều khiển ego đọc được: `state=green`, `relevance=relevant`, `pictogram=circle`; head giao lộ xa không rõ movement dùng `relevance=unknown`, `needs_review=true`. Bỏ toàn bộ reflection trên mặt đường.
Rationale: Phản chiếu mang màu đúng nhưng không phải object; nhiều giao lộ liên tiếp không được gộp relevance.
Common mistake: Vẽ box trên vệt xanh, label đèn hậu, hoặc gán mọi head xa relevant.
Diversity: dusk / wet_road / reflection / multi_depth / relevance

---

CASE ID: BDD-EC-12-NIGHT-PEDESTRIAN-CONFLICT
Sample: BDD26
Scene: Ban đêm có một vehicle signal xanh gần bên trái, pedestrian signal đỏ ngay cạnh và nhiều tín hiệu đỏ/xanh ở xa.
Observation: Màu đỏ gần không thuộc class vehicle; các head đỏ xa vẫn cần label nếu housing đạt ngưỡng.
Decision: LABEL
Expected: Head xanh gần: `state=green`, `relevance=relevant`, `pictogram=circle`. Pedestrian signal đỏ: IGNORE. Head xe đỏ xa đủ 8 px: `state=red`, `relevance=unknown`, `pictogram=unknown`, `needs_review=true`; box riêng từng housing, không gộp theo màu.
Rationale: Nhầm pedestrian red thành vehicle red làm sai state; bỏ sót vehicle red xa là lỗi critical theo downstream contract.
Common mistake: Gán head xanh gần `state=red` vì thấy biểu tượng đỏ kế bên, hoặc bỏ mọi head xa.
Diversity: night / critical / conflict / small_far / inclusion_exclusion

---
