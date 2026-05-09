# Data Format: objective.md

File này định nghĩa schema cho `.okr/objective.md`. SOT (Source of Truth), ghi đè khi cập nhật.

## Project type

```yaml
---
type: project
objective: "string"
quarter: "Q1-2026"
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
status: active | completed | paused | cancelled
---
```

Body chứa: `## Objective` (WHY), `## Key Results` (bảng: #, Key Result, Baseline, Target, Current, Status), `## Context`.

KR Status: `pending` | `in-progress` | `achieved` | `missed`

## Habit type

```yaml
---
type: habit
objective: "string"
start_date: YYYY-MM-DD
frequency: daily | weekly | monthly
status: active | paused
---
```

Body chứa: `## Objective` (WHY), `## Key Indicators` (bảng: #, Chỉ số, Tần suất, Streak hiện tại, Streak dài nhất), `## Checklist lặp lại`.
