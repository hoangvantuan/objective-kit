# Thuật toán chọn action ưu tiên

Dùng chung cho orchestrator (gợi ý khi mở OKR) và okr-track (đề xuất sau track).

## Input

- Danh sách actions từ `.okr/actions/*.md` (frontmatter: status, due_date, priority, depends_on, key_result)
- KR/KI metrics từ context (on-track, at-risk, gap)
- Tham số caller truyền:
  - `max_items`: số gợi ý tối đa (orchestrator: 2, track: 3)
  - `horizon_days`: cửa sổ deadline (orchestrator: 3, track light: 7, track deep: 7)

## Thuật toán (chọn theo thứ tự, first match, tối đa `max_items`)

1. **Quá hạn** (overdue): `due_date < today` AND status ∈ {doing, blocked, pending}. Ghi số ngày quá hạn. Sắp xếp: quá hạn lâu nhất trước.
2. **Đang block action khác**: action A có ID nằm trong `depends_on` của action B (status B ∈ {pending, blocked}). Ghi rõ: "AXXX đang chặn AYYY, AZZZ". Sắp xếp: block nhiều action nhất trước.
3. **Deadline trong `horizon_days` ngày tới**: `today ≤ due_date ≤ today + horizon_days`. Status ∈ {doing, pending}. Sắp xếp: deadline gần nhất trước.
4. **Priority cao đang doing**: priority ∈ {critical, high} AND status = doing. Sắp xếp: critical trước high.
5. **KR at-risk cần hành động**: action gắn KR có trend at-risk, status ∈ {pending, doing}. Ghi rõ: "KR2 cần +N đơn vị/tháng".

## Format output

Mỗi gợi ý 1 dòng, kèm lý do ≤10 từ:

```
→ A003: Xây MVP (deadline mai, chặn A004 + A005)
→ A007: Phân tích data (quá hạn 3 ngày)
→ A012: Landing page (KR2 at-risk, cần +8/tháng)
```
