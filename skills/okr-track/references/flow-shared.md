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

**Metrics đã được `okr-analyze` tính** (orchestrator chạy analyze TRƯỚC track light/deep). Track NHẬN qua input `analysis`, KHÔNG tự tính lại. Nếu track chạy KHÔNG qua analyze (vd inbox-only độc lập) → tự tính theo `../../okr-shared/references/metrics.md`.

SOT data Tier 1 (objective/plan frontmatter, actions frontmatter, `resources.md` full body, inbox count, `lessons/index.md`, conditional `context/index.md` khi `context/` tồn tại) đã có từ orchestrator (Preload Contract Tier 1, `../../okr-shared/references/preload.md`). KHÔNG đọc lại. Chạy lẻ (inbox-only độc lập, hoặc không qua harness) → tự nạp Tier 1 phần thiếu trước khi thao tác.

Đọc thêm (on-demand, KHÔNG nằm trong preload):
- `.okr/inbox/*.md` body: đọc khi xử lý từng item (count pending đã có từ preload)
- `.okr/log/`: light → KHÔNG đọc. Deep → adaptive rule (xem `data-format.md` Log Reading Rules). Closure → tất cả.
- `.okr/context/<slug>.md` body: đọc khi `context/index.md` đã nạp và cột "Khi nào cần đọc" khớp việc hiện tại (xem `../../okr-shared/references/preload.md` "Áp dụng context").
- Trace: đọc `log/` theo yêu cầu user.

`analysis` cung cấp (công thức ở `../../okr-shared/references/metrics.md`):

**Project:**
- KR % đạt, trend, timeline
- Actions tổng/done/doing/blocked/pending (done tổng từ counter `plan.md`; tốc độ done/tuần chỉ tính ở deep/closure)
- Actions group by KR (quy tắc hiển thị active IDs: xem `okr-analyze` "Dashboard layout: Project").

**Ongoing:**
- KI status (healthy/warning/critical), trend so review trước
- Practices streak (từ `plan.md` body)
- Inbox count pending

## Phase 2: Hiển thị dashboard

Render do `okr-analyze` đảm nhiệm. **Layout canonical (Project + Ongoing) ở `okr-analyze` Output format → "Dashboard layout"**. KHÔNG định nghĩa layout song song ở đây.

- Track chạy SAU analyze (light/deep qua orchestrator): analyze ĐÃ render dashboard → track KHÔNG render lại.
- Track chạy ĐỘC LẬP (inbox-only, hoặc light/deep không qua analyze): tự render theo đúng layout canonical đó.

Quy tắc liên quan đều canonical, không chép lại:
- Period overdue (Project, `status = active`): block cảnh báo đầu dashboard. Hành vi: `../../okr-shared/references/metrics.md` "Period Overdue".
- Nhắc review: metrics.md "Nhắc review" (first match, phân Project/Ongoing).
- Streak Ongoing (🔥 ≥4 tuần, mốc 7/30/100): xem `okr-analyze` "Dashboard layout: Ongoing".

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

> User nói "trace / xem lại / history" → KHÔNG phải mode của track. Đó là `okr-analyze` mode trace (read-only, xem `../../okr-analyze/references/flow-trace.md`). Harness route thẳng sang analyze.

```
Đề xuất mode: [light]. Inbox: [N] items. Đổi mode? (light/deep/closure)
```

## Quy tắc chung

- **Objective `status: paused`**: KHÔNG hỏi update progress (chỉ render dashboard + cho phép inbox/capture). In 1 dòng `(Objective đang paused. Resume qua /okr sửa mục tiêu để track lại.)` rồi skip phần hỏi update. Inbox vẫn xử lý được.
- Dashboard TRƯỚC khi hỏi update.
- Mọi thay đổi qua phase confirm.
- Action chuyển `done` → set `completed_date: today` (1 lần) trước khi archive (xem Archive Rules Bước 0).
- Light: chỉ progress fields. Cấm sửa cấu trúc.
- Deep: KHÔNG tự sửa KR target / action mới / deadline. Áp dụng qua `okr-init`/`okr-plan`.
- Root cause (≥3 lần "tại sao") do `okr-analyze` deep tính. Track deep trình bày + tinh chỉnh, không làm lại từ đầu (trừ khi chạy không qua analyze).
- Ghi đè SOT progress fields, append log. Không ngược lại.
- Output: ghi đè `## Output/Deliverable` theo `data-format.md`, giữ/cập nhật dòng `Path:` nếu deliverable là file riêng. Nhắc khi mark done, cho phép skip.
- Log vào `log/YYYY-MM-DD.md`. Deep/closure append sections review.
- Cuối flow LUÔN đề xuất next action cụ thể (theo `action-priority.md`).
- Ongoing: KI status + practices adherence thay vì % target.
- Inbox: xử lý SAU khi update progress.
- Inbox items processed → giữ file, đổi status. Không xoá.
