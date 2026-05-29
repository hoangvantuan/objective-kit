# Metrics: công thức tính & tín hiệu chẩn đoán (canonical)

Logic ĐỌC dùng chung. `okr-analyze` đọc để render dashboard + phát hiện issue. `okr-track` đọc để compute lại `Current`/`Status` trước khi ghi đè SOT. Đây là bản canonical duy nhất: skill khác link về đây, KHÔNG chép lại công thức (tránh drift).

## Tiến độ Key Result (Project type)

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên, dùng ở MỌI mode): user cung cấp giá trị mới cho KR.
2. **Tính từ actions** (CHỈ `deep`/`closure`): đếm actions `done` thuộc KR đó / tổng actions thuộc KR. Cách này phải đọc cả `actions/archive/` vì action `done` đã được dời khỏi `actions/` active. Light/dashboard KHÔNG dùng cách 2 (chỉ đọc active, không thấy action done đã archive).

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng). Cách 2 chỉ là tham chiếu phụ khi user chưa nhập current và đang ở mode đọc lịch sử.

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
- `warning`: 80% × ngưỡng tối thiểu ≤ current < ngưỡng tối thiểu
- `critical`: current < 80% × ngưỡng tối thiểu

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

## Period Overdue (Project type)

Áp dụng chỉ cho `type: project`. Ongoing không có deadline → không cần.

### Detect

```
period_overdue_days = max(0, today - end_date)
overdue = (period_overdue_days > 0) AND (objective.status == "active")
```

Ví dụ: `end_date = 2026-12-31`, `today = 2027-01-12`, `status = active` → `overdue = true`, `period_overdue_days = 12`.

### Hành vi dashboard (analyze render, track tái sử dụng)

- Dashboard: nếu `overdue` → render block cảnh báo ở vị trí đầu (trước Key Results), liệt kê KR chưa `achieved` + đề xuất hành động (extend end_date hoặc đổi status).
- Không tự đổi status. Chỉ ĐỀ XUẤT, user quyết.
- Nếu `objective.status` đã là `completed` / `cancelled` / `paused` / `archived` → KHÔNG render cảnh báo (period overdue chỉ relevant khi user vẫn theo đuổi).

### Đề xuất hành động (in trong cảnh báo)

| Tình huống | Đề xuất |
|------------|---------|
| Còn KR gần đạt (≥80%) | Extend end_date thêm vài tuần qua `okr-init update-objective`, deadline mới thực tế dựa trên tốc độ hiện tại |
| Mọi KR < 50% | Cân nhắc `cancelled` (scope không khả thi với capacity) hoặc reset KR target qua `update-objective` |
| Mix (1 KR đạt, 1 KR cách xa) | Chia objective: đóng phần đạt thành `completed`, tách phần còn lại thành objective mới |

Đề xuất chỉ là gợi ý. Track không enforce, user tự chọn.

## Nhắc review (dashboard, dùng chung)

> Canonical duy nhất. `okr-analyze` (render dashboard), `okr-harness` (dashboard mặc định), `okr-track` flow-shared (Phase 2) đều áp quy tắc này. KHÔNG chép ngưỡng ở nơi khác.

Khi render dashboard, in tối đa 1 dòng nhắc (first match theo thứ tự bảng). Phân theo loại mục tiêu:

**Project:**

| # | Điều kiện | Thông báo |
|---|-----------|-----------|
| 1 | `last_track_date` is null | "Chưa track lần nào." |
| 2 | `today > start_date + (end_date - start_date) / 2` AND `last_review_date` is null | "Đã qua nửa period, chưa review." |
| 3 | `today - last_track_date > 14 ngày` | "Chưa track 2 tuần." |

**Ongoing:**

| # | Điều kiện | Thông báo |
|---|-----------|-----------|
| 1 | `last_track_date` is null | "Chưa track lần nào." |
| 2 | `last_review_date` is null AND `today - last_track_date > review_cycle × 2` | "Track nhiều lần nhưng chưa review." |
| 3 | `today - last_review_date > review_cycle × 1.5` | "Quá hạn review N ngày." |
| 4 | `today - last_track_date > 14 ngày` | "Chưa track 2 tuần." |

Period overdue (Project) là cảnh báo riêng, ưu tiên cao hơn nhắc review: render block đầu dashboard (xem "Period Overdue" ở trên).

## Action health

- **Done rate**: `completed / total_actions`, lấy từ **counters frontmatter `plan.md`**, KHÔNG đếm lại từ active (action `done` đã archive khỏi `actions/`, đếm active sẽ luôn ra 0).
- **Overdue**: `due_date < today` AND status ∈ {doing, blocked, pending}. Tính từ frontmatter `actions/*.md` active. Ghi số ngày quá hạn.
- **Blocked**: status = blocked (active). Liệt kê blocker reason.
- **Checkpoint slip**: action `effort: xl` có body section `## Checkpoints`, mốc `- [ ]` quá hạn (`by YYYY-MM-DD < today`) mà chưa tick. Ghi rõ "Action AXXX trượt checkpoint N".

## Tốc độ hoàn thành (CHỈ deep/closure)

Tốc độ = số action `done` trong N tuần gần / N tuần. Cần `completed_date` của action (track ghi khi status→done; xem action schema `okr-plan/references/data-format.md`). Vì action done đã archive, chỉ tính ở mode đọc lịch sử:

- **deep**: đọc `log/` (đã có dòng `status: doing > done` theo ngày) để đếm done/tuần kể từ review trước.
- **closure**: đọc `actions/archive/` (có `completed_date`) tính tốc độ toàn period.

Light/dashboard KHÔNG tính tốc độ (không đọc log/archive). Dashboard chỉ hiển thị **tổng done** từ counter `plan.md` (`completed`).

## Capacity / xung đột tài nguyên (signals)

Đọc `resources.md` (Solo Profile capacity, skills, tool/tài liệu status) đối chiếu `actions/*.md`. Dùng khi: `okr-analyze` phát hiện issue runtime, `okr-init update-resource` hoàn tất, `okr-plan` check fit lúc tạo/sửa plan.

| Tín hiệu | Cảnh báo |
|----------|----------|
| Tổng giờ ước tính của actions chưa done > capacity còn lại đến end_date | Quá tải, đề xuất giảm scope hoặc dời deadline |
| ≥3 actions cùng deadline (±2 ngày) | Tuần đó dồn việc, đề xuất tách deadline |
| Action cần skill chưa có trong Solo Profile | Đề xuất thêm action học/outsource trước |
| Tool/tài liệu status `missing` mà có action phụ thuộc | Block, đề xuất bổ sung trước |
| Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done | Cảnh báo scope rủi ro, đề xuất xem lại plan |

Mỗi cảnh báo phải kèm đề xuất giải pháp cụ thể: dời deadline, tách task, học/outsource skill, hoặc giảm scope.
