---
name: okr-capture
description: "Ghi nhanh ý tưởng, note, action, blocker, resource vào inbox."
---

# OKR Capture: Ghi nhanh vào inbox

Chạy inline, thao tác nhẹ: chỉ phân loại + ghi inbox.

## Preload Contract Tier 2

Trước khi phân loại, đảm bảo `.okr/lessons/index.md` (toàn bộ) đã có trong context (`../okr-shared/references/preload.md` Tier 2). Idempotent: qua harness đã có, chạy lẻ tự đọc index nếu thiếu. Dùng để phân loại type bám bài học đã có. KHÔNG nạp objective/plan/resources/actions (capture không suy luận tiến độ).

## Flow

1. Nhận input user
2. Phân loại type: `action` / `blocker` / `resource` / `thought`
3. Tạo file `.okr/inbox/YYYY-MM-DD-HHmm-slug.md`
4. Báo: "Đã ghi vào inbox. Xử lý khi track tiếp."

## Tiêu chí phân loại type

| Type | Khi nào | Tín hiệu nhận biết |
|------|---------|--------------------|
| `action` | Việc cụ thể cần làm, có thể thành task | Động từ hành động + có output ("viết", "gọi", "sửa", "mua") |
| `blocker` | Thứ đang chặn việc khác tiến triển | "không thể", "đang chờ", "bị kẹt", "server down", "chưa có access" |
| `resource` | Tool/tài liệu/ngân sách mới phát hiện | Tên công cụ/tài liệu/link, "dùng được X", "có thư viện Y" |
| `thought` | Ý tưởng/ghi chú chưa rõ scope | Suy nghĩ mở, "hay là", "cân nhắc", chưa rõ thành việc gì |

Không chắc → chọn `thought` (track làm rõ khi xử lý). Phân loại chỉ là gợi ý, `okr-track` verify lại lúc xử lý inbox.

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
