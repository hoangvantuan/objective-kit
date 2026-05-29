---
name: okr-plan
description: "Tạo hoặc sửa plan, milestones, actions, dependencies, deadline, external_ids. Load khi cần plan-new, plan-update. Cũng nhận pre-confirmed từ okr-track deep để áp dụng thay đổi cấu trúc."
---

# OKR Plan: Tạo + cập nhật plan & actions

Quản lý 2 SOT: `plan.md` và `actions/`.

## Modes

| Mode | Trigger | Flow chi tiết |
|------|---------|---------------|
| `new` | Chưa có plan.md | `references/flow-new.md` |
| `update` | Sửa plan/action/milestone | `references/flow-update.md` |
| `pre-confirmed` | Áp dụng thay đổi cấu trúc từ okr-track deep | `references/flow-update.md` + `okr-shared` delegate-protocol |

## SOT quyền ghi

| File | Fields |
|------|--------|
| plan.md | Milestones, counters (khi tạo plan) |
| actions/*.md | title, deadline, deps, effort, priority, DoD, quality criteria, notes, external_ids |

> Subset của bảng canonical `../okr-shared/references/sot-ownership.md`. Sửa canonical trước, bảng này theo sau.

## Điều kiện tiên quyết

- `objective.md` tồn tại. Thiếu → cần init trước.
- `resources.md` nên tồn tại. Thiếu → cảnh báo cross-check fit.

## Nguyên tắc

- Mỗi action BẮT BUỘC: DoD rõ ràng, Output/Deliverable, tiêu chí chất lượng.
- Action mơ hồ ("Nghiên cứu thêm" không output) → CẤM.
- Effort xl → BẮT BUỘC Checkpoints hoặc tách.
- Confirm bảng trước ghi. Render Roadmap sau ghi.

## Schema

- `references/data-format.md`: schema plan.md + action frontmatter
- `references/action-guide.md`: viết action chất lượng, anti-patterns
- `references/task-format.md`: template body action file
- Roadmap format: xem `okr-shared` skill `schemas.md`
