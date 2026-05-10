# Metrics: Cách tính và log format

## Tiến độ Key Result (Project type)

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên): user cung cấp giá trị mới cho KR
2. **Tính từ actions**: đếm actions `done` thuộc KR đó / tổng actions thuộc KR

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng).

## KR Status auto-compute (Project type)

KR Status có 4 giá trị: `pending` | `in-progress` | `achieved` | `missed`. `okr-track` (mode `light` hoặc `deep`) tự compute lại status mỗi lần KR.current thay đổi, theo rule:

Status được compute theo **thứ tự ưu tiên** (first match wins, rule trên đè rule dưới):

| # | Status | Điều kiện |
|---|--------|-----------|
| 1 | `achieved` | `current >= target` (đạt hoặc vượt target, kể cả khi đã quá hạn) |
| 2 | `missed` | `current < target` AND `now > end_date` (hết hạn, chưa đạt) |
| 3 | `pending` | `current == baseline` AND `now <= end_date` (chưa bắt đầu, còn hạn) |
| 4 | `in-progress` | còn lại (còn hạn, current khác baseline và chưa đạt target, gồm cả regression `current < baseline`) |

**Lưu ý**:
- Rule `achieved` (1) đè `missed` (2): vượt target rồi thì coi như đạt, dù đã quá hạn.
- Rule `missed` (2) đè `in-progress` (4): quá hạn mà chưa đạt thì miss, không còn "đang chạy".
- Regression (`current < baseline` khi còn hạn) rơi vào `in-progress` (rule 4 catch-all). Vẫn coi là đang chạy, chỉ là tụt lùi.
- KR `missed` không tự reset: user extend `end_date` qua `okr-init update-objective` thì lần track kế re-compute (rule 4 catch-all → `in-progress`).
- Không cần user tự set status khi tạo KR mới: mặc định `pending` (rule 3) vì `current == baseline`. Khi user nhập `current` lần đầu, status tự nhảy theo rule.

## Key Indicator Status (Ongoing type)

KI không tính % tiến độ. KI đánh giá trạng thái so với ngưỡng tối thiểu.

Status logic:
- `healthy`: current ≥ ngưỡng tối thiểu
- `warning`: current < ngưỡng nhưng chênh lệch < 20%
- `critical`: current < 80% ngưỡng tối thiểu

Ví dụ: KI "Tập thể dục ≥3 lần/tuần"
- Current = 3 → healthy
- Current = 2.5 → warning (< 3 nhưng > 2.4)
- Current = 2 → critical (< 80% × 3 = 2.4)

Cập nhật Current: user tự nhập khi check-in theo review_cycle (weekly/biweekly/monthly).

## Trend (Project type)

| Trend | Điều kiện |
|-------|----------|
| on-track | Tiến độ thực >= tiến độ kỳ vọng theo timeline |
| at-risk | Tiến độ thực < kỳ vọng nhưng chênh < 20% |
| off-track | Chênh >= 20% hoặc có actions blocked |

Tiến độ kỳ vọng = % thời gian đã trôi trong period.

Ví dụ: period 3 tháng, đã qua 1.5 tháng = kỳ vọng 50%. KR đạt 35% → at-risk.

## Trend (Ongoing type)

Ongoing không dùng timeline trend. Thay vào đó, so sánh status hiện tại vs lần review trước:
- `improving`: có KI chuyển từ critical/warning → healthy
- `stable`: không đổi
- `declining`: có KI chuyển từ healthy → warning/critical

## Cập nhật SOT

Khi track, cập nhật:
1. `objective.md`: cột Current và Status trong bảng KR (Project) hoặc KI (Ongoing)
2. `plan.md`: frontmatter `completed`, `in_progress`, `blocked` (chỉ Project)
3. Mỗi action có thay đổi: frontmatter `status` (chỉ Project)

## Log format

Mỗi lần track = 1 entry trong `log/YYYY-MM-DD.md`. Nếu file ngày đó đã có, append thêm section mới.

Log ghi:
- Giá trị thay đổi (cũ → mới)
- Actions thay đổi status (Project)
- KI status thay đổi (Ongoing)
- Blockers mới phát hiện
- Ghi chú của user
