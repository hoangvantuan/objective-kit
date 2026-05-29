# Data Format: plan.md + actions/

Schema cho `.okr/plan.md` và `.okr/actions/*.md`. SOT (Source of Truth), ghi đè khi cập nhật.

## plan.md

```yaml
---
total_actions: int
completed: int
in_progress: int
blocked: int
last_track_date: YYYY-MM-DD    # okr-track cập nhật mỗi lần light/deep/closure
last_review_date: YYYY-MM-DD   # okr-track cập nhật khi deep/closure
milestones:
  - name: "string"
    target_date: YYYY-MM-DD
    key_results: [KR1, KR2]
    status: pending | in-progress | done
---
```

Body: `## Roadmap` với bảng action per-milestone. Bảng auto-generated từ action files, là view (không sửa trực tiếp). SOT là action files.

### Roadmap format

Roadmap format canonical: xem skill `okr-shared/references/schemas.md` section "Roadmap format" (load từ orchestrator).

## actions/AXXX-slug.md

```yaml
---
id: AXXX
title: "string"
description: "string"
key_result: KR1  # Ongoing: ghi KI ID (vd KI1)
milestone: "string"
status: pending | doing | done | blocked
priority: critical | high | medium | low
effort: xs | s | m | l | xl
pic: "string"
due_date: YYYY-MM-DD
depends_on: [A001, A002]
notes: "string (optional, ≤50 ký tự)"
external_ids:              # optional
  tool_key: "external_id_string"
---
```

Field mới (optional):

| Field | Type | Mô tả |
|-------|------|-------|
| `notes` | string, optional | Ghi chú ngắn hiển thị trong bảng Roadmap. Khuyến nghị ≤50 ký tự. Vd: "Cần xong trước A003", "Blocked by API access". Khác với `## Ghi chú` body (chi tiết dài). Không có thì cell trống. |
| `external_ids` | YAML map, optional | Map tool_key → external ID string. Key lowercase, match tên tool trong resources.md (vd: `things3`, `notion`). Nhiều tool được. Dùng cho sync 2 chiều với tool bên thứ 3. |

Ví dụ `external_ids`:

```yaml
external_ids:
  things3: "ABC-123"
  notion: "page-xyz-456"
```

Body: `## Definition of Done` (checklist), `## Output/Deliverable`, `## Tiêu chí chất lượng`, `## Ghi chú`.

## Tham chiếu objective.md

Skill đọc `.okr/objective.md` để biết Key Results. Frontmatter quan trọng:

```yaml
type: project | ongoing
objective: "string"
```

Nếu type=ongoing, plan.md body chứa `## Practices` (schema xem section "## Practices schema" bên dưới). Ongoing + actions: xem `action-guide.md` section "Ongoing type".

## Practices schema (chỉ Ongoing)

Practices là hành động lặp lại theo chu kỳ để duy trì KI healthy. Nằm trong body `plan.md` dưới heading `## Practices`. KHÔNG nằm ở `objective.md`. KHÔNG có frontmatter riêng (đọc từ markdown body).

Mỗi practice là 1 sub-heading `### PN: <Tên>` với các field bullet:

```markdown
## Practices

### P1: Tập thể dục
- frequency: weekly
- target_count: 3
- current_streak: 2
- description: Cardio + strength, mỗi buổi 45 phút
- ki_link: KI1

### P2: Ngủ đủ giấc
- frequency: daily
- target_count: 7
- current_streak: 5
- description: Đi ngủ trước 23:00, ngủ ≥7 giờ
- ki_link: KI2
```

Field meanings:

| Field | Type | Mô tả |
|-------|------|-------|
| `frequency` | `daily` \| `weekly` \| `biweekly` \| `monthly` | Chu kỳ practice |
| `target_count` | int | Số lần mục tiêu trong 1 chu kỳ (vd: 3 lần/tuần) |
| `current_streak` | int | Số chu kỳ liên tiếp gần nhất đã đạt `target_count`. Mặc định `0` khi tạo mới. |
| `description` | string | Mô tả ngắn cách thực hiện |
| `ki_link` | `KI<N>` | KI mà practice này phục vụ (vd: `KI1`, `KI2`), link tới Key Indicator trong `objective.md` |

Quy tắc:
- ID practice (`P1`, `P2`...) duy nhất trong file. Không reuse khi xoá.
- 1 practice link tới đúng 1 KI. 1 KI có thể có nhiều practices.
- `current_streak` được update bởi `okr-track` Ongoing flow (Phase 4a Ongoing step 4 "Update practice streak"): với mỗi practice, agent hỏi "Tuần vừa rồi đạt target chưa? (y/n/skip)". `y` → +1, `n` → reset 0, `skip` → giữ nguyên. Track ghi đè `current_streak` trong `plan.md` body khi user confirm.
- `okr-plan` mode `update` thêm/sửa/xoá Practices structure (frequency, target_count, description, ki_link). KHÔNG sửa `current_streak`.

## actions/archive/

Chứa action files đã hoàn thành (`status: done`), được dời từ `actions/` bởi `okr-track` sau phase confirm.

- **Schema**: giống hệt `actions/AXXX-slug.md` (cùng frontmatter + body).
- **Read-only**: archive files KHÔNG bị sửa sau khi archive. Không ai ghi đè, không ai thêm field.
- **Khi nào đọc**: chỉ khi `okr-track` mode `closure` (tổng kết) hoặc mode `trace` (user yêu cầu xem lại).
- `okr-plan` KHÔNG đọc, KHÔNG sửa, KHÔNG tạo file trong `actions/archive/`.
