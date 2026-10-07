# BTVN5 – Stage 02: Tool `list_files` + Skill `refund-policy`

Agent LangChain trả lời câu hỏi **"đơn hàng có được hoàn tiền không?"** dựa trên tài liệu chính sách trong workspace, không hard-code tên file hay đáp án trong prompt.

## Đã thêm gì

| Loại | Thành phần | Tác dụng |
|---|---|---|
| **Tool** | `list_files(path)` ([tools/files.py](stage-02-skills-baitap/tools/files.py)) | Liệt kê file/thư mục (không đệ quy, sắp theo tên). Giúp agent **tìm file thật lúc chạy** thay vì đoán tên. Chặn path tuyệt đối, `~`, `..`, symlink thoát workspace. |
| **Skill** | `refund-policy` ([SKILL.md](stage-02-skills-baitap/workspace/skills/refund-policy/SKILL.md)) | Quy trình 7 bước: kiểm tra đủ thông tin → `list_files data/policies` → đọc chính sách → chọn theo **ngày mua** (không theo tên file) → tính số ngày → kết luận → trả lời theo mẫu. |
| **Reference** | [answer-template.md](stage-02-skills-baitap/workspace/skills/refund-policy/references/answer-template.md) | Mẫu trả lời 5 mục: chính sách áp dụng, số ngày, kết luận, phí, căn cứ. |
| **Policy** | [tai-lieu-1.md](stage-02-skills-baitap/workspace/data/policies/tai-lieu-1.md) | Từ 2026-10-01: hoàn trong 14 ngày, không phí. |
| **Policy** | [tai-lieu-2.md](stage-02-skills-baitap/workspace/data/policies/tai-lieu-2.md) | Trước 2026-10-01: hoàn trong 7 ngày, phí 10%. |
| **Agent** | [agent.py](stage-02-skills-baitap/agent.py) | Đăng ký `TOOLS = [list_files, read_file, write_file]`, nạp skill catalog vào system prompt. |

Cả hai chính sách: **không hoàn tiền nếu sản phẩm đã kích hoạt**.

## Hiệu quả

| | Stage 00 (không tool) | Stage 02 (tool + skill) |
|---|---|---|
| Tìm tài liệu | Không, tự đoán 7/14/30 ngày | `list_files` thấy đúng file |
| Kết luận | Không kết luận được | Đúng, có dẫn file căn cứ |
| Thiếu thông tin | — | Hỏi lại, không giả định |
| Đổi tên file | — | Vẫn đúng (chọn theo nội dung) |

Kết quả chạy thật (trace trong [traces/](stage-02-skills-baitap/traces/)):

- **Case A** (mua trước 10/2026, 8 ngày) → không đủ điều kiện (giới hạn 7 ngày).
- **Case B** (mua từ 10/2026, 10 ngày) → đủ điều kiện, không thu phí (giới hạn 14 ngày).
- **Thiếu trạng thái kích hoạt** → agent chỉ hỏi lại, không kết luận.
- **Đổi tên** `policy-*.md` → `tai-lieu-*.md` (đảo thứ tự) → kết luận A, B không đổi.

## Vì sao cần cả tool và skill

- **Tool = khả năng**: model không thấy hệ thống file; chỉ `list_files` mới đưa tên file thật vào context.
- **Skill = quy trình**: chỉ cách chọn đúng chính sách, tính ngày, khi nào hỏi lại. Chỉ nạp khi câu hỏi khớp, không làm nặng context.
- **Sửa prompt không thay được tool**: prompt viết cố định trước khi chạy, file đổi tên là `read_file` lỗi `FILE_NOT_FOUND`.

Chi tiết phân tích: [analysis.md](analysis.md).
