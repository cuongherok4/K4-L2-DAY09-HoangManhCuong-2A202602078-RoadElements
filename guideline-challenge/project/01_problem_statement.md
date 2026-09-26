# Problem statement + downstream contract

## Bài toán

Gắn nhãn **đèn giao thông cho xe (vehicle signal)** tại giao lộ có nhiều đầu đèn, kể cả ban đêm và chạng vạng: mỗi đầu
đèn cần `state` đúng và biết đèn đó **có điều khiển làn của ego vehicle hay không** (`relevance`). Chỗ khó: nhiều đầu
đèn cùng lúc (đèn thẳng + đèn mũi tên rẽ), đèn nhỏ/xa, bị che, và ban đêm chỉ thấy đốm sáng.

## Downstream contract

1. **Downstream task / model / user là ai?** Module traffic-light perception của hệ thống ADAS/AV: detector + state
   classifier, và bộ lọc chọn "đèn áp dụng cho ego lane" để planner quyết định dừng / đi.
2. **Output annotation nào thực sự cần?** Bbox ôm vỏ đèn (`traffic_light`) + attribute `state`
   (red/yellow/green/off/unknown), `relevance` (relevant/not_relevant/unknown), `pictogram`
   (circle/arrow_left/arrow_right/arrow_straight/unknown), `occluded`, `needs_review`; tag ảnh `image_escalate`.
3. **Failure nào gây hậu quả lớn nhất?** Đèn **đỏ/vàng đang điều khiển ego lane** bị gán sai `state` (thành green) hoặc
   sai `relevance` (thành not_relevant), hoặc bị bỏ sót không vẽ → planner cho xe vượt đèn đỏ. Đây là decision
   `critical`. Ngược lại, gán nhầm đèn xanh của làn khác thành relevant là `major` (xe đi khi làn mình đang đỏ).
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator không đoán: đặt giá trị `unknown`
   (khi thiếu bằng chứng về một attribute) và tick `needs_review` trên đèn đó; nếu cả ảnh không xác định được đèn nào
   điều khiển ego lane thì thêm tag `image_escalate`. QA owner của nhóm xử lý hàng đợi `needs_review` /
   `image_escalate` và quyết định có đổi guideline không.

## Scope

- **Trong scope (bắt buộc label):** mọi đầu đèn giao thông dành cho **xe** nhìn thấy được vỏ đèn hoặc bóng đèn sáng,
  cao ≥ 8 px, kể cả đèn quay mặt đi hướng khác (vẽ, `state=unknown`, `relevance=not_relevant`).
- **Ngoài scope (ignore, không vẽ):** đèn cho người đi bộ / xe đạp, đèn nhấp nháy của phương tiện, đèn đường, biển
  báo, đèn phản chiếu trên kính/xe, đèn cao < 8 px.
- **Geometry tolerance:** box ôm phần vỏ đèn nhìn thấy (không gồm cột, cần treo, tấm nền đen phía sau); lệch ≤ 2 px mỗi
  cạnh với đèn cao ≥ 20 px, ≤ 1 px với đèn nhỏ hơn. Đèn bị che: chỉ ôm phần nhìn thấy, tick `occluded`.

## Output chấm được

- **LABEL:** có box `traffic_light` trong export (đếm số đèn mỗi ảnh).
- **IGNORE:** không có box cho object ngoài scope (ví dụ đèn đi bộ).
- **UNKNOWN:** attribute = `unknown` trong export.
- **ESCALATE:** `needs_review` = true trên đèn, hoặc tag `image_escalate` trên ảnh.
- **Attribute:** `state`, `relevance`, `pictogram` đọc trực tiếp từ export; `__undefined__` còn lại = lỗi chưa gán.
- **Geometry:** so box với rule "ôm vỏ đèn" khi mở export trong CVAT.

## Dữ liệu và giới hạn

- `lisa`: 30 frame liên tiếp của **một** clip ban ngày → chỉ dùng cho example / calibration; các frame gần như giống
  nhau nên không dùng làm blind.
- `bdd100k`: 12 ảnh có đèn (theo README, gồm cả ảnh đêm và chạng vạng) → blind lấy từ đây để là cảnh chưa thấy.
- Chỉ là ảnh đơn → không suy được đèn nhấp nháy hay chuyển pha; `mutable` chỉ có ý nghĩa khi dùng track.
- Dữ liệu ở Mỹ: không có pha đỏ+vàng kiểu châu Âu nên bỏ giá trị `red_yellow`.
