# Data Format: Inbox Items

Schema cho `.okr/inbox/*.md`.

## Inbox item file

Tên file: `YYYY-MM-DD-HHmm-slug.md`

Ví dụ: `2026-05-09-1430-viet-unit-test-api.md`, `2026-05-09-1445-server-staging-down.md`

### Frontmatter

```yaml
---
captured_at: "YYYY-MM-DDTHH:mm"
type: action | blocker | resource | thought
title: "string"
description: "string (1-2 câu)"
related_kr: KR1 | null
related_action: A005 | null
status: pending | processed | discarded
---
```

### Body

```markdown
## Mô tả

[Agent viết dựa trên context user cung cấp. Rõ ràng, actionable.]

## Context gốc

> [Nguyên văn user nhập, không chỉnh sửa. Giữ để trace lại ý gốc.]
```

## Status values

| Status | Nghĩa |
|--------|--------|
| `pending` | Chưa xử lý. Mặc định khi capture. |
| `processed` | Đã xử lý bởi `okr-track` (chuyển thành action, ghi log, update resource...) |
| `discarded` | User quyết định bỏ (không cần thiết, duplicate...) |

## Xử lý inbox

`okr-capture` chỉ tạo file. Xử lý là trách nhiệm `okr-track`. Mapping chi tiết (type → delegate): xem `okr-shared/references/schemas.md` section "Inbox type → Delegate mapping".

## Inbox Aging

Inbox Aging canonical: xem skill `okr-shared/references/schemas.md` section "Inbox Aging" (load từ orchestrator).

## Quy tắc

- Mỗi item = 1 file riêng (dễ xử lý từng cái).
- `captured_at` lấy từ thời điểm user gọi capture, KHÔNG phải thời điểm ghi file.
- Slug trong tên file: tiếng Việt không dấu hoặc tiếng Anh, tối đa 5 từ, dấu gạch ngang.
- `related_kr` và `related_action` là gợi ý của agent, có thể sai. `okr-track` sẽ verify khi xử lý.
