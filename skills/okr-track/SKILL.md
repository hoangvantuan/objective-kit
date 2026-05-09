---
name: okr-track
description: "Theo dõi tiến độ dự án. Cập nhật Key Results, actions, phát hiện trễ và blockers. Trigger: /okr-track, hoặc tự động khi /okr phát hiện dự án đang thực thi"
---

# okr-track: Theo dõi tiến độ

Đọc SOT files, nhận update từ user, cập nhật SOT + ghi log.

## Điều kiện tiên quyết

- `.okr/objective.md` và `.okr/plan.md` phải tồn tại.

## Checklist

1. Đọc `.okr/objective.md` (KR hiện tại)
2. Đọc frontmatter tất cả `.okr/actions/*.md` (status)
3. Hiển thị dashboard ngắn:
   - Mỗi KR: Current/Target, %, trend
   - Actions: tổng, done, doing, blocked
   - Highlight: actions chậm deadline, blocked
4. Hỏi user có update gì mới (tiến độ, hoàn thành task, blockers)
5. Cập nhật SOT files (đọc `references/metrics.md` cho cách tính)
6. Tạo thư mục `.okr/log/` nếu chưa có
7. Ghi entry vào `.okr/log/YYYY-MM-DD.md`

## Dashboard format

Hiển thị khi bắt đầu track:

```
Tiến độ dự án: [Tên Objective]

Key Results:
  KR1: ████░░░░░░ 40/100 (40%) > on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending

Cần chú ý:
  - A005 blocked: [lý do]
  - A007 quá hạn 3 ngày
```

## Quy tắc

- Luôn hiển thị dashboard trước khi hỏi update
- Sau khi nhận update, cập nhật SOT files ngay (ghi đè giá trị mới)
- Ghi log mọi thay đổi vào `.okr/log/YYYY-MM-DD.md`
- Nếu log ngày hôm nay đã có, append section mới (không ghi đè log)
- Với habit type: hiển thị streak thay vì %, checklist thay vì actions
