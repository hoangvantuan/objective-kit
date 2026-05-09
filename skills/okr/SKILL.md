---
name: okr
description: "Quản lý mục tiêu & dự án theo OKR. Entry point chính, tự phát hiện phase phù hợp. Gọi /okr để bắt đầu hoặc tiếp tục dự án. Gọi /okr init|plan|resource|track|review để vào phase cụ thể."
---

# okr: Orchestrator quản lý dự án

Entry point chính. Phân tích trạng thái `.okr/` rồi route đến skill phù hợp.

## Logic routing

Khi user gọi `/okr` không kèm tham số:

1. `.okr/` chưa tồn tại → thông báo "Chưa có mục tiêu. Dùng `/okr-init` để khởi tạo."
2. `.okr/objective.md` chưa có → thông báo tương tự
3. `.okr/plan.md` chưa có → thông báo "Có mục tiêu nhưng chưa có kế hoạch. Dùng `/okr-plan` để lập plan."
4. Đọc frontmatter tất cả `.okr/actions/*.md`:
   - Tất cả status=done → thông báo "Mọi action hoàn thành. Dùng `/okr-review` để review tổng kết."
   - Có status=blocked → hiển thị blockers + hỏi user muốn `/okr-track` hay `/okr-review`
   - Có actions đang thực thi → thông báo "Dự án đang thực thi. Dùng `/okr-track` để cập nhật tiến độ."

Khi user gọi `/okr <phase>`:

- `/okr init` → hướng dẫn dùng `/okr-init`
- `/okr plan` → hướng dẫn dùng `/okr-plan`
- `/okr resource` → hướng dẫn dùng `/okr-resource`
- `/okr track` → hướng dẫn dùng `/okr-track`
- `/okr review` → hướng dẫn dùng `/okr-review`

## Hiển thị khi routing

Trước khi route, hiển thị trạng thái ngắn gọn:

```
Dự án: [Objective]
Period: [start] > [end]
KR: X/Y đạt | Actions: X done, Y doing, Z blocked
Đề xuất: [phase phù hợp]
```

## Quy tắc

- Skill này CHỈ phân tích và route. Không tạo/sửa file nào
- Đọc frontmatter tối thiểu (objective.md + glob actions/*.md)
- Luôn hiển thị trạng thái ngắn trước khi đề xuất phase
