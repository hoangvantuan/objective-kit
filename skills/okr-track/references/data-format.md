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
