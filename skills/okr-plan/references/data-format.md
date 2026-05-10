# Data Format: plan.md + actions/

Schema cho `.okr/plan.md` và `.okr/actions/*.md`. SOT (Source of Truth), ghi đè khi cập nhật.

## plan.md

```yaml
---
total_actions: int
completed: int
in_progress: int
blocked: int
milestones:
  - name: "string"
    target_date: YYYY-MM-DD
    key_results: [KR1, KR2]
    status: pending | in-progress | done
---
```

Body: `## Roadmap` với các heading milestone, mỗi milestone liệt kê actions.

## actions/AXXX-slug.md

```yaml
---
id: AXXX
title: "string"
description: "string"
key_result: KR1
milestone: "string"
status: pending | doing | done | blocked
priority: critical | high | medium | low
effort: xs | s | m | l | xl
pic: "string"
verifier: "string (người hoặc cơ chế verify output)"
due_date: YYYY-MM-DD
depends_on: [A001, A002]
---
```

Body: `## Definition of Done` (checklist), `## Output/Deliverable`, `## Tiêu chí chất lượng`, `## Ghi chú`.

## Tham chiếu objective.md

Skill đọc `.okr/objective.md` để biết Key Results. Frontmatter quan trọng:

```yaml
type: project | ongoing
objective: "string"
```

Nếu type=ongoing, plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Schema xem section "## Practices schema" bên dưới. Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI cụ thể (vd: "Mua đồ tập gym"). Actions tuân quy tắc bình thường. Milestones không bắt buộc với Ongoing.

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
| `current_streak` | int | Số chu kỳ liên tiếp gần nhất đã đạt `target_count` |
| `description` | string | Mô tả ngắn cách thực hiện |
| `ki_link` | `KI<N>` | KI mà practice này phục vụ (vd: `KI1`, `KI2`), link tới Key Indicator trong `objective.md` |

Quy tắc:
- ID practice (`P1`, `P2`...) duy nhất trong file. Không reuse khi xoá.
- 1 practice link tới đúng 1 KI. 1 KI có thể có nhiều practices.
- `current_streak` được update bởi `okr-track` Ongoing flow (Phase 4a Ongoing): hỏi user "Tuần này đạt practice nào?" → tăng/reset streak. **TODO (wave 3, item G2/M9)**: bước này chưa implement trong `skills/okr-track/SKILL.md` Phase 4a, sẽ thêm ở wave 3.
- `okr-plan` mode `update` thêm/sửa/xoá Practices structure (frequency, target_count, description, ki_link). KHÔNG sửa `current_streak`.

## actions/archive/

Chứa action files đã hoàn thành (`status: done`), được dời từ `actions/` bởi `okr-track` sau phase confirm.

- **Schema**: giống hệt `actions/AXXX-slug.md` (cùng frontmatter + body).
- **Read-only**: archive files KHÔNG bị sửa sau khi archive. Không ai ghi đè, không ai thêm field.
- **Khi nào đọc**: chỉ khi `okr-track` mode `closure` (tổng kết) hoặc mode `trace` (user yêu cầu xem lại).
- `okr-plan` KHÔNG đọc, KHÔNG sửa, KHÔNG tạo file trong `actions/archive/`.
