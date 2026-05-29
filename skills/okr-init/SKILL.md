---
name: okr-init
description: "Tạo hoặc sửa objective, KR, KI, period, deadline, resources, capacity, skill, tool, ngân sách. Load khi cần init-new, update-objective, update-resource. Cũng nhận pre-confirmed từ okr-track deep để áp dụng thay đổi cấu trúc."
---

# OKR Init: Khởi tạo + cập nhật objective & resource

Quản lý 2 SOT: `objective.md` và `resources.md`.

## Modes

| Mode | Trigger | Flow chi tiết |
|------|---------|---------------|
| `new` | Chưa có `.okr/` hoặc objective.md thiếu | `references/flow-new.md` |
| `update-objective` | Sửa objective/KR/KI/period | `references/flow-update-objective.md` |
| `update-resource` | Sửa capacity/skill/tool/ngân sách | `references/flow-update-resource.md` |

## SOT quyền ghi

| File | Fields |
|------|--------|
| objective.md | Objective text, KR/KI target/baseline/ngưỡng, period, status |
| resources.md | Solo Profile (capacity, skills), tool, tài liệu, ngân sách |

> Subset của bảng canonical `../okr-shared/references/sot-ownership.md`. Sửa canonical trước, bảng này theo sau.

## Nguyên tắc

- Hỏi từng câu một, không hỏi hàng loạt.
- BẮT BUỘC confirm bảng trước khi ghi file.
- Quality Gate 3 câu trước mỗi follow-up (xem `okr-shared` skill).
- Lessons: trước khi đề xuất/ghi, đảm bảo `.okr/lessons/index.md` đã nạp + áp dụng lesson liên quan (xem `okr-shared` skill "Auto-load lessons"). Quan trọng khi chạy lẻ, không qua `okr-harness`.
- Solo only: 1 user, 1 objective.
- Đề xuất + lý do, user quyết.

## Schema

- `references/data-format.md`: schema objective.md + resources.md
- `references/okr-guide.md`: tiêu chí SMART KR, KI guidelines

## Quy tắc

- KHÔNG ghi file trước confirm.
- KR không SMART → chỉ rõ thiếu gì + gợi ý sửa.
- Không tạo plan.md hay action files (việc okr-plan).
- Không sửa action status (việc okr-track).
