# Phân vai SOT (Source of Truth)

Mỗi field SOT chỉ được sửa bởi skill được chỉ định. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để apply. Ngoại lệ: `action.status` được cả track lẫn plan sửa.

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| Action `## Output/Deliverable` (ghi đè output thực tế)           | `okr-track` `light`/`deep`    |
| KR.current, KI.current, plan counters                             | `okr-track` `light`/`deep`    |
| plan.last_track_date, plan.last_review_date                       | `okr-track` `light`/`deep`    |
| action.status                                                     | `okr-track` `light`/`deep`, `okr-plan` `update` |
| action.completed_date (set khi status→done)                       | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |
| Bài học (`.okr/lessons/**`: tạo/sửa/đánh dấu obsolete, ported)   | `okr-retro`                   |

> Bảng này là bản canonical. Load cùng skill okr khi chạy.
