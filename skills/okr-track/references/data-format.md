# Data Format: log/ và log/reviews/

Skill `okr-track` ghi 2 loại log tuỳ mode (light hay deep).

## Mode LIGHT: log/YYYY-MM-DD.md

```yaml
---
date: YYYY-MM-DD
type: tracking
---
```

Body:
- `## Thay đổi` (danh sách thay đổi cụ thể, vd: "KR1: 40 > 50", "A003: doing > done")
- `## Ghi chú` (free-form, blockers, observations)

File ngày đã có → append section mới `## Thay đổi (lần N, HH:MM)`, KHÔNG ghi đè.

## Mode DEEP: log/reviews/YYYY-MM-DD.md

```yaml
---
date: YYYY-MM-DD
type: review
period: "YYYY-MM-DD to YYYY-MM-DD"
mode: deep | closure
---
```

Body:
- `## Tổng kết` (bảng KR: Target, Current, %, Trend)
- `## Phân tích`
  - `### Đạt tốt`
  - `### Cần cải thiện` (kèm root cause, ≥3 lần "tại sao")
- `## Đề xuất điều chỉnh` (bảng: #, đề xuất, lý do, skill áp dụng, đã apply?)
- `## Lessons` (chỉ có ở mode `closure`)

Khi mode deep ghi review, đồng thời append tóm tắt vào `log/YYYY-MM-DD.md` với link sang file review.

## SOT fields: PROGRESS vs STRUCTURE

`okr-track` chỉ được ghi đè **progress fields**. Mọi thay đổi **structure** delegate sang `okr-init` hoặc `okr-plan`.

### Progress fields (track mode `light` hoặc `deep` được ghi đè)

`.okr/objective.md`:
- Bảng KR: cột `Current`, `Status` (in-progress/achieved/missed)
- Frontmatter `status` (chỉ ở mode `closure`, qua delegate sang `okr-init` để xác nhận)

`.okr/plan.md`:
- Frontmatter counters: `total_actions`, `completed`, `in_progress`, `blocked`
- Frontmatter milestones[].status (pending/in-progress/done)

`.okr/actions/AXXX-*.md`:
- Frontmatter `status`: `pending` | `doing` | `done` | `blocked`

### Structure fields (track CHỈ đề xuất, delegate để apply)

| Field | Skill áp dụng |
|-------|---------------|
| `objective` text, `quarter`, `start_date`, `end_date` | `okr-init` `update-objective` |
| KR target, KR baseline, thêm/xoá KR | `okr-init` `update-objective` |
| `objective.md` frontmatter `status` (active/paused/completed/cancelled) | `okr-init` `update-objective` |
| Resource: người, tool, ngân sách, PIC, khả dụng | `okr-init` `update-resource` |
| Action: title, description, due_date, depends_on, deliverable | `okr-plan` `update` |
| Thêm/xoá action | `okr-plan` `update` |
| Milestone deadline, thêm/xoá milestone | `okr-plan` `update` |

`.okr/resources.md`: TRACK không sửa. Update qua `okr-init` `update-resource`.

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** | `.okr/objective.md`, `plan.md`, `resources.md`, `actions/` | Ghi đè |
| **Log thường** | `.okr/log/YYYY-MM-DD.md` | Append-only |
| **Log review** | `.okr/log/reviews/YYYY-MM-DD.md` | Append-only |
| **Inbox** | `.okr/inbox/*.md` | Status transition (pending → processed/discarded) |

## Inbox Processing

`okr-track` xử lý inbox items khi chạy track (sau update progress).

### Quy trình

1. Đọc tất cả `.okr/inbox/*.md` có `status: pending`
2. Đối chiếu với SOT hiện tại (objective, plan, actions, resources) để gợi ý xử lý
3. Hiển thị bảng tóm tắt + gợi ý cho user chọn
4. Xử lý từng item: delegate hoặc tự apply
5. Đổi status trong file inbox (không xoá file)

### Status transitions

```
pending → processed   (đã xử lý: tạo action, update resource, ghi log...)
pending → discarded   (user quyết định bỏ)
pending → pending     (giữ inbox, chờ rõ hơn)
```

### Delegate rules

| Inbox type | Track tự xử lý? | Delegate sang |
|------------|-----------------|---------------|
| `action` | Không | `okr-plan` mode `update` (tạo action file) |
| `idea` → action | Không | `okr-plan` mode `update` |
| `idea` → giữ | Có (không làm gì) | - |
| `blocker` | Có (sửa action.status = blocked) | - |
| `resource` | Không | `okr-init` mode `update-resource` |
| `note` | Có (append log) | - |

### Gom delegate

Nhiều inbox items cùng delegate sang 1 skill → gom thành 1 lần delegate. Ví dụ: 3 items type=action → 1 lần gọi `okr-plan update` với danh sách 3 actions mới.

### Ghi log

Inbox items đã xử lý ghi vào `log/YYYY-MM-DD.md`:
```
## Inbox processed
- [item title] → [hành động: tạo A014, block A007, ghi log...]
```

