# Quy trình commit và chờ lead duyệt

> Áp dụng cho repo nhóm `guideline-challenge`. Mỗi thành viên làm trên nhánh riêng và gửi thay đổi qua Pull Request (PR); không push trực tiếp lên nhánh mặc định.

## 1. Đồng bộ và tạo nhánh

Mở terminal tại thư mục repo, chuyển về nhánh mặc định (thường là `main`) và lấy bản mới nhất:

```bash
git switch main
git pull origin main
git switch -c feat/ten-noi-dung
```

Đặt tên nhánh theo dạng `<loai>/<noi-dung-ngan>`:

- `feat/traffic-light-rules`: thêm hoặc thay đổi nội dung/tính năng.
- `fix/cvat-labels`: sửa lỗi.
- `docs/qa-plan`: cập nhật tài liệu.
- `chore/update-sample-pack`: công việc bảo trì.

Nếu nhánh mặc định của repo không phải `main`, dùng tên nhánh mặc định đang hiển thị trên GitHub.

## 2. Kiểm tra và tạo commit

Chỉ đưa vào commit các file thuộc phần mình phụ trách. Trước khi commit, xem lại thay đổi:

```bash
git status
git diff
git add project/ten_file.md
git diff --cached
git commit -m "docs(qa): clarify review escalation rules"
```

Đặt commit message theo mẫu:

```text
<loai>(<pham-vi>): <mo-ta-ngan-bang-tieng-Anh>
```

Loại thường dùng: `feat` (bổ sung), `fix` (sửa lỗi), `docs` (tài liệu), `test` (kiểm thử), `chore` (bảo trì). Mô tả bắt đầu bằng động từ, nói rõ thay đổi và giữ ngắn gọn.

Ví dụ:

- `docs(guideline): clarify unknown traffic-light state`
- `fix(schema): correct relevance attribute values`
- `test(qa): add critical red-light cases`
- `chore(project): update team responsibilities`

Mỗi commit nên là một thay đổi có mục đích rõ ràng. Không dùng các message mơ hồ như `update`, `fix`, `final`, `abc` hoặc `sửa bài`. Không dùng `git add .` nếu chưa kiểm tra kỹ danh sách file; không đưa dữ liệu cá nhân, secret, file tạm hoặc thay đổi ngoài phạm vi vào commit.

## 3. Push nhánh và mở PR

Push nhánh của bạn lên GitHub:

```bash
git push -u origin feat/ten-noi-dung
```

Trên GitHub, tạo Pull Request với:

- **Base:** nhánh mặc định của repo.
- **Compare:** nhánh vừa push.
- **Title:** theo mẫu commit, ví dụ `docs(qa): clarify review escalation rules`.
- **Description:** nêu mục đích, file/phần đã đổi, cách tự kiểm tra và điểm cần lead xem kỹ.

Trong PR, gắn lead làm reviewer. PR là yêu cầu review, chưa phải thay đổi đã được chấp nhận: không tự merge và không push thẳng lên nhánh mặc định. Nếu lead yêu cầu chỉnh sửa, tiếp tục commit trên cùng nhánh và push lại; PR sẽ tự cập nhật.

## 4. Checklist trước khi yêu cầu duyệt

- [ ] Đã pull nhánh mặc định mới nhất trước khi làm.
- [ ] Đã kiểm tra `git status` và nội dung `git diff`/`git diff --cached`.
- [ ] Commit chỉ chứa thay đổi liên quan, message đúng quy ước.
- [ ] Đã chạy kiểm tra phù hợp với thay đổi (ví dụ `make check` khi sửa deliverable của challenge).
- [ ] PR có mô tả, đúng base branch và đã gắn lead làm reviewer.
- [ ] Không sửa gold hoặc sample pack sau khi đã freeze; nếu thay đổi này liên quan đến gold, dừng lại và báo lead trước.

Chỉ merge sau khi lead approve. Sau khi PR được merge, cập nhật bản local:

```bash
git switch main
git pull origin main
```