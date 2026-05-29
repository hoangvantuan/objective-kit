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
| `plan.md`      | Frontmatter + Roadmap body + `## Practices` (Ongoing) | Counters + practices streak (render dashboard Ongoing)  |
| `resources.md` | Frontmatter                           | Capacity, skills, tools                                 |
| `actions/*.md` | Frontmatter only                      | Dùng Bash `ls` rồi đọc từng file, skip body trừ khi cần |
| `inbox/`       | Count + đọc frontmatter pending items | Compute staleness on-the-fly                            |
| `log/`         | 3 entry gần nhất                      | Dùng `ls -t` rồi đọc 3 file đầu                         |


Nếu `.okr/` không tồn tại → trả ngay: "Chưa khởi tạo OKR. Cần chạy init."

## Tính metrics

Mọi công thức ở `../okr-shared/references/metrics.md` (canonical). KHÔNG chép lại ở đây để tránh drift. Áp dụng đúng theo loại mục tiêu:

- **Project**: KR Progress %, KR Status (4 giá trị, first-match), Trend (on-track/at-risk/off-track), Period Overdue.
- **Ongoing**: KI Status (healthy/warning/critical), Trend (improving/stable/declining).
- **Cả hai**: Action health (done rate, overdue, blocked, checkpoint slip).

Mỗi số liệu phải trích được từ file (Nguyên tắc 3). Không ước lượng.

## Phát hiện issues

Quét theo thứ tự nghiêm trọng:

1. **Period overdue**: end_date < today + objective.status = active
2. **Action overdue**: due_date < today + status chưa done. Ghi số ngày quá hạn
3. **Action blocked**: liệt kê blocker + action bị chặn
4. **KR at-risk/off-track**: trend tính từ metrics
5. **Checkpoint slip**: action `effort: xl` có `## Checkpoints`, mốc quá hạn chưa tick (xem metrics.md "Action health")
6. **Inbox aging**: items pending > 30 ngày
7. **Capacity / xung đột tài nguyên**: dùng bảng signals ở `../okr-shared/references/metrics.md` ("Capacity / xung đột tài nguyên") — quá tải, dồn deadline, skill gap, tool missing, capacity drop

## Xếp priority (top N actions)

Dùng thuật toán first-match canonical ở `../okr-shared/references/action-priority.md` (5 bước: overdue → đang block action khác → deadline trong horizon → priority cao đang doing → KR at-risk). Truyền `max_items` + `horizon_days` từ tham số. KHÔNG chép lại các bước ở đây để tránh lệch.

Format mỗi gợi ý: `→ AXXX: Title (lý do ≤10 từ)`

## Tham số

| Param        | Mặc định                 | Mô tả                                    |
| ------------ | ------------------------ | ---------------------------------------- |
| focus        | full                     | "full", "progress", "issues", "priority" |
| max_items    | 2 (dashboard), 3 (track) | Số priority items                        |
| horizon_days | 3 (dashboard), 7 (track) | Cửa sổ deadline                          |


## Output format

Trả về phân tích cấu trúc. **Dashboard layout dưới đây là canonical**: harness (dashboard mặc định) và `okr-track` (khi chạy độc lập, không qua analyze) đều render theo đây. KHÔNG định nghĩa layout song song ở nơi khác.

Cấu trúc tổng:

```
## Dashboard
[Layout theo loại mục tiêu — xem "Dashboard layout" bên dưới]

## Issues (nếu có)
[Severity] [Type]: [description]
  Evidence: [file:line hoặc giá trị]
  Recommendation: [hành động, áp dụng qua: okr-init/okr-plan/okr-track]

## Root cause (chỉ mode deep)
[Mỗi issue cần cải thiện: nhân gốc vs duyên, tách triệu chứng]

## Priority hôm nay
→ AXXX: Title (lý do ≤10 từ)

## Recommendations (nếu có issues)
[Đề xuất, chỉ rõ skill đích + mode]
```

### Dashboard layout — Project

- Mở đầu: 1 câu tổng quan sức khoẻ.
- Nhắc review: 1 dòng (first match, `../okr-shared/references/metrics.md` "Nhắc review").
- Period overdue (`status = active`): block cảnh báo ĐẦU dashboard (trước Key Results).
- **Active actions per KR**: lọc `key_result: KR<N>`, liệt kê active IDs (blocked + doing trước, pending sau). >5 active → 3 đầu + "(+N nữa)".

```
Dashboard: [Tên Objective]
[Câu tổng quan]
[Nhắc review nếu có]
Period: [start] > [end] ([X]% thời gian đã dùng)

Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
    Actions: 3 done | 2 doing | 1 blocked | 0 pending
    Active: A001 (doing), A003 (doing), A005 (blocked: [lý do])

Actions: N tổng | X done | Y doing | Z blocked | W pending
Tốc độ: [X] done/tuần (kế hoạch: [Y])
Inbox: [N] items chưa xử lý

Cần chú ý
  - [issues: blocked, overdue, at-risk]
```

Block period overdue (render trước phần còn lại khi overdue, chi tiết hành vi: metrics.md "Period Overdue"):

```
⚠️ Period đã qua [N] ngày
End date [date], hôm nay [date]. Status vẫn `active`.

KR chưa achieved:
  - KR1: [current]/[target] ([%]) — còn thiếu [N]

Đề xuất:
  - Extend end_date (chạy `okr-init` mode update-objective)
  - Đổi status sang completed/cancelled

Dashboard: [phần còn lại]
```

### Dashboard layout — Ongoing

- Mở đầu: 1 câu tổng quan.
- Nhắc review: 1 dòng (first match, metrics.md "Nhắc review").
- **Streak**: hiển thị `current_streak` cạnh KI (từ `plan.md` `## Practices`, match `ki_link`). ≥4 tuần kèm 🔥. Mốc 7, 30, 100 tuần → ghi nhận.

```
Dashboard: [Tên Objective]
[Câu tổng quan]
[Nhắc review nếu có]
Review cycle: [weekly] (lần cuối: [date])

Key Indicators
  KI1: [tên]  ≥[ngưỡng]  current: [N]  [status]  streak: [N] tuần

Trend: [improving/stable/declining]
Inbox: [N] items

Cần chú ý
  - [KI dưới ngưỡng, streak milestones]
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

- Metrics + capacity signals: `../okr-shared/references/metrics.md`
- Priority algo: `../okr-shared/references/action-priority.md`
- Shared schemas: `../okr-shared/references/schemas.md`
- Log/progress format: `../okr-track/references/data-format.md`

  
