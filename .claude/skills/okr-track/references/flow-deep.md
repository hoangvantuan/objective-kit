# Mode DEEP (review sâu + delegate điều chỉnh cấu trúc)

Phạm vi: phân tích root cause + ĐỀ XUẤT điều chỉnh. KHÔNG tự sửa cấu trúc. Việc apply đẩy sang `okr-init` hoặc `okr-plan`.

## Bước 1: Update progress nếu cần

Hỏi user có update progress nào trước phân tích (giống light). Nếu có → áp dụng + log + **nhắc output cho actions done** (step 4 light) + **archive actions done** như light (Phase 4a bước 5). Bao gồm sync pull/push (step 0 + 3b) và re-render Roadmap (step 5) như light.

## Bước 2: Phân tích root cause

Cho mỗi vấn đề (KR at-risk, KI critical/warning, blocker, quá hạn):

- Hỏi "tại sao?" tối thiểu 3 lần.
- Phân biệt nhân (gốc) vs duyên (điều kiện).
- Tách triệu chứng vs nguyên nhân.

Hiển thị:

```
Phân tích
## Đạt tốt
  - KR3 vượt kế hoạch vì [lý do]
## Cần cải thiện
  - KR2 trễ 15%: root cause là [...] (không phải triệu chứng [...])
  - A005 blocked 5 ngày: nguyên nhân là dependency [...]
```

## Bước 3: Đề xuất điều chỉnh cấu trúc (kèm phân loại delegate)

Mỗi đề xuất gắn nhãn skill sẽ áp dụng:

```
Đề xuất điều chỉnh
| # | Đề xuất                            | Lý do            | Áp dụng qua            |
|---|------------------------------------|------------------|------------------------|
| 1 | Giảm KR2 target: 50 > 35           | Market shift     | okr-init update-objective |
| 2 | Thêm action A013 "Tăng marketing"  | Thiếu đẩy KR2    | okr-plan update        |
| 3 | Dời M2: 2026-11-15 > 2026-11-30    | A005 chậm        | okr-plan update        |
| 4 | Tăng capacity: 15h > 20h/tuần       | Scope tăng       | okr-init update-resource|
| 5 | Thêm tool Mailchimp cho A013         | Cần email blast   | okr-init update-resource|
```

## Bước 4: All-changes confirm + chọn áp dụng

Bước 1: User chọn cái nào áp dụng:

```
Đồng ý đề xuất nào? (vd: 1,3,5 / không / sửa N: <new value>)
```

Bước 2: Sau khi user chọn (vd `1,3,5`), gom thay đổi theo skill đích + render **all-changes diff** trước khi delegate. Bảng phải gom theo skill (nhóm các change cùng skill vào 1 block), để user thấy tổng quan + biết init/plan sẽ áp dụng gì:

```
All-changes preview (gom theo skill đích)

→ okr-init update-objective
  - KR2.target: 50 > 35  (lý do: market shift Q4)

→ okr-plan update
  - Thêm A013 "Tăng marketing"  (gắn KR2, effort m)
  - M2.deadline: 2026-11-15 > 2026-11-30  (A005 chậm)

→ okr-init update-resource
  - tool: thêm Mailchimp (cho A013 email campaign)

Confirm tất cả? (y / sửa N / huỷ N / huỷ tất)
```

User trả lời:
- `y` → đi tiếp Bước 5 với `pre_confirmed: true` cho mỗi delegate payload. Init/plan nhận signal sẽ SKIP phase confirm riêng (xem `okr-init` Phase 6 + `okr-plan` Phase 4).
- `sửa N` → quay lại Bước 3 sửa đề xuất số N, xong render lại all-changes preview.
- `huỷ N` → loại N khỏi danh sách áp dụng, render lại preview.
- `huỷ tất` → quay lại Bước 4 lựa chọn ban đầu.

## Bước 5: Delegate sang skill phù hợp

Gom đề xuất user đồng ý theo skill target. Mỗi delegate KÈM payload format:

```yaml
delegate_to: okr-init update-objective | okr-init update-resource | okr-plan update
context:
  changes:
    - field: <vd: KR2.target | A005.due_date | tool.X>
      from: <giá trị cũ>
      to: <giá trị mới>
    - field: ...
      from: ...
      to: ...
  reason: "<1-2 câu giải thích vì sao điều chỉnh, lấy từ Bước 2 root cause>"
  source_review: log/YYYY-MM-DD.md
  pre_confirmed: true
```

Field meanings:

| Field | Bắt buộc | Mô tả |
|-------|----------|-------|
| `delegate_to` | có | Skill + mode đích. 1 trong 3 giá trị. |
| `context.changes[]` | có | Danh sách thay đổi (field, from, to). 1 payload có thể chứa nhiều changes nếu cùng skill đích. |
| `context.reason` | có | Lý do GỐC từ Bước 2 root cause. Track viết vào, init/plan đọc + hiển thị trong CONFIRM diff. |
| `context.source_review` | có | Path file review log để init/plan trace lại. Mặc định `log/<today>.md`. |
| `context.pre_confirmed` | có | `true` sau khi user reply `y` cho all-changes preview ở Bước 4 (track đã gom + show full diff trước). Skill nhận BẮT BUỘC honor: skip ask "y/sửa/huỷ" ở phase confirm riêng, đi thẳng ghi file. Vẫn hiển thị diff + reason để trace. |

Ví dụ payload thực tế (KR2 giảm target do market shift):

```yaml
delegate_to: okr-init update-objective
context:
  changes:
    - field: KR2.target
      from: 50
      to: 35
  reason: "Market shift Q4 (từ root cause Bước 2): tăng trưởng ngành chậm 30%, target 50 không khả thi."
  source_review: log/2026-12-01.md
  pre_confirmed: true
```

Skill được delegate sẽ tự chạy phase confirm + ghi file (theo flow của riêng nó). CONFIRM phase BẮT BUỘC hiển thị `reason` cùng diff (xem `okr-init` Phase 6 update-objective + `okr-plan` Phase 4 update). Track CHỈ truyền context, KHÔNG tự ghi SOT objective/plan.

## Bước 6: Xử lý inbox (nếu có items pending)

Chạy Inbox Processing Flow (xem Phase 5).

## Bước 7: Ghi log review + update SOT timestamps

Sau khi tất cả delegate + inbox processing hoàn tất:

1. Ghi log vào `.okr/log/YYYY-MM-DD.md`:
  - Nếu file ngày đã có (từ Bước 1 update progress): append sections review, update frontmatter `type` thành union (vd `[tracking]` → `[tracking, review]`).
  - Nếu file ngày chưa có: tạo mới với `type: [review]`.
  - Nội dung review: Tổng kết KR/KI, Phân tích root cause, Đề xuất + cái nào đã apply, Inbox items đã xử lý.

2. Update `plan.md` frontmatter: `last_review_date: today`, `last_track_date: today`.

## Bước 8: Đề xuất next action

Chọn theo thuật toán [action-priority.md](../../okr-shared/references/action-priority.md) với `max_items=3`, `horizon_days=7`.

Khác với light: sau khi đã phân tích root cause ở Bước 2, mỗi gợi ý kèm insight từ phân tích (không chỉ lý do ngắn, mà nối với root cause đã tìm).

```
Việc tiếp theo
  → A003: Xây MVP (deadline mai, chặn A004 + A005. Root cause: scope creep từ sprint trước)
  → A007: Phân tích data (quá hạn 3 ngày. Nguyên nhân: thiếu access data warehouse)
  → A012: Landing page (KR2 at-risk, cần +8/tháng. Đã đề xuất thêm tool Mailchimp ở Bước 3)
```
