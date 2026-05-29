# Inbox Processing Flow

Chạy sau khi update progress (light) hoặc sau delegate (deep). Cũng có thể chạy độc lập nếu user gọi `/okr track` chỉ để xử lý inbox.

## Bước 1: Đọc inbox + compute staleness

Đọc tất cả `.okr/inbox/*.md` có `status: pending`. Nếu không có → skip toàn bộ Phase 5.

Với mỗi item, compute `staleness_days = today - captured_at` (xem Inbox Aging đã có trong context từ orchestrator). Phân loại:

- Mới (≤7 ngày): xử lý bình thường ở Bước 2.
- Đang chờ (7 < staleness ≤ 30): xử lý bình thường, nhưng sort lên đầu trong bảng Bước 2.
- Cũ (>30 ngày): chuyển sang Bước 1.5 xử lý riêng trước.

## Bước 1.5: Cảnh báo stale items (chỉ chạy nếu có items >30 ngày)

Hiển thị block riêng TRƯỚC bảng Bước 2:

```
⚠️ Inbox cũ ≥30 ngày (3 items)
| # | Type     | Title                        | Captured   | Staleness |
|---|----------|------------------------------|------------|-----------|
| 1 | thought  | Thử framework X cho frontend | 2026-04-01 | 40 ngày   |
| 2 | action   | Viết blog post về Y          | 2026-03-25 | 47 ngày   |
| 3 | resource | Library Z hỗ trợ chart       | 2026-03-15 | 57 ngày   |

Còn relevant không? (vd: "1 giữ, 2 bỏ, 3 giữ" / all giữ / all bỏ)
```

User trả lời:

- `<N> giữ`: giữ `status: pending` (không thay đổi). Item đi tiếp vào Bước 2 xử lý bình thường.
- `<N> bỏ`: đổi `status: discarded` ngay (không cần qua Bước 2-3).
- `all giữ`: giữ tất cả, đẩy hết sang Bước 2.
- `all bỏ`: discard tất cả.

KHÔNG auto-discard. User quyết định cuối.

Sau Bước 1.5, đi tiếp Bước 2 với items đã filter (đã giữ + items ≤30 ngày).

## Bước 2: Hiển thị inbox + gợi ý xử lý

Agent đọc context từng item, đối chiếu với SOT hiện tại (objective, plan, actions, resources), rồi gợi ý cách xử lý:

```
Inbox: 3 items chưa xử lý
| # | Type     | Title                         | Captured  | Gợi ý xử lý                    |
|---|----------|-------------------------------|-----------|----------------------------------|
| 1 | action   | Viết unit test cho API auth   | 05-09     | → Tạo A014, gán M2, due 05-15   |
| 2 | thought  | Thử framework X cho frontend  | 05-08     | → Giữ inbox (chưa rõ scope)     |
| 3 | blocker  | Server staging down           | 05-09     | → Block A007, ghi log            |

Xử lý items nào? (1,3 / all / skip / bỏ N)
```

Gợi ý xử lý dựa trên:

- `action`: map vào KR/milestone nào, capacity còn lại bao nhiêu, deadline nào hợp lý
- `blocker`: action nào bị ảnh hưởng?
- `resource`: resources.md cần thêm gì (Công cụ / Tài liệu)?
- `thought`: đủ rõ để thành action chưa? Nếu rõ → tạo action; nếu chưa rõ → giữ inbox; nếu chỉ là ghi chú → append log ngày

## Bước 3: Xử lý từng item user chọn

**Step 3.0: Validate related ID (M5b).** Trước khi xử lý từng item user chọn, agent validate field `related_kr`/`related_action`/`related_tool` (do capture ghi vào, không guarantee đúng):

| Field            | Quy tắc validate                                                                                          | Hành vi nếu sai                                                                                            |
| ---------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `related_kr`     | Phải khớp KR hoặc KI ID trong `objective.md` (vd `KR1`, `KI2`).                                           | Tìm KR/KI có title match fuzzy với context item → đề xuất ID đúng. Nếu không tìm được → set `null`, hỏi user. |
| `related_action` | Phải khớp action ID trong `actions/*.md` (vd `A003`). KHÔNG khớp `actions/archive/*.md` (action đã done). | Action đã archive → cảnh báo "Action X đã done/archive. Bỏ link?". Không tồn tại → set `null`, hỏi user.   |


Hiển thị cho user khi có sai sót:

```
Validate inbox related fields
  Item #1 (action "Viết unit test cho API auth"): related_kr=KR9 → KR9 không tồn tại.
    Đề xuất: KR2 (code coverage) hoặc null.
  Item #3 (blocker "Server staging down"): related_action=A100 → A100 không tồn tại.
    Đề xuất: A007 (deploy staging) match fuzzy 78%, hoặc null.

Áp dụng đề xuất? (y / sửa N / null tất / huỷ)
```

User trả lời:

- `y` → áp dụng tất cả đề xuất, ghi đè frontmatter inbox file (chỉ field `related_*`, không đổi `status`).
- `sửa N` → user nói ID đúng cho item N.
- `null tất` → set tất cả related sai thành `null`.
- `huỷ` → giữ nguyên related sai, đi tiếp Bước 3 xử lý (sẽ skip mapping liên quan).

Sau validate, đi tiếp xử lý từng item theo bảng:

Xử lý từng item theo bảng "Inbox type → Delegate mapping" (xem `okr/references/shared-schemas.md`). Migrate dữ liệu cũ (`idea`/`note` → `thought`) cũng theo quy tắc ở shared-schemas.


Với mỗi item xử lý xong → đổi `status: processed` trong file inbox.

User chọn "bỏ N" → đổi `status: discarded`.

Giữ inbox (thought chưa rõ) → giữ `status: pending`, không làm gì.

## Bước 4: Gom delegate

Nếu nhiều items cùng delegate sang 1 skill (vd: 3 actions mới cùng sang `okr-plan update`) → gom thành 1 lần delegate. Skill được delegate sẽ tự chạy phase confirm + ghi file.

## Bước 5: Báo cáo

```
Inbox đã xử lý: 2/3 items
  - #1 → A014 (tạo mới, gán M2)
  - #3 → A007 blocked (ghi log)
  - #2 → giữ inbox (chờ rõ hơn)
Inbox còn lại: 1 item pending
```
