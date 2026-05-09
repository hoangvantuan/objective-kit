# Metrics: Cách tính và log format

## Tiến độ Key Result

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên): user cung cấp giá trị mới cho KR
2. **Tính từ actions**: đếm actions `done` thuộc KR đó / tổng actions thuộc KR

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng).

## Trend

| Trend | Điều kiện |
|-------|----------|
| on-track | Tiến độ thực >= tiến độ kỳ vọng theo timeline |
| at-risk | Tiến độ thực < kỳ vọng nhưng chênh < 20% |
| off-track | Chênh >= 20% hoặc có actions blocked |

Tiến độ kỳ vọng = % thời gian đã trôi trong period.

Ví dụ: period 3 tháng, đã qua 1.5 tháng = kỳ vọng 50%. KR đạt 35% → at-risk.

## Cập nhật SOT

Khi track, cập nhật:
1. `objective.md`: cột Current và Status trong bảng KR
2. `plan.md`: frontmatter `completed`, `in_progress`, `blocked`
3. Mỗi action có thay đổi: frontmatter `status`

## Log format

Mỗi lần track = 1 entry trong `log/YYYY-MM-DD.md`. Nếu file ngày đó đã có, append thêm section mới.

Log ghi:
- Giá trị thay đổi (cũ → mới)
- Actions thay đổi status
- Blockers mới phát hiện
- Ghi chú của user
