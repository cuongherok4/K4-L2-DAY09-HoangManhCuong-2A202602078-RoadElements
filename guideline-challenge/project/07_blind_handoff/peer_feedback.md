# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền. Thay mọi placeholder mới
là xong (gate G5).

- **Nhóm peer:** team09
- **Người label blind:** Nguyễn Tú Anh
- **Job:** CVAT task 21 / job 25 — 5 ảnh: BDD07, BDD11, BDD17, BDD21, BDD26
- **Thời gian label:** 17 phút (tính đến lúc label xong, **chưa gồm** thời gian tự kiểm lại)
- **Kết quả:** 12 box `traffic_light`, 0 tag `image_escalate`

| Ảnh | Box | Tóm tắt |
| --- | --- | --- |
| BDD07 | 2 | 2 đầu đèn đi thẳng `green/circle/relevant`. Không vẽ: vỏ vàng chỉ thấy cạnh, 2 đèn xa < 6 px, phản chiếu trên capo |
| BDD11 | 0 | Chỉ có hộp đèn đi bộ bàn tay đỏ, nên không vẽ |
| BDD17 | 4 | Trời mưa, kính ướt. Cả 4 đèn đều có `relevance = unknown` và `needs_review` (1 đèn lõi cam `state = unknown`, 1 đèn đỏ, 2 đèn xanh) |
| BDD21 | 2 | 1 đèn trên cần treo `green/circle/relevant`; 1 đèn xanh xa `green/unknown/unknown` + `needs_review` |
| BDD26 | 4 | Ảnh đêm. 1 đèn xanh lớn `green/circle/relevant` (box ôm lõi sáng); 3 vật không chắc (đèn xanh xa, vật đỏ hình oval, vật hồng/xanh xa), cả 3 `unknown` + `needs_review` |

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?**
   - Rule "**vị trí ô quyết định `state`**" (mục 4.1) và "**xe luôn đi thẳng** nếu không thấy mũi tên sơn trên đường"
     (mục 4.3). Hai rule này bỏ được phần lớn việc phải đoán.
   - Cây quyết định mục 7, bước 1–4 (phản chiếu / đèn xe cơ giới / thấy mặt đèn / ≥ 6 px) giúp loại nhanh phản
     chiếu trên capo (BDD07, BDD26), hộp đèn đi bộ bàn tay (BDD11) và các vỏ đèn chỉ thấy cạnh (BDD07, BDD21).
   - Quy tắc "`unknown` luôn kèm `needs_review`" và checklist cuối mục "Quy trình" dễ tự kiểm.

2. **Rule nào mơ hồ hoặc phải tự suy diễn?**
   - **Relevance khi không thấy vạch dừng** (mục 4.3): cách nhận "điểm dừng gần nhất" bắt đầu từ việc tìm vạch
     dừng / vạch sang đường. Ở BDD17, BDD21 và BDD26 không thấy rõ vạch dừng nào gắn với các đèn nhỏ ở xa, nên phải
     tự chọn giữa `not_relevant` (theo gợi ý "đèn nhỏ, thẳng hàng cuối đường") và `unknown`.
   - **`state` khi không thấy vỏ và lõi có màu cam** (mục 4.1): một câu nói "chỉ dùng `unknown` khi không xác định được
     ô nào đang sáng **và** màu không phân biệt được", câu sau lại nói lõi cam "nếu các đầu đèn khác cùng hướng thấy
     rõ là đỏ thì `red`, không chắc thì `unknown`". Ở BDD17 các đầu đèn khác đều xanh, không có đầu nào đỏ để đối
     chiếu, nên phải tự suy ra là `unknown`.
   - **Ranh giới "lõi sáng" và "quầng loá"** (mục 3) không có tiêu chí đo. Với đèn 6–8 px, lệch 1 px là đổi luôn
     quyết định vẽ hay không vẽ theo ngưỡng 6 px (ví dụ đèn xanh xa ở BDD26 và BDD17).
   - **`pictogram` với đèn 6–12 px**: mục 4.2 nói "lõi loá nhưng tròn đều thì `circle`", mục 6 lại nói "nhỏ/xa, hình không
     rõ thì `unknown`". Chưa rõ đèn nhỏ nhưng trông tròn thì chọn giá trị nào.
   - **Đèn đi bộ ban đêm**: mục 5 nhận biết đèn đi bộ qua "hộp vuông ở cột ngang tầm người, hình bàn tay". Ban đêm
     không thấy hộp, chỉ thấy một vật đỏ cam hình oval ngay dưới đèn xe cùng cột (BDD26). Khi đó rule "không chắc →
     vẽ + `needs_review`" và rule "đèn đi bộ → không vẽ" mâu thuẫn nhau.

3. **Sample nào khiến guideline "vỡ"?**
   - **BDD17** (mưa, kính ướt): cùng lúc có đèn đỏ và đèn xanh ở xa, không xác định được đèn nào thuộc điểm dừng gần
     nhất, nên cả 4 đèn đều `relevance = unknown`. Hệ quả là:
     - `conflicting_lights` không áp dụng được, vì rule yêu cầu hai đèn "cùng được coi là `relevant`";
     - `low_visibility` cũng không áp dụng được, vì rule yêu cầu "không đèn nào của điểm dừng gần nhất đọc được
       `state`", mà ở đây không biết đèn nào thuộc điểm dừng gần nhất.

     Vì vậy một ảnh có tín hiệu đỏ/xanh lẫn lộn lại **không được escalate cấp ảnh**.
   - **BDD26** (đêm):
     - vật đỏ cam hình oval dưới đèn xanh cùng cột: không phân biệt được đèn đi bộ (bàn tay) hay đèn xe;
     - vật hồng/trắng có mảng xanh bên dưới ở xa (khoảng x 661–674, y 209–232): không phân biệt được đèn tín hiệu
       hay biển hiệu.

     Cả hai đều được vẽ theo rule "không chắc → vẽ + `needs_review`", nhưng nếu là đèn đi bộ thì đây là lỗi critical
     "false STOP".
   - **BDD21 / BDD07**: đèn ở giao lộ xa chỉ 2–7 px, sát ngưỡng 6 px. Chỉ cần lệch 1 px khi kéo box là kết quả đổi.

4. **Attribute / default nào trong CVAT dễ gây thao tác sai?**
   - Thứ tự attribute trong CVAT là `state` → `relevance` → `pictogram` → `needs_review` (theo spec id 49, 50, 51, 52),
     còn guideline (bước 5) hướng dẫn `state` → `pictogram` → `relevance`. Trong chế độ **Attribute annotation**, dùng
     ↑/↓ theo thứ tự của guideline dễ chọn nhầm giá trị vào attribute khác.
   - Checkbox `needs_review` mặc định là `false`, **không** có trạng thái `__undefined__`. Nếu quên tick thì export
     vẫn hợp lệ, QA không phát hiện được lỗi thiếu tick.
   - Giá trị `unknown` có ở cả 3 attribute select, nên dễ chọn `unknown` cho nhầm attribute khi thao tác nhanh bằng
     phím số.
   - Box 6–8 px rất khó vẽ chính xác bằng hai cú click. Text **Dimensions** chỉ hiện khi bật trong Settings, mà bước
     này dễ bị bỏ qua.

5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?**
   Thêm vào mục 4.3 và mục 7 một rule cho trường hợp **không xác định được điểm dừng gần nhất**:
   > Nếu không thấy vạch dừng / vạch sang đường gắn với đèn → mọi đèn chưa rõ điểm dừng gán `relevance = unknown` +
   > `needs_review`. Nếu trong số các đèn `unknown` đó có đèn đỏ/vàng và đèn xanh cùng quay về phía xe mình → thêm tag
   > `image_escalate` với `reason = lane_unclear` (hoặc thêm reason mới, ví dụ `stop_unclear`).

   Rule này xử lý được trường hợp BDD17. Ngoài ra nên bổ sung thêm:
   - (a) một ví dụ ảnh đêm có đèn đi bộ bàn tay cam **không thấy hộp**;
   - (b) cách đo "lõi sáng" (ví dụ: vùng có màu rõ, không tính viền mờ).

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| BDD26 d2 (critical): peer vẽ box lên hộp đèn đi bộ ban đêm (433-446 x 109-131, `state=red`, `needs_review`) | guideline gap | accept + revise | Guideline v2 mục "Không vẽ" (dòng 29) và bảng mục 5 (dòng 291) cấm vẽ đèn đi bộ, nhưng dòng 177 lại cho "không chắc có phải đèn → giữ box, tick `needs_review`". Ban đêm không thấy hộp/bàn tay nên hai rule mâu thuẫn; peer nêu đúng mâu thuẫn này ở câu 2 và câu 3. v3: vật đỏ/cam ngay dưới đèn xe cùng cột, ban đêm → coi là đèn đi bộ, không vẽ; không chắc thì dùng tag ảnh, không vẽ box |
| BDD17 d1 (major): thiếu tag `image_escalate` | guideline gap | add escalation rule | Bảng reason (dòng 344–345): `conflicting_lights` cần hai đèn "cùng được coi là `relevant`", `low_visibility` cần xác định được điểm dừng gần nhất. Khi mọi đèn đều `relevance=unknown` thì không reason nào áp dụng (peer câu 3, câu 5). v3: thêm reason cho trường hợp không xác định được điểm dừng mà có đèn đỏ/vàng lẫn xanh |
| FB câu 2: relevance khi không thấy vạch dừng / vạch sang đường (mục 4.3) | guideline gap | accept + revise | BDD17, BDD21, BDD26 không có vạch dừng gắn với các đèn xa; peer phải tự chọn giữa `not_relevant` và `unknown`. v3: không thấy vạch dừng gắn với đèn → `relevance=unknown` + `needs_review` |
| FB câu 2: `state` khi lõi màu cam và không có đầu đèn đỏ để đối chiếu | guideline gap | accept + revise | Dòng 219 chỉ nói trường hợp "đầu đèn khác cùng hướng thấy rõ là đỏ"; BDD17 các đầu còn lại đều xanh. v3: không có đầu đèn đối chiếu → `unknown` + `needs_review` |
| FB câu 2–3: ranh giới lõi sáng / quầng loá sát ngưỡng 6 px | data ambiguity | add example | Dòng 177; đèn xanh xa ở BDD17, BDD26 chỉ 6–8 px, lệch 1 px đổi quyết định vẽ/không vẽ. v3: thêm ví dụ + tiêu chí "chỉ tính vùng có màu rõ, không tính viền mờ" |
| FB câu 2: `pictogram` cho đèn 6–12 px trông tròn | guideline gap | accept + revise | Mục 4.2 "lõi loá nhưng tròn đều → `circle`" mâu thuẫn dòng 317 "nhỏ/xa, hình không rõ → `pictogram = unknown`". v3: cạnh ngắn < 12 px → luôn `unknown` |
| FB câu 4: thứ tự attribute trong CVAT khác bước 5 của guideline | guideline gap | accept + revise | `03_cvat_labels.json` (đã freeze): `state` → `relevance` → `pictogram` → `needs_review`; guideline dòng 86: `state` → `pictogram` → `relevance`. v3: sửa bước 5 theo thứ tự JSON, không đổi JSON |
| FB câu 4: `needs_review` mặc định `false`, quên tick không lộ trong export | data ambiguity | reject with evidence | Schema đã freeze nên không đổi default trong đợt này; `05_qa_plan.md` mục Self-QC đã yêu cầu "mọi `unknown` đều có `needs_review`", nên lỗi thiếu tick bắt được ở bước tự kiểm |
| FB câu 5 (b): thêm ví dụ đèn đi bộ bàn tay cam ban đêm không thấy hộp | guideline gap | accept + revise | Gắn với lỗi BDD26 d2; v3 thêm một ví dụ từ ảnh example/calibration có đèn đi bộ ban đêm |
