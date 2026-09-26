# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | rectangle | class | — | — | — | Một đầu đèn tín hiệu cho xe cơ giới (giao lộ, vạch sang đường giữa đoạn, ramp meter, đèn công trường) thấy được mặt đèn. Downstream cần detect từng đầu đèn; rectangle đủ cho detector, rẻ hơn polygon và dễ đo tolerance |
| `state` | — | attribute của `traffic_light` (select) | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | false | Tín hiệu STOP/GO cho planning. `off` tách khỏi `unknown` vì "thấy rõ đèn tắt" khác "không đọc được". Không tách mũi tên vì downstream chỉ cần màu + relevance |
| `relevance` | — | attribute của `traffic_light` (select) | `__undefined__`, `relevant`, `not_relevant`, `unknown` | `__undefined__` | false | Downstream phải biết đèn nào điều khiển ego. Là thuộc tính của cùng một object và phụ thuộc ngữ cảnh ảnh, không phải loại vật thể khác → attribute |
| `pictogram` | — | attribute của `traffic_light` (select) | `__undefined__`, `circle`, `arrow_left`, `arrow_right`, `arrow_straight`, `other`, `unknown` | `__undefined__` | false | Hình của ô đang sáng (`other` = quay đầu, mũi tên chéo, thanh; `unknown` khi đèn tắt/không rõ). Là bằng chứng cho `relevance`: QA bắt được tổ hợp mâu thuẫn như `arrow_left` + `relevant` khi ego đi thẳng — đúng loại lỗi critical |
| `needs_review` | — | attribute của `traffic_light` (checkbox) | `false` / `true` | `false` | false | ESCALATE cấp object; QA lọc nhanh các object annotator không chắc |
| `image_escalate` | tag (cả ảnh) | class kiểu tag | — | — | — | ESCALATE cấp ảnh khi không gán được relevance/state cho giao lộ gần nhất |
| `reason` | — | attribute của `image_escalate` (select) | `__undefined__`, `low_visibility`, `conflicting_lights`, `lane_unclear`, `other` | `__undefined__` | false | QA thống kê vì sao ảnh bị escalate → biết cần sửa guideline hay loại ảnh |

IGNORE không có label riêng: thể hiện bằng **không vẽ box** (guideline mục 5 liệt kê những gì không vẽ).
`mutable = false` cho mọi attribute vì task là ảnh tĩnh, dùng Shape, không có track.

## Class hay attribute

- **Một class `traffic_light`**, không tách `red_light`/`green_light` hay `relevant_light`: state và relevance là
  thuộc tính của **cùng một vật thể**, đổi theo thời gian/ngữ cảnh; tách class sẽ ra 5 × 3 = 15 class và detector
  phải học 15 lớp trông gần như giống nhau.
- **Không label đèn đi bộ/xe đạp/buýt/tàu điện, đèn điều khiển làn (X đỏ), đèn chắn tàu, đèn quay ngang** thành
  class riêng: downstream (module STOP/GO cho xe cơ giới) không dùng, thêm class chỉ tăng công label và nguy cơ nhầm.
  Chúng là IGNORE có bảng nhận biết ở guideline mục 5.
- **Có `pictogram` nhưng chỉ cho đèn xe**: không có giá trị `pedestrian`/`bicycle` vì đèn đi bộ/xe đạp là IGNORE
  (không vẽ). Nếu label chúng thành `traffic_light`, model học chúng là đèn xe → false STOP.
- **Không có `red_yellow`**: dữ liệu là đèn Mỹ (BDD/LISA), không có pha đỏ+vàng; trường hợp hai ô cùng sáng xử lý
  bằng rule "chọn màu hạn chế hơn + `needs_review`" (guideline mục 4).
- **Không có checkbox `occluded`**: downstream không dùng; box đã là visible-only, còn che mất ô sáng thì đã thể hiện
  bằng `state = unknown`. Thêm checkbox chỉ tăng thao tác mà không đổi quyết định nào.
- **Default `__undefined__` cho `state`, `relevance`, `pictogram`**: nếu default là giá trị đầu danh sách (`red`/`relevant`/`circle`), annotator quên gán sẽ
  tạo ra đèn xanh relevant "im lặng" — đúng loại lỗi critical nhất. `__undefined__` còn trong export = lỗi thấy được.
- **Default `false` cho `needs_review`**: bias về phía "không escalate" là chấp nhận được vì mọi giá trị `unknown` đều
  bắt buộc đi kèm tick (guideline mục 7) và QA kiểm tổ hợp `unknown` + `needs_review = false`.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): 2.74.1 (http://localhost:8080)
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): `team01-calib-v1-<tên người label>` (mỗi người một task)
- **Guide của task đã dán `02_guideline.md`?** TODO (có / chưa)
- **Nhóm dùng Track hay Shape, vì sao:** **Shape**. Mọi ảnh gán như ảnh tĩnh; LISA chỉ dùng 4 frame cách xa nhau, không
  cần nội suy. Export dùng **CVAT for images 1.1**, khớp với `make calib` và `make score`.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

TODO — làm sau khi tạo task calibration.
