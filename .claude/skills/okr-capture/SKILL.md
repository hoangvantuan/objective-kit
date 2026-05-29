---
name: okr-capture
description: "Ghi nhanh ý tưởng, ghi chú, action, blocker, resource vào inbox. Load khi cần capture/ghi nhanh/note."
---

# OKR Capture: Ghi nhanh vào inbox

Chạy inline, thao tác nhẹ: chỉ phân loại + ghi inbox.

## Flow

1. Nhận input user
2. Phân loại type: `action` / `blocker` / `resource` / `thought`
3. Tạo file `.okr/inbox/YYYY-MM-DD-HHmm-slug.md`
4. Báo: "Đã ghi vào inbox. Xử lý khi track tiếp."

## Schema

Canonical đầy đủ ở `references/data-format.md`. Frontmatter (khớp canonical):

```yaml
---
captured_at: "YYYY-MM-DDTHH:mm"   # thời điểm user gọi capture (có giờ), không phải lúc ghi file
type: action | blocker | resource | thought
title: "string"
description: "string (1-2 câu)"
related_kr: KR1 | null            # gợi ý, track verify lại
related_action: A005 | null       # gợi ý, track verify lại
status: pending
---
```

Body: `## Mô tả` (agent viết, actionable) + `## Context gốc` (nguyên văn user, để trace). Chi tiết: `references/data-format.md`.

## Quy tắc

- KHÔNG xử lý inbox. Chỉ capture + phân loại.
- Xử lý inbox là việc của okr-track.
