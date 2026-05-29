---
name: okr-analyze
description: "Phân tích trạng thái OKR read-only: đọc .okr/, tính metrics (KR%, KI status, trend), phát hiện issues (overdue, blocked, at-risk, period overdue), xếp priority top N, render dashboard. Load khi cần: dashboard, xem tiến độ, 'hôm nay làm gì', phân tích trước track/review, hoặc bước phân tích đầu của track light/deep. Read-only, KHÔNG ghi file."
---

# OKR Analyze: Phân tích trạng thái (read-only)

Đọc toàn bộ `.okr/`, tính metrics, phát hiện vấn đề, xếp ưu tiên, phân tích root cause. Đây là bước phân tích đứng trước mọi thao tác ghi (dashboard, track light, review deep).

**KHÔNG BAO GIỜ ghi file.** Skill này read-only. Việc ghi thuộc okr-init, okr-plan, okr-track.

## Nguyên tắc

1. **Read-only**: không tạo, sửa, xoá file nào trong `.okr/`.
2. **Song song**: đọc objective.md, plan.md, resources.md cùng lúc (nhiều Read call một lượt).
3. **Chính xác**: dùng công thức chuẩn, không ước lượng. Số liệu phải trích được từ file.
4. **Root cause**: hỏi "tại sao?" tối thiểu 3 lần trước khi kết luận issue (mode deep).
5. **Cụ thể**: mọi nhận định kèm evidence (file, dòng, giá trị).

## Đọc state

Đọc song song (nhiều Read call cùng lúc):

| File           | Đọc gì                                | Ghi chú                                                 |
| -------------- | ------------------------------------- | ------------------------------------------------------- |
| `objective.md` | Frontmatter + KR/KI bảng              | Loại mục tiêu (project/ongoing) quyết định metrics      |
| `plan.md`      | Frontmatter + Roadmap body            | Counters: total, completed, in_progress, blocked        |
| `resources.md` | Frontmatter                           | Capacity, skills, tools                                 |
| `actions/*.md` | Frontmatter only                      | Dùng Bash `ls` rồi đọc từng file, skip body trừ khi cần |
| `inbox/`       | Count + đọc frontmatter pending items | Compute staleness on-the-fly                            |
| `log/`         | 3 entry gần nhất                      | Dùng `ls -t` rồi đọc 3 file đầu                         |


Nếu `.okr/` không tồn tại → trả ngay: "Chưa khởi tạo OKR. Cần chạy init."

## Tính metrics

Công thức chi tiết: `../okr-track/references/metrics.md` (KR%, KI status, trend, period overdue).

### Project type

**KR Progress**: `% = (Current - Baseline) / (Target - Baseline) * 100`

**KR Status** (first match):

| #   | Status      | Điều kiện                               |
| --- | ----------- | --------------------------------------- |
| 1   | achieved    | current >= target                       |
| 2   | missed      | current < target AND now > end_date     |
| 3   | pending     | current == baseline AND now <= end_date |
| 4   | in-progress | còn lại                                 |


**Trend**:

- on-track: tiến độ thực >= % thời gian đã trôi
- at-risk: chênh < 20%
- off-track: chênh >= 20% hoặc có actions blocked

**Period Overdue**: `end_date < today` AND `objective.status == active`

### Ongoing type

**KI Status**:

- healthy: current >= ngưỡng min
- warning: 80% × min <= current < min
- critical: current < 80% × min

**Trend**: so status hiện tại vs lần review trước (improving/stable/declining)

### Action health

- Done rate: completed / total
- Overdue: due_date < today AND status ∈ {doing, blocked, pending}
- Blocked: status = blocked (liệt kê blocker reason)

## Phát hiện issues

Quét theo thứ tự nghiêm trọng:

1. **Period overdue**: end_date < today + objective.status = active
2. **Action overdue**: due_date < today + status chưa done. Ghi số ngày quá hạn
3. **Action blocked**: liệt kê blocker + action bị chặn
4. **KR at-risk/off-track**: trend tính từ metrics
5. **Inbox aging**: items pending > 30 ngày
6. **Capacity mismatch**: actions in-progress > capacity

## Xếp priority (top N actions)

Thuật toán first-match, tham khảo `../okr-shared/references/action-priority.md`:

1. Quá hạn + đang block action khác
2. Deadline trong horizon_days ngày
3. Priority critical/high đang doing
4. Gắn KR at-risk

Format mỗi gợi ý: `→ AXXX: Title (lý do ≤10 từ)`

## Tham số

| Param        | Mặc định                 | Mô tả                                    |
| ------------ | ------------------------ | ---------------------------------------- |
| focus        | full                     | "full", "progress", "issues", "priority" |
| max_items    | 2 (dashboard), 3 (track) | Số priority items                        |
| horizon_days | 3 (dashboard), 7 (track) | Cửa sổ deadline                          |


## Output format

Trả về phân tích cấu trúc:

```
## Dashboard
[Objective summary 1 dòng]
[KR/KI progress bảng: ID | Tên | Current | Target | % | Status | Trend]
[Action summary: X done / Y total, Z blocked, W overdue]

## Issues (nếu có)
[Severity] [Type]: [description]
  Evidence: [file:line hoặc giá trị]
  Recommendation: [hành động, áp dụng qua: okr-init/okr-plan/okr-track]

## Priority hôm nay
→ AXXX: Title (lý do ≤10 từ)

## Recommendations (nếu có issues)
[Đề xuất, chỉ rõ skill đích + mode]
```

## Mode deep (phân tích sâu)

Khi gọi cho review deep, bổ sung:

- Đọc thêm log history (3 review log gần + light log từ review cuối).
- Root cause mỗi issue: hỏi "tại sao?" ≥3 lần. Phân biệt nhân (gốc) vs duyên (điều kiện).
- Tách triệu chứng vs nguyên nhân.
- Output thêm section "Root cause" cho mỗi issue cần cải thiện.

## Error handling

| Lỗi                         | Xử lý                                |
| --------------------------- | ------------------------------------ |
| `.okr/` không tồn tại       | Trả "Chưa khởi tạo"                  |
| File rỗng/thiếu frontmatter | Báo cụ thể file nào, field nào thiếu |
| Actions dir rỗng            | Báo "Chưa có plan"                   |
| Log dir rỗng                | Skip phần trend history              |


## Tham khảo chi tiết

- Metrics: `../okr-track/references/metrics.md`
- Priority algo: `../okr-shared/references/action-priority.md`
- Shared schemas: `../okr-shared/references/schemas.md`
- Data format: `../okr-track/references/data-format.md`

  
