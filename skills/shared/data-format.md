# Data Format: Schema YAML Frontmatter

Mọi file trong `.ok/` dùng markdown + YAML frontmatter. Skill đọc frontmatter để hiểu trạng thái nhanh, đọc body khi cần chi tiết.

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** (Source of Truth) | `.ok/objective.md`, `.ok/plan.md`, `.ok/actions/`, `.ok/resources.md` | Ghi đè khi cập nhật. Luôn phản ánh trạng thái mới nhất |
| **Log** | `.ok/log/YYYY-MM-DD.md`, `.ok/log/reviews/` | Append-only. Ghi lịch sử thay đổi |

Nguyên tắc: SOT = hiện tại. Log = quá khứ.

## objective.md

### Project

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

### Habit

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
due_date: YYYY-MM-DD
depends_on: [A001, A002]
---
```

Body: `## Definition of Done` (checklist), `## Output/Deliverable`, `## Ghi chú`.

## resources.md

```yaml
---
last_updated: YYYY-MM-DD
---
```

Body: `## Vai trò & Trách nhiệm` (bảng: Tên, Vai trò, Trách nhiệm, Actions, Khả dụng), `## Công cụ & Tài liệu`, `## Ngân sách`, `## Thiếu hụt & Rủi ro`.

## log/YYYY-MM-DD.md

```yaml
---
date: YYYY-MM-DD
type: tracking
---
```

Body: `## Thay đổi` (danh sách thay đổi), `## Ghi chú`.

## log/reviews/YYYY-MM-DD.md

```yaml
---
date: YYYY-MM-DD
type: review
period: "YYYY-MM-DD to YYYY-MM-DD"
---
```

Body: `## Tổng kết` (bảng KR), `## Phân tích` (Đạt tốt, Cần cải thiện), `## Điều chỉnh đã áp dụng`.
