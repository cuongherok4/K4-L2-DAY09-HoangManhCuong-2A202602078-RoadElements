# Problem statement + downstream contract

Tối đa nửa trang, viết **trước khi mở CVAT**. Đây là bằng chứng của gate G1 (topic lock). Thay mọi placeholder
mới là xong.

## Bài toán

Gắn nhãn **state** và **ego relevance** của đèn giao thông cho xe cơ giới trong ảnh dashcam, ở giao lộ có nhiều đầu
đèn. Khó ở ba chỗ: (1) một giao lộ có nhiều đầu đèn — đèn mũi tên rẽ, đèn lặp lại ở cột gần/xa, đèn giao lộ kế tiếp;
(2) đèn nhỏ/xa, ban đêm, mưa, loá sáng làm khó đọc state; (3) nhiều thứ trông giống đèn nhưng không phải: đèn đi bộ,
đầu đèn quay ngang, phản chiếu trên nắp capo/mặt đường ướt.

## Downstream contract

1. **Downstream task / model / user là ai?** Module perception của ADAS: detector đèn + classifier state + bộ chọn
   "đèn nào điều khiển xe ego", cấp tín hiệu STOP/GO cho planning ở **điểm dừng có đèn gần nhất phía trước** (giao lộ, vạch sang đường giữa
   đoạn, ramp meter).
2. **Output annotation nào thực sự cần?** Rectangle `traffic_light` cho từng đầu đèn; attribute `state`, `relevance`,
   `pictogram` (tròn/mũi tên — bằng chứng cho relevance), `needs_review`; tag ảnh `image_escalate` có `reason`. Không
   label cột đèn, đèn đi bộ/xe đạp.
3. **Failure nào gây hậu quả lớn nhất?** (a) Đèn relevant đang `red`/`yellow` bị bỏ sót, gán `green` hoặc gán
   `not_relevant` → xe vượt đèn đỏ. (b) Đèn không điều khiển ego (mũi tên rẽ trái đỏ, đèn giao lộ kế tiếp) gán
   `relevant` → xe phanh gấp giữa giao lộ. (c) Phản chiếu/đèn đi bộ bị label thành đèn → false STOP. Đây là các
   decision `critical` trong gold.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?** Annotator không đoán: giá trị không chắc để
   `unknown` + tick `needs_review`; cả ảnh không đọc được hoặc các đèn relevant mâu thuẫn nhau thì thêm tag
   `image_escalate`. QA owner xử lý trong vòng review; quyết định mới được ghi thành rule/ví dụ, tăng version.

## Scope

- **Trong scope (bắt buộc label):** mọi đầu đèn tín hiệu cho xe cơ giới (giao lộ, vạch sang đường, ramp meter, đèn
  công trường tạm) **thấy được mặt đèn**, cạnh ngắn box ≥ 6 px ở ảnh gốc — kể cả đèn tắt, bị che một phần, bị cắt mép ảnh, ở giao lộ xa.
- **Ngoài scope (không vẽ):** đầu đèn quay lưng/quay ngang, đèn đi bộ/xe đạp, đèn riêng buýt/tàu điện, đèn điều
  khiển làn (X đỏ), đèn chắn tàu, đèn xe cộ, đèn đường, biển điện tử, mọi phản chiếu (capo, kính, mặt đường ướt), đèn
  có cạnh ngắn < 6 px.
- **Geometry tolerance:** box ôm sát **vỏ đèn nhìn thấy** (không gồm cột, cần treo, biển phụ, tấm hắt); đêm không
  thấy vỏ thì ôm bóng đèn đang sáng, không gồm quầng sáng. Mỗi cạnh lệch ≤ 2 px (đèn rộng < 20 px) hoặc ≤ 10% chiều
  tương ứng (đèn lớn hơn).

## Output chấm được

LABEL = có box `traffic_light` cho từng đầu đèn trong scope (đúng số lượng); IGNORE = không có box cho thứ ngoài scope;
attribute `state` và `relevance` đúng giá trị; UNKNOWN = giá trị `unknown`; ESCALATE = `needs_review = true` hoặc tag
`image_escalate`; geometry theo tolerance trên. Tất cả nằm trong export CVAT for images 1.1; còn `__undefined__` trong
export = annotator quên gán, tính là sai.

## Dữ liệu và giới hạn

- `bdd100k` (nguồn chính): 9 ảnh có đèn xe cơ giới (ngày, mưa, chạng vạng, đêm), 3 ảnh chỉ có đèn đi bộ, còn lại
  không có đèn → làm case `negative`. **Toàn bộ ảnh blind lấy từ BDD.**
- `lisa` (bổ sung): 30 frame liên tiếp của một giao lộ lúc chạng vạng, đèn đỏ → xanh ở frame 16, luôn có đèn mũi tên
  rẽ trái đỏ cạnh đèn đi thẳng. Chỉ dùng 4 frame cách xa nhau cho example/calibration, label như ảnh tĩnh (không track).
- Giới hạn: relevance chỉ suy ra từ một ảnh tĩnh, không biết xe sẽ rẽ → quy ước ego **đi thẳng trong làn hiện tại**,
  trừ khi thấy rõ ego ở làn chỉ-rẽ. Ít ảnh đêm/thời tiết xấu (2 đêm, 2 chạng vạng, 1 mưa).
