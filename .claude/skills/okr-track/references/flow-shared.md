# Flow: Track Shared Logic (mode detection + dashboard + rules)

> Self-contained. Orchestrator chạy inline, mang theo context đã có.

## Phân vai

- `okr-track` ghi đè **progress fields** (KR.current, KI.current, action.status, action body `## Output/Deliverable`) và ghi log.
- `okr-track` **xử lý inbox**: phân loại → áp dụng qua skill khác hoặc tự apply.
- Thay đổi **cấu trúc** (KR target, action mới, deadline) → áp dụng qua `okr-init`/`okr-plan`. Track đề xuất, init/plan áp dụng.

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại (tối thiểu).
- `.okr/plan.md` + `.okr/actions/` nên có (project). Thiếu → cảnh báo, vẫn cho track KR/KI + inbox.
- Ongoing có thể track chỉ với `objective.md` (KI status).

## Phase 1: Lấy metrics (từ analysis) + đọc thêm

**Metrics đã được `okr-analyze` tính** (orchestrator chạy analyze TRƯỚC track light/deep). Track NHẬN qua input `analysis`, KHÔNG tự tính lại. Nếu track chạy KHÔNG qua analyze (vd inbox-only hoặc trace độc lập) → tự tính theo `../../okr-shared/references/metrics.md`.

SOT data (objective, plan frontmatter, actions frontmatter, resources) đã có từ orchestrator. KHÔNG đọc lại.

Đọc thêm:
- `.okr/inbox/*.md` status=pending: frontmatter
- `.okr/log/`: light → KHÔNG đọc. Deep → adaptive rule (xem `data-format.md` Log Reading Rules). Closure → tất cả.
- Trace: đọc `log/` theo yêu cầu user.

`analysis` cung cấp (công thức ở `../../okr-shared/references/metrics.md`):

**Project:**
- KR % đạt, trend, timeline
- Actions tổng/done/doing/blocked/pending, tốc độ done/tuần
- Actions group by KR: lọc `key_result: KR<N>`, đếm status, liệt kê active IDs (blocked + doing trước, pending sau). >5 active → 3 đầu + "(+N nữa)".

**Ongoing:**
- KI status (healthy/warning/critical), trend so review trước
- Practices streak (từ `plan.md` body)
- Inbox count pending

## Phase 2: Hiển thị dashboard

### Project type

Trước dashboard, tính `period_overdue_days = max(0, today - end_date)`. Nếu overdue AND `status = active` → block cảnh báo ĐẦU TIÊN.

Mở đầu: 1 câu tổng quan sức khoẻ.

**Nhắc review** (1 dòng, sau câu tổng quan, first match):

| Điều kiện | Thông báo |
|-----------|-----------|
| `last_track_date` is null | "Chưa track lần nào." |
| `today > start + (end - start) / 2` AND `last_review_date` is null | "Đã qua nửa period, chưa review." |
| `today - last_track_date > 14 ngày` | "Chưa track 2 tuần." |

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

Dashboard khi period overdue:

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

### Ongoing type

Mở đầu: 1 câu tổng quan.

**Nhắc review** (first match):

| Điều kiện | Thông báo |
|-----------|-----------|
| `last_track_date` is null | "Chưa track lần nào." |
| `last_review_date` is null AND `today - last_track_date > review_cycle × 2` | "Track nhiều lần nhưng chưa review." |
| `today - last_review_date > review_cycle × 1.5` | "Quá hạn review N ngày." |
| `today - last_track_date > 14 ngày` | "Chưa track 2 tuần." |

**Streak**: hiển thị `current_streak` cạnh KI (từ `plan.md` `## Practices`, match `ki_link`). Streak ≥ 4 tuần kèm 🔥. Mốc 7, 30, 100 tuần → ghi nhận.

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

## Phase 3: Tự xác định mode

Heuristic, hỏi user xác nhận:

| Tín hiệu | Mode |
|-----------|------|
| Daily update, on-track | `light` (xem `flow-light.md`) |
| Milestone deadline / KR at-risk / nhiều blocked | `deep` (xem `flow-deep.md`) |
| User nói "review", "tổng kết", "đánh giá" | `deep` |
| User nói "update", "xong rồi", "cập nhật" | `light` |
| Mọi action done | `closure` (xem `flow-closure.md`) |
| Ongoing + đến review_cycle | `deep` |
| User nói "inbox", "xử lý inbox" | `inbox-only` (xem `flow-inbox.md`) |
| User nói "trace", "xem lại", "history" | `trace` (xem `flow-trace.md`) |

```
Đề xuất mode: [light]. Inbox: [N] items. Đổi mode? (light/deep/closure)
```

## Quy tắc chung

- Dashboard TRƯỚC khi hỏi update.
- Mọi thay đổi qua phase confirm.
- Light: chỉ progress fields. Cấm sửa cấu trúc.
- Deep: KHÔNG tự sửa KR target / action mới / deadline. Áp dụng qua `okr-init`/`okr-plan`.
- Root cause (≥3 lần "tại sao") do `okr-analyze` deep tính. Track deep trình bày + tinh chỉnh, không làm lại từ đầu (trừ khi chạy không qua analyze).
- Ghi đè SOT progress fields, append log. Không ngược lại.
- Output: ghi đè `## Output/Deliverable`. Nhắc khi mark done, cho phép skip.
- Log vào `log/YYYY-MM-DD.md`. Deep/closure append sections review.
- Cuối flow LUÔN đề xuất next action cụ thể (theo `action-priority.md`).
- Ongoing: KI status + practices adherence thay vì % target.
- Inbox: xử lý SAU khi update progress.
- Inbox items processed → giữ file, đổi status. Không xoá.
