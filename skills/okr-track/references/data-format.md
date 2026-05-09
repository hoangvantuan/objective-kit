# Data Format: log/YYYY-MM-DD.md

Schema cho `.okr/log/YYYY-MM-DD.md`. Log (lịch sử thay đổi), append-only.

```yaml
---
date: YYYY-MM-DD
type: tracking
---
```

Body: `## Thay đổi` (danh sách thay đổi cụ thể), `## Ghi chú`.

## Tham chiếu SOT cập nhật

Khi track, cập nhật các SOT files:

`.okr/objective.md` - bảng KR, cột Current và Status
`.okr/plan.md` - frontmatter:
```yaml
total_actions: int
completed: int
in_progress: int
blocked: int
```

`.okr/actions/AXXX-*.md` - frontmatter `status`:
```yaml
status: pending | doing | done | blocked
```

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** | `.okr/objective.md`, `plan.md`, `actions/`, `resources.md` | Ghi đè |
| **Log** | `.okr/log/YYYY-MM-DD.md` | Append-only |
