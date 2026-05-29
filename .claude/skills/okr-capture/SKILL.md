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

Xem `references/data-format.md` cho inbox file format.

```yaml
---
type: {type}
status: pending
captured_at: {YYYY-MM-DD}
related_kr: {nếu xác định được}
related_action: {nếu xác định được}
tags: []
---

{Nội dung user nhập}
```

## Quy tắc

- KHÔNG xử lý inbox. Chỉ capture + phân loại.
- Xử lý inbox là việc của okr-track.
