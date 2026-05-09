# Data Format: log/reviews/YYYY-MM-DD.md

Schema cho file review session. Lưu trong `.okr/log/reviews/YYYY-MM-DD.md`.

```yaml
---
date: YYYY-MM-DD
type: review
period: "YYYY-MM-DD to YYYY-MM-DD"
---
```

Body chứa các sections:

- `## Tổng kết` (bảng KR: Target, Current, %, Trend)
- `## Phân tích` chia thành `### Đạt tốt` và `### Cần cải thiện` (kèm root cause)
- `## Điều chỉnh đã áp dụng` (danh sách các thay đổi đã đồng ý)

## Tham chiếu SOT đầy đủ

Skill cần đọc và có thể cập nhật:

`.okr/objective.md` - đọc bảng KR + frontmatter
`.okr/plan.md` - đọc milestones, có thể dời deadline
`.okr/actions/*.md` - đọc tất cả frontmatter, có thể sửa status, due_date, pic

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** | `.okr/objective.md`, `plan.md`, `actions/`, `resources.md` | Ghi đè khi áp dụng điều chỉnh |
| **Log review** | `.okr/log/reviews/YYYY-MM-DD.md` | Append-only |
| **Log tracking** | `.okr/log/YYYY-MM-DD.md` | Đọc để phân tích progression |
