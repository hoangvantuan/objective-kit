---
name: okr-plan
description: "Lập kế hoạch hành động cho dự án. Phân tách Key Results thành milestones và actions cụ thể. Dùng khi có objective nhưng chưa có plan. Trigger: /okr-plan, hoặc tự động khi /okr phát hiện thiếu plan.md"
---

# okr-plan: Lập kế hoạch hành động

Đọc `.okr/objective.md`, phân tách Key Results thành milestones và actions, tạo `.okr/plan.md` + `.okr/actions/`.

## Điều kiện tiên quyết

- `.okr/objective.md` phải tồn tại. Nếu chưa có, hướng user chạy `/okr-init` trước.

## Checklist

1. Đọc `.okr/objective.md` để hiểu Objective và Key Results
2. Với mỗi Key Result, đề xuất Initiatives (sáng kiến lớn)
3. Với mỗi Initiative, phân tách thành Actions cụ thể
4. Nhóm actions thành Milestones (theo thời gian)
5. Xác định dependencies giữa actions
6. Hỏi user về deadline milestones
7. Hỏi user về PIC (Person In Charge) cho từng action nếu có nhiều người
8. Tạo `.okr/plan.md` với roadmap tổng
9. Tạo file cho mỗi action trong `.okr/actions/`

## Format output

Đọc `references/data-format.md` để lấy schema cho `plan.md` và `actions/`.
Đọc `references/task-format.md` để lấy template action file.

## Quy tắc

- Mỗi action phải có Definition of Done rõ ràng
- Mỗi action phải có Output/Deliverable cụ thể
- Không tạo action mơ hồ kiểu "Nghiên cứu thêm" mà không có output
- Dependencies phải hợp lệ (ID tồn tại, không vòng tròn)
- Cập nhật `total_actions` trong frontmatter plan.md
- Nếu objective.md type=habit, plan.md chứa recurring tasks thay vì milestones
