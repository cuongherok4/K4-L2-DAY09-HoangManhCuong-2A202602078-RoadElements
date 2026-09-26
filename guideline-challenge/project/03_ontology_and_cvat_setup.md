# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp với từng dòng bên dưới.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | Rectangle | Class | — | — | — | Đại diện cho đối tượng traffic light vật lý, cần được annotate bằng bounding box. |
| `state` | Attribute của `traffic_light` | Attribute | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | Có | Mô tả trạng thái hiện tại của traffic light. `__undefined__` tránh việc vô tình gán state khi annotator chưa đưa ra quyết định. |
| `relevance` | Attribute của `traffic_light` | Attribute | `__undefined__`, `relevant`, `not_relevant`, `unknown` | `__undefined__` | Có | Mô tả traffic light có liên quan đến hướng di chuyển của ego vehicle hay không. |
| `pictogram` | Attribute của `traffic_light` | Attribute | `__undefined__`, `circle`, `arrow_left`, `arrow_right`, `arrow_straight`, `unknown` | `__undefined__` | Không | Mô tả pictogram/loại tín hiệu nhìn thấy trên traffic light. |
| `occluded` | Attribute của `traffic_light` | Attribute | `false`, `true` | `false` | Có | Mô tả traffic light có bị che khuất hay không. Có thể thay đổi giữa các frame, nên được đặt là mutable. |
| `needs_review` | Attribute của `traffic_light` | Attribute | `false`, `true` | `false` | Không | QA flag cho biết annotation/object có cần review hay không. Đây là metadata của annotation nên không cần thay đổi theo thời gian. |
| `image_escalate` | Image-level tag | Tag | — | — | — | Dùng khi ambiguity ở cấp độ toàn ảnh không thể được giải quyết an toàn bằng các attribute hiện có. |

## Class hay attribute

- `traffic_light` là **class** vì đây là một đối tượng vật lý có geometry.
- `state` là **attribute** vì nó mô tả trạng thái của một traffic-light object và có thể thay đổi theo thời gian.
- `relevance` là **attribute** vì nó mô tả mối quan hệ giữa traffic light và ego vehicle.
- `pictogram` là **attribute** vì nó mô tả loại/pictogram hiển thị trên traffic light và thường không thay đổi trong quá trình track.
- `occluded` là **attribute** vì nó mô tả điều kiện visibility của traffic light và có thể thay đổi giữa các frame.
- `needs_review` là **attribute** vì đây là QA metadata gắn với annotation/object.
- `image_escalate` là **tag** vì escalation áp dụng cho toàn bộ image, không gắn với một geometry cụ thể.

### Default nào có thể gây bias?

Các default hiện tại nhìn chung tránh được bias đối với các semantic attributes chính:

- `state = __undefined__`: phù hợp; annotator phải đưa ra quyết định thay vì mặc định một state.
- `relevance = __undefined__`: phù hợp; tránh mặc định traffic light là relevant.
- `pictogram = __undefined__`: phù hợp; tránh mặc định loại signal.
- `occluded = false`: phù hợp nếu giả định mặc định là object không bị occluded và annotator sẽ bật `true` khi quan sát thấy occlusion.
- `needs_review = false`: phù hợp vì review flag chỉ được bật khi annotator xác định annotation cần review.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): 2.76.1
- **Tên task calibration**: TEAM01-CALIB-V1
- **Guide của task đã dán `02_guideline.md`?**: Có
- **Nhóm dùng Track hay Shape, vì sao:** **Shape**, vì các ảnh chọn cho calib v1 là các ảnh riêng biệt, không theo sequence.

## Setup test

Một thành viên **chưa tham gia setup** cần mở calibration task và trả lời:

- Label object nào?
- Dùng tool nào trong CVAT?
- Gán những attribute nào?
- Khi nào dùng `unknown`?
- Khi nào dùng `image_escalate`?
- Khi nào bật `occluded = true`?
- Khi nào bật `needs_review = true`?

**Tester:** TODO  
**Kết quả:** TODO  
**Label expected:** `traffic_light`  
**Tool expected:** Rectangle / Shape  
**Attributes expected:** `state`, `relevance`, `pictogram`, `occluded`, `needs_review` khi applicable  
**Escalation:** `image_escalate` khi ambiguity không thể được resolve một cách an toàn từ evidence hiện có  
**Tester vấp ở đâu:** TODO

## Confirmation required trước Gate G2

**Escalation policy:** Cần xác nhận khi nào dùng `image_escalate` thay vì gán `unknown` cho một attribute cụ thể.