---
name: ok-review
description: "Lookback & điều chỉnh dự án. Phân tích tiến độ tổng thể, tìm root cause trễ, đề xuất điều chỉnh plan. Trigger: /ok-review"
---

# ok-review: Lookback & điều chỉnh

Đọc SOT + log, phân tích toàn diện, đề xuất và áp dụng điều chỉnh.

## Điều kiện tiên quyết

- `.ok/objective.md`, `.ok/plan.md`, `.ok/actions/` phải tồn tại.

## Checklist

1. Đọc `.ok/objective.md` + `.ok/plan.md` (SOT: trạng thái hiện tại)
2. Đọc `.ok/log/*.md` (Log: lịch sử thay đổi theo thời gian)
3. Tính tổng kết:
   - Mỗi KR: Target vs Current, %, trend
   - Timeline: % thời gian đã dùng vs % tiến độ thực
4. Phân tích (đọc `../shared/data-format.md` phần `log/reviews/`):
   - Cái gì đạt tốt, tại sao
   - Cái gì trượt, root cause (hỏi "tại sao?" ít nhất 3 lần, không dừng ở triệu chứng)
   - Actions blocked lâu nhất, nguyên nhân
   - Tốc độ hoàn thành actions: nhanh hơn hay chậm hơn kế hoạch
5. Đề xuất điều chỉnh cụ thể:
   - Thay đổi target KR (tăng/giảm)
   - Thêm/bớt/sửa actions
   - Dời deadline milestones
   - Thay đổi PIC, thêm resource
6. Hỏi user từng đề xuất (đồng ý/từ chối/sửa)
7. Áp dụng điều chỉnh: cập nhật SOT files
8. Tạo thư mục `.ok/log/reviews/` nếu chưa có
9. Ghi log vào `.ok/log/reviews/YYYY-MM-DD.md`

## Format review

Hiển thị trước khi đề xuất:

```
Review dự án: [Tên Objective]
Period: YYYY-MM-DD > YYYY-MM-DD

Tổng kết KR:
| KR | Target | Current | % | Trend |
|----|--------|---------|---|-------|
| KR1 | 100 | 45 | 45% | on-track |
| KR2 | 50 | 10 | 20% | at-risk |

Timeline: 60% thời gian đã dùng | 35% tiến độ trung bình

Đạt tốt:
  - [Phân tích]

Cần cải thiện:
  - [Phân tích + root cause]
```

## Quy tắc

- Phân tích root cause, không dừng ở triệu chứng
- Mỗi đề xuất điều chỉnh phải kèm lý do
- Hỏi user confirm từng điều chỉnh trước khi sửa SOT
- Log review ghi đầy đủ: tổng kết, phân tích, điều chỉnh đã áp dụng
- Nếu objective type=habit: review streak, adherence rate, đề xuất điều chỉnh frequency hoặc checklist
