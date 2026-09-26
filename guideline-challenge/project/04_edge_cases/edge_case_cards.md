# Edge-case library

**Scope:** Gold Owner — Traffic light (state, relevance, direction/pictogram).  
**Dataset used in this version:** `bdd100k` + `lisa` only.  
**Note:** `lisa` is a single 30-frame daytime clip and is for example/calibration, not blind; `bdd100k` supplies the blind candidates. The repository explicitly requires ≥8 cards, recommends 10–12, and asks for diversity across ambiguity, conflict, small/far, critical risk and escalation.

> **Ontology used by the cards:** class `traffic_light`; attributes `state`, `relevance`, `pictogram`, `occluded`.  
> `pictogram` is the direction-related field in the current CVAT schema: `circle`, `arrow_left`, `arrow_right`, `arrow_straight`, `pedestrian`, `bicycle`, `unknown`.

---

## CASE ID: TL-EC-01
Sample: BDD07
Scene: Giao lộ đô thị, nhiều đầu đèn, ban ngày.
Observation: Có nhiều đầu đèn trong cùng một giao lộ; đầu đèn bên trái không phải là đầu đèn áp dụng cho chuyển động của ego.
Decision: LABEL
Expected: `traffic_light` cho các đầu đèn xe nhìn thấy đủ; với đầu đèn bên trái, `relevance=not_relevant`. Box chỉ ôm phần vỏ đèn nhìn thấy.
Rationale: Relevance được xác định theo chuyển động/làn mà đầu đèn điều khiển, không theo khoảng cách hoặc vị trí gần camera. Gán đèn của chuyển động khác thành relevant có thể làm planner chọn sai tín hiệu.
Common mistake: Chọn đầu đèn gần nhất hoặc tất cả đèn cùng phía làm relevant.
Diversity: ambiguity / conflict / critical

---

## CASE ID: TL-EC-02
Sample: BDD12
Scene: Đường đô thị nhiều làn, nhiều phương tiện và giao cắt.
Observation: Có bằng chứng giao thông phức tạp nhưng không đủ để suy ra chắc chắn đầu đèn nào điều khiển ego lane.
Decision: ESCALATE
Expected: Không đoán `relevance`; giữ các attribute thiếu bằng chứng ở `unknown` nếu đã label được object, đồng thời yêu cầu escalation ở mức ảnh theo downstream contract (`image_escalate`).
Rationale: Project quy định khi không xác định được đèn nào điều khiển ego lane thì phải escalation thay vì suy đoán.
Common mistake: Dùng vị trí gần nhất, hướng camera hoặc màu đèn để tự suy ra relevance.
Diversity: ambiguity / escalation / critical

> **Implementation note:** file `03_cvat_labels.json` hiện tại chưa có `needs_review` hoặc tag `image_escalate`, trong khi `01_problem_statement.md` và README dùng hai cơ chế này để biểu diễn ESCALATE. Card này vì vậy là **schema-gap cần sửa trước `make freeze`**, không nên giả vờ rằng export hiện tại đã chấm được ESCALATE.

---

## CASE ID: TL-EC-03
Sample: LISA16
Scene: Giao lộ nhiều đầu đèn, cuối ngày, nhiều đầu đèn trên cùng một giàn.
Observation: Trong cùng một frame, đầu đèn có pictogram mũi tên trái vẫn đỏ trong khi đầu đèn tròn/đi thẳng đang xanh.
Decision: LABEL
Expected: Tạo một `traffic_light` instance cho từng đầu đèn xe nhìn thấy; gán `state` theo từng instance, không theo toàn giao lộ. Đầu có mũi tên trái: `pictogram=arrow_left`, `state=red`; các đầu đèn tròn đang sáng xanh: `pictogram=circle`, `state=green`, với `relevance` chỉ gán theo bằng chứng về ego movement.
Rationale: Một giao lộ có thể đồng thời có nhiều pha và nhiều movement. State là thuộc tính của từng head.
Common mistake: Gán một state chung cho toàn bộ cụm đèn.
Diversity: ambiguity / conflict / multi-head

---

## CASE ID: TL-EC-04
Sample: LISA15 → LISA16
Scene: Hai frame liên tiếp của cùng một clip LISA.
Observation: LISA15 còn đầu đèn tròn chính ở trạng thái đỏ; sang LISA16 đầu đèn tròn chính đã chuyển sang xanh, trong khi đầu đèn mũi tên trái vẫn đỏ.
Decision: LABEL
Expected: `state` được đọc từ frame hiện tại. LISA15: head tròn chính `state=red`; LISA16: head tròn chính `state=green`; head mũi tên trái vẫn `state=red`.
Rationale: `state` là attribute có thể thay đổi theo frame khi dùng track. Không được copy state từ frame trước/sau vào frame hiện tại.
Common mistake: Giữ nguyên state của frame trước hoặc gán green cho toàn bộ heads sau khi một head chuyển green.
Diversity: temporal / ambiguity / conflict

---

## CASE ID: TL-EC-05
Sample: LISA01
Scene: Giao lộ nhiều làn, nhiều đầu đèn trên giàn ngang.
Observation: Một đầu đèn hiển thị mũi tên trái màu đỏ nằm cạnh các đầu đèn hình tròn.
Decision: LABEL
Expected: Head có biểu tượng mũi tên trái: `pictogram=arrow_left`, `state=red`; head tròn: `pictogram=circle` và `state` theo màu đang sáng. Không suy `pictogram` từ vị trí trái/phải của head trong ảnh.
Rationale: Direction trong ontology hiện tại được biểu diễn bởi `pictogram`, không phải một field `direction` riêng.
Common mistake: Thấy head nằm bên trái thì gán `arrow_left`, hoặc thấy green thì mặc định `arrow_straight`.
Diversity: ambiguity / pictogram

---

## CASE ID: TL-EC-06
Sample: BDD18
Scene: Đường đô thị ban đêm.
Observation: Nhiều điểm sáng nhỏ cùng xuất hiện: đèn giao thông, đèn đường, đèn xe và nguồn sáng đô thị. Có tín hiệu xe ở xa và tín hiệu pedestrian ở phía phải.
Decision: LABEL / IGNORE theo từng object.
Expected: Chỉ vẽ `traffic_light` dành cho xe đạt ngưỡng nhìn thấy; đèn pedestrian/nguồn sáng không phải vehicle signal thì không vẽ. Với vehicle signal không đủ bằng chứng về màu hoặc relevance, dùng `state=unknown`/`relevance=unknown` thay vì đoán.
Rationale: Ban đêm làm tăng nguy cơ nhầm một điểm sáng thành traffic light hoặc nhầm pedestrian signal thành vehicle signal.
Common mistake: Label mọi điểm sáng màu đỏ/xanh; dùng halo làm geometry.
Diversity: low_visibility / ambiguity / conflict

---

## CASE ID: TL-EC-07
Sample: BDD25
Scene: Đường đô thị lúc chạng vạng, mặt đường ướt, nhiều đèn xanh và phản chiếu.
Observation: Ánh sáng traffic light phản chiếu trên mặt đường ướt tạo các vùng sáng màu xanh/đỏ kéo dài.
Decision: LABEL / IGNORE
Expected: Label đầu đèn xe thật nếu đạt scope; không tạo box cho vùng phản chiếu trên mặt đường, kính hoặc thân xe. State của head thật được lấy từ bóng/nguồn sáng của head, không lấy từ reflection.
Rationale: Reflection là nguồn false positive trực tiếp đối với detector; downstream cần số instance thật chứ không phải số vùng sáng.
Common mistake: Vẽ thêm traffic_light cho reflection hoặc kéo box xuống bao gồm cả reflection.
Diversity: conflict / low_visibility / false_positive

---

## CASE ID: TL-EC-08
Sample: BDD11
Scene: Đường đô thị ban ngày tại giao cắt nhỏ.
Observation: Ở phía phải ảnh có tín hiệu dành cho pedestrian, trong khi bài toán chỉ yêu cầu vehicle traffic signal.
Decision: IGNORE
Expected: Không vẽ `traffic_light` cho pedestrian signal; `pictogram=pedestrian` là dấu hiệu để loại khỏi scope nếu object đã được nhận diện.
Rationale: Problem statement loại pedestrian/bicycle signal khỏi scope để tránh trộn hai loại điều khiển.
Common mistake: Label mọi signal có màu đỏ/xanh như vehicle traffic light.
Diversity: ambiguous_semantics / exclusion

---

## CASE ID: TL-EC-09
Sample: BDD22
Scene: Highway lúc chạng vạng; tín hiệu ở rất xa trên đường chân trời.
Observation: Các tín hiệu ở xa rất nhỏ, khó xác định vỏ đèn và màu; object có thể dưới ngưỡng 8 px.
Decision: IGNORE nếu chiều cao object < 8 px; UNKNOWN chỉ khi object đạt scope nhưng attribute không đủ bằng chứng.
Expected: Không tạo `traffic_light` box cho object vehicle signal có chiều cao < 8 px; không dùng `unknown` để giữ một object vốn đã ngoài scope.
Rationale: Scope của project đặt ngưỡng ≥8 px; đây là ranh giới small/far quan trọng để tránh label nhiễu.
Common mistake: Label mọi chấm sáng ở horizon hoặc dùng `unknown` để né ngưỡng kích thước.
Diversity: small_far / low_visibility / ambiguity

---

## CASE ID: TL-EC-10
Sample: BDD21
Scene: Đường nhiều làn ban ngày, nhiều cây và vật thể nền.
Observation: Có đầu đèn xe nhìn thấy ở xa trong nền cây; background foliage làm biên vỏ đèn khó quan sát hơn so với cảnh nền sạch.
Decision: LABEL nếu object đạt ngưỡng; dùng `occluded=true` chỉ khi vật thể thực sự che một phần vỏ đèn.
Expected: Box chỉ ôm phần vỏ đèn nhìn thấy; nếu có che một phần bởi foliage/vật thể khác thì `occluded=true`; không coi background clutter tự động là occlusion.
Rationale: `occluded` phải phản ánh việc object bị vật khác che, không phải chỉ vì object nhỏ hoặc nền phức tạp.
Common mistake: Gán `occluded=true` chỉ vì background rối; hoặc box gồm cả cây/cột.
Diversity: small_far / occlusion / geometry

---

## CASE ID: TL-EC-11
Sample: BDD26
Scene: Đường đô thị ban đêm.
Observation: Một đèn xanh khá sáng tạo halo/bloom lớn trong vùng tối; xung quanh có nhiều nguồn sáng khác.
Decision: LABEL
Expected: Label đúng traffic-light housing/head; `state=green` khi lõi tín hiệu xanh đủ rõ; geometry ôm vỏ đèn nhìn thấy, không ôm toàn bộ halo.
Rationale: Halo không phải một instance mới. State dựa vào tín hiệu/nguồn sáng của head chứ không dựa vào kích thước vùng phát sáng.
Common mistake: Box quanh halo; tạo thêm object cho halo; nhầm green light với street light.
Diversity: low_visibility / ambiguity / geometry

---

## CASE ID: TL-EC-12
Sample: BDD17
Scene: Đường đô thị ban ngày trong mưa, kính xe và mặt đường ướt.
Observation: Contrast giảm, nhiều vùng phản sáng và nguồn sáng nhỏ ở xa; bằng chứng về traffic-light object có thể không đủ.
Decision: IGNORE hoặc UNKNOWN theo scope: chỉ LABEL khi thấy vehicle signal đạt ≥8 px; nếu đã có object đạt scope nhưng không xác định được attribute thì dùng `unknown`.
Expected: Không hallucinate `traffic_light` từ các điểm sáng/reflective highlight; object ngoài scope không vẽ.
Rationale: Low visibility không phải lý do tự động tạo object. Evidence threshold phải giữ nguyên trong điều kiện mưa.
Common mistake: “Thấy chấm sáng thì chắc là đèn giao thông”; dùng `unknown` cho object <8 px.
Diversity: low_visibility / ambiguity / negative