# Data Format: log/

Skill `okr-track` ghi log vào `log/YYYY-MM-DD.md`. Phân biệt loại entry bằng frontmatter `type` (array).

## Schema file log: log/YYYY-MM-DD.md

### Frontmatter

~~~yaml
---
date: YYYY-MM-DD
type: [tracking]          # array, giá trị hợp lệ: tracking | review | closure
---
~~~

`type` là array vì cùng ngày có thể có light sáng (tracking) + deep chiều (review) → `type: [tracking, review]`.

### Body theo type

**tracking** (từ light hoặc deep Bước 1 update progress):

~~~markdown
## Thay đổi
- KR1.current: 40 > 50
- A003.status: doing > done

## Ghi chú
- A005 vẫn blocked
~~~

**review** (từ deep): gồm cả `## Thay đổi` (nếu deep có update progress) + phân tích.

~~~markdown
## Thay đổi
- KR1.current: 40 > 50

## Tổng kết
(bảng KR: Target, Current, %, Trend)

## Phân tích
### Đạt tốt
### Cần cải thiện (root cause ≥3 lần "tại sao")

## Đề xuất điều chỉnh
(bảng: #, đề xuất, lý do, skill áp dụng, đã apply?)
~~~

**closure** (từ closure): giống review + thêm section `## Lessons`.

~~~markdown
## Thay đổi
...
## Tổng kết
...
## Phân tích
...
## Đề xuất điều chỉnh
...
## Lessons
(chỉ closure mới có section này)
~~~

### Append rule

- Cùng ngày, cùng type: append section mới `## Thay đổi (lần N, HH:MM)`.
- Cùng ngày, khác type (light sáng + deep chiều): append deep sections vào file đã có, update frontmatter `type` thành union (vd `[tracking]` → `[tracking, review]`).

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
- Frontmatter `status`: `pending` | `doing` | `done` | `blocked` (cũng được sửa bởi `okr-plan` `update`)

### Structure fields (track CHỈ đề xuất, delegate để apply)

> Bảng canonical ở `okr/references/sot-ownership.md`. Bảng dưới là **chi tiết hoá theo field cụ thể** trong `objective.md` / `plan.md` / `resources.md`, dùng để track lookup khi delegate.

| Field | Skill áp dụng |
|-------|---------------|
| `objective` text, `quarter`, `start_date`, `end_date` | `okr-init` `update-objective` |
| KR target, KR baseline, thêm/xoá KR | `okr-init` `update-objective` |
| `objective.md` frontmatter `status` (active/paused/completed/cancelled) | `okr-init` `update-objective` |
| Resource: Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Action: title, description, due_date, depends_on, deliverable | `okr-plan` `update` |
| Thêm/xoá action | `okr-plan` `update` |
| Milestone deadline, thêm/xoá milestone | `okr-plan` `update` |

`.okr/resources.md`: TRACK không sửa. Update qua `okr-init` `update-resource`.

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** | `.okr/objective.md`, `plan.md`, `resources.md`, `actions/` | Ghi đè |
| **Log thường** | `.okr/log/YYYY-MM-DD.md` | Append-only |
| **Inbox** | `.okr/inbox/*.md` | Status transition (pending → processed/discarded) |

## Inbox Processing

> Đã chuyển sang shared-schemas.md (đã có trong context từ orchestrator okr).

## Archive Rules

> Đã chuyển sang shared-schemas.md (đã có trong context từ orchestrator okr).

## Log Reading Rules

| Mode                | Đọc log                 |
| ------------------- | ----------------------- |
| Orchestrator `/okr` | Không đọc               |
| Light               | Không đọc               |
| Deep                | Adaptive (xem bên dưới) |
| Closure             | Tất cả files            |
| Trace               | Lazy theo yêu cầu user  |

**Deep adaptive rule:**

1. Scan `log/` sorted desc by filename (date).
2. Tìm file deep gần nhất = file có `review` hoặc `closure` trong `type` array.
3. **Nếu file mới nhất đã là deep**: đọc 3 file deep gần nhất (để nắm xu hướng).
4. **Nếu file mới nhất là light**: đọc tất cả light từ deep cuối đến nay + 3 file deep gần nhất.

Mục đích: deep luôn nắm (a) xu hướng dài hạn qua 3 review trước, (b) mọi thay đổi nhỏ kể từ review trước.

**Light không đọc log**: light là quick update, không cần historical context. SOT (objective.md, plan.md, actions/) đã đủ.

## External Sync (optional)

Sync 2 chiều giữa OKR action files và tool bên thứ 3 (Things 3, Notion, Jira...). Chỉ chạy nếu action có field `external_ids` (xem `skills/okr-plan/references/data-format.md`).

### Pull (trước tracking, Phase 4a/4b)

1. Đọc tất cả active action files có `external_ids` trong frontmatter.
2. Với mỗi tool trong `external_ids`:
   a. Tra `resources.md` `## Công cụ` → tìm cột "Cách dùng" để biết integration method (`skill: <name>`, `mcp: <name>`).
   b. Gọi skill/MCP tương ứng → lấy status task bên ngoài.
3. So sánh status:
   - Khác biệt → hiển thị: "Tool X: A001 = completed, OKR: A001 = doing. Đồng bộ? (y/n)"
   - User confirm → update action status trong OKR (progress field, track tự xử lý).
4. Tool không có skill/MCP hoặc cột "Cách dùng" rỗng → skip, log cảnh báo.

### Push (sau tracking, Phase 4a/4b)

1. Với mỗi action vừa thay đổi status + có `external_ids`:
   a. Tra integration method từ `resources.md`.
   b. Gọi skill/MCP → push status mới lên tool ngoài.
2. Báo kết quả: "Đã sync A001 → Things 3 (completed), A002 → Notion (doing)"
3. Push thất bại → log lỗi, không retry tự động.

### Mapping status

| OKR Status | Hướng map chung |
|------------|----------------|
| pending | Tool-specific "todo" / "not started" |
| doing | Tool-specific "in progress" |
| done | Tool-specific "completed" / "done" |
| blocked | Tool-specific "on hold" / "blocked" (nếu tool hỗ trợ) |

Mapping cụ thể theo từng tool nằm trong config của skill/MCP tương ứng. OKR system không hardcode.

### Nguyên tắc sync

- Pull: mọi thay đổi từ tool ngoài cần user confirm trước khi ghi vào OKR.
- Push: tự động sau confirm tracking (không hỏi thêm).
- Tool không có integration → skip, không lỗi.
- Sync là optional step, không block tracking flow nếu thất bại.
- Archive: action đã archive không sync nữa (read-only).

## Trace Flow

Mode `trace` dùng nguyên tắc **lazy loading**: đọc dần, không đọc hết.

### 4 kiểu trace

| Kiểu | Trigger | Đọc gì |
|------|---------|--------|
| Action cụ thể | "trace A003" | `actions/archive/A003-*.md` frontmatter → body khi drill-down |
| Milestone | "trace M1" | `plan.md` frontmatter + lọc `actions/archive/*.md` theo milestone (frontmatter) |
| Theo thời gian | "actions done tháng 4" | Lọc `actions/archive/*.md` theo `due_date` (frontmatter) |
| Log | "xem log tuần trước" | Lọc `log/*.md` theo filename (ngày), filter theo `type` nếu user chỉ cần review |

### Quy trình chung

1. Frontmatter trước (hiển thị tóm tắt)
2. User chọn item cụ thể → đọc body
3. Không đọc toàn bộ archive cùng lúc
