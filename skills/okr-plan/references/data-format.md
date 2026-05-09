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

Nếu type=ongoing, plan.md body chứa `## Practices` (hành động lặp lại). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI cụ thể (vd: "Mua đồ tập gym"). Actions tuân quy tắc bình thường. Milestones không bắt buộc với Ongoing.
