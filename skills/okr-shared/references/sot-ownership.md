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
| Bài học (`.okr/lessons/**`: 3 ngăn skill/workflow/project, tạo/sửa/thăng scope/đánh dấu obsolete, ported; gồm cả dọn kho mode `consolidate`) | `okr-retro` |
| `## Tài liệu & Knowledge Base` (resources.md): đăng ký nguồn DÙNG | `okr-init` `update-resource`  |
| `context/<slug>.md` (nội dung cross-cutting)                     | Owner = skill tạo file (`okr-init`/`okr-plan`/`okr-track`), 1 file 1 owner |
| `context/index.md` entry (đăng ký file context)                 | Append-only đa-skill. KEY = cột `Path`. Mỗi entry single-owner = skill tạo. Cấm sửa entry owner khác |
| Reachability audit (phát hiện mồ côi / link chết)               | `okr-analyze` (read-only, Light + Deep)          |

> **`context/index.md` đa-skill nhưng không vi phạm "1 field 1 skill":** index chỉ là TẬP HỢP entry single-owner. Mỗi entry có 1 owner (skill tạo file), chỉ owner đó được sửa entry của mình. Nhiều skill ghi cùng bảng, nhưng không skill nào sửa entry của skill khác. Bản đồ neo canonical: `preload.md` "Reachability khi ghi".
> Bảng này là bản canonical. Load cùng skill okr khi chạy.
