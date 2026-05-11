# Phân vai SOT (Source of Truth)

Mỗi field SOT chỉ được sửa bởi đúng 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để apply.

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| KR.current, KI.current, action.status, plan counters              | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |

> Bảng này là bản canonical. Load cùng skill okr khi chạy.
