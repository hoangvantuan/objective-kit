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

## Archive Rules

### Trigger

Khi `okr-track` (light hoặc deep) đánh `status: done` cho action, **trong cùng lần track, sau phase confirm**.

### Flow

1. Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md` (tạo thư mục `archive/` nếu chưa có).
2. Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`.
3. Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter `plan.md` (với `status: done`).
4. Cập nhật counters frontmatter `plan.md` (`completed` +N).

### Invisible by Default

| Nguồn dữ liệu | Mặc định đọc | Khi nào đọc thêm |
|----------------|-------------|-------------------|
| `actions/*.md` | Có (chỉ active) | Luôn đọc |
| `actions/archive/` | **Không** | User trace hoặc mode closure |
| `log/` | **Không** | User trace theo ngày |
| `log/reviews/` latest | Có | Luôn đọc |
| `log/reviews/` cũ hơn | **Không** | User trace theo ngày |

### Quy tắc theo skill

| Skill | Actions archive | Log cũ |
|-------|----------------|--------|
| Orchestrator `/okr` | Không đọc | Không đọc log. Chỉ latest review |
| `okr-track` light | Không đọc | Chỉ latest log (để so trend) |
| `okr-track` deep | Không đọc | Chỉ latest log + latest review |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả reviews** (tổng kết) |
| `okr-track` trace | **Đọc archive** (lazy) | **Đọc log cũ** (lazy) |
| `okr-plan` update | Không đọc, không sửa | Không đọc |

### Archive file schema

Giống hệt `actions/AXXX-slug.md` (cùng frontmatter + body). Archive files là **read-only**, không bị sửa sau khi archive.

## Log Reading Rules

- Orchestrator `/okr` Bước 1: **KHÔNG đọc `log/`**. Chỉ đọc **1 file mới nhất** trong `log/reviews/`.
- `okr-track` Phase 1: chỉ đọc **1 file mới nhất** trong `log/` và **1 file mới nhất** trong `log/reviews/` (để so trend).
- Log cũ hơn: KHÔNG đọc, trừ khi user yêu cầu trace.
- `okr-track` closure: đọc tất cả `log/reviews/` (cần tổng kết period).

## Trace Flow

Mode `trace` dùng nguyên tắc **lazy loading**: đọc dần, không đọc hết.

### 4 kiểu trace

| Kiểu | Trigger | Đọc gì |
|------|---------|--------|
| Action cụ thể | "trace A003" | `actions/archive/A003-*.md` frontmatter → body khi drill-down |
| Milestone | "trace M1" | `plan.md` frontmatter + lọc `actions/archive/*.md` theo milestone (frontmatter) |
| Theo thời gian | "actions done tháng 4" | Lọc `actions/archive/*.md` theo `due_date` (frontmatter) |
| Log | "xem log tuần trước" | Lọc `log/*.md` hoặc `log/reviews/*.md` theo filename (ngày) |

### Quy trình chung

1. Frontmatter trước (hiển thị tóm tắt)
2. User chọn item cụ thể → đọc body
3. Không đọc toàn bộ archive cùng lúc
