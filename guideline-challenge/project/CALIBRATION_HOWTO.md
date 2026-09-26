# Calibration nội bộ — làm từ A đến Z (~20 phút)

Mục đích: mỗi người label **cùng 7 ảnh**, **độc lập**, chỉ dựa vào `02_guideline.md`. Chỗ mọi người làm khác nhau =
chỗ guideline chưa rõ → sửa thành v2 trước khi gửi team09. Không có đáp án đúng/sai ở bước này; **đừng bàn nhau**.

Cần: Docker Desktop + CVAT đã cài từ Day 2, Chrome/Edge, repo nhóm đã clone.

## 1. Lấy bản mới và gom ảnh (2 phút)

```bash
git switch main && git pull
cd guideline-challenge
make cvat-status              # phải thấy: ✓ CVAT ... tại http://localhost:8080
make pack SPLIT=calibration   # tạo build/calibration/ với 7 ảnh
```

- Không có `make` (Windows): thay bằng `python lab9.py cvat` và `python lab9.py pack calibration`.
- `CVAT chưa chạy`: mở Docker Desktop → vào thư mục CVAT (`cvat-day2`) → `docker compose start` → chạy lại.
  Chi tiết: `GUIDE.md` mục 1 và mục 7.

7 ảnh phải có trong `build/calibration/`: `BDD02`, `BDD18`, `BDD22`, `BDD24`, `BDD25`, `LISA09`, `LISA16`.

## 2. Tạo task trên CVAT (5 phút)

1. Mở `http://localhost:8080`, đăng nhập → **Tasks** → nút **+** → **Create a new task**.
2. **Name:** `team01-calib-v1-<tên>` (ví dụ `team01-calib-v1-trung`).
3. **Labels** → tab **Raw** → xoá hết nội dung có sẵn → dán **toàn bộ** file `project/03_cvat_labels.json` → **Done**.
4. Chuyển tab **Constructor**, kiểm tra:
   - `traffic_light` (Rectangle) có 4 attribute: `state`, `relevance`, `pictogram`, `needs_review`;
   - `image_escalate` (Tag) có attribute `reason`;
   - default của `state` / `relevance` / `pictogram` / `reason` là `__undefined__`.
5. **Select files** → **My computer** → chọn **cả 7 ảnh** trong `build/calibration/`.
6. **Submit & Open**.
7. Ở trang task, dưới **Task description** bấm **Edit** → dán toàn bộ `project/02_guideline.md` → **Submit**.

Dán sai labels hoặc thiếu ảnh: **xoá task, tạo lại** — nhanh hơn sửa.

## 3. Cài đặt một lần (1 phút)

Tên tài khoản (góc trên phải) → **Settings** → **Workspace**:
- bật **Always show object details**;
- **Content of a text**: tick thêm **Dimensions** (để thấy box rộng × cao bao nhiêu px — ngưỡng là 6 px).

## 4. Label (10–15 phút)

1. Trang task → bấm dòng **Job #…** → màn hình gắn nhãn.
2. Bấm **Guide** (góc trên phải), đọc **Tóm tắt 30 giây**, **Quy trình làm một ảnh** và **Làm mẫu LISA03** trong
   guideline. Muốn xem hình minh hoạ: mở `02_guideline.md` trên GitHub.
3. Làm từng ảnh đúng **Quy trình làm một ảnh** (mục trong guideline): quét ảnh → vẽ box (**Draw new rectangle** →
   `traffic_light` → **Shape**) → gán 4 attribute (chế độ **Attribute annotation**) → tag nếu cần → **Ctrl+S** →
   **F** sang ảnh sau.
4. Ảnh không có đèn nào trong scope: **không vẽ gì**, vẫn **Ctrl+S**.

**Luật:**
- Không xem màn hình, không hỏi nhau, không xem file export của người khác.
- Gặp tình huống guideline không nói tới: làm theo cách **bạn** hiểu guideline, tick `needs_review`, và **ghi lại
  câu hỏi** (gửi Trung cùng file export) — đó chính là thứ calibration cần tìm.
- Không cần làm "đúng ý lead" — làm đúng những gì guideline viết.

**Tự kiểm trước khi export** (sidebar **Objects**, lướt 7 ảnh bằng **F**/**D**):
- [ ] không attribute nào còn `__undefined__`;
- [ ] mọi `unknown` đều có tick `needs_review`;
- [ ] không box nào trên đèn đi bộ, phản chiếu, đèn xe;
- [ ] đã **Ctrl+S**.

## 5. Export và gửi (2 phút)

1. Trong màn hình job: **Menu** (góc trên trái) → **Export job dataset**.
2. **Export format:** **CVAT for images 1.1** · **Save images:** **tắt** → **OK**.
3. Tải file từ thông báo hoặc trang **Requests** (thanh trên cùng).
4. Đổi tên file thành **`<tên>.zip`** — chữ thường, không dấu, không khoảng trắng (ví dụ `tri.zip`, `long.zip`,
   `ducuong.zip`, `trung.zip`).
5. Gửi file cho **Trịnh Nam Trung** qua nhóm chat, kèm các câu hỏi đã ghi ở bước 4 (nếu có).
   **Không push file export lên git** — Trung gom đủ rồi mới đưa vào `project/06_calibration_exports/`.

## Sau khi gửi

Trung chạy `make calib` so tất cả file → viết `06_calibration_report.csv` → Cường sửa guideline lên **v2** →
Thái Đức Cường chạy `make freeze` → gửi gói blind cho team09. Bạn không cần làm gì thêm cho calibration.

Kẹt kỹ thuật (CVAT, Docker, lệnh) quá 3 phút: gọi Lab Coach. Hỏi về rule gán nhãn: **không hỏi**, ghi lại câu hỏi
như bước 4.
