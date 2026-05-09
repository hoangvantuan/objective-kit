---
name: ok
description: "Quản lý mục tiêu & dự án theo OKR. Entry point chính, tự phát hiện phase phù hợp. Gọi /ok để bắt đầu hoặc tiếp tục dự án. Gọi /ok init|plan|resource|track|review để vào phase cụ thể."
---

# ok: Orchestrator quản lý dự án

Entry point chính. Phân tích trạng thái `.ok/` rồi route đến skill phù hợp.

## Logic routing

Khi user gọi `/ok` không kèm tham số:

1. `.ok/` chưa tồn tại → thông báo "Chưa có mục tiêu. Dùng `/ok-init` để khởi tạo."
2. `.ok/objective.md` chưa có → thông báo tương tự
3. `.ok/plan.md` chưa có → thông báo "Có mục tiêu nhưng chưa có kế hoạch. Dùng `/ok-plan` để lập plan."
4. Đọc frontmatter tất cả `.ok/actions/*.md`:
   - Tất cả status=done → thông báo "Mọi action hoàn thành. Dùng `/ok-review` để review tổng kết."
   - Có status=blocked → hiển thị blockers + hỏi user muốn `/ok-track` hay `/ok-review`
   - Có actions đang thực thi → thông báo "Dự án đang thực thi. Dùng `/ok-track` để cập nhật tiến độ."

Khi user gọi `/ok <phase>`:

- `/ok init` → hướng dẫn dùng `/ok-init`
- `/ok plan` → hướng dẫn dùng `/ok-plan`
- `/ok resource` → hướng dẫn dùng `/ok-resource`
- `/ok track` → hướng dẫn dùng `/ok-track`
- `/ok review` → hướng dẫn dùng `/ok-review`

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
