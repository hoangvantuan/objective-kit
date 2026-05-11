---
name: okr-track
description: "Sub-skill của /okr. Đánh giá trạng thái + cập nhật progress + xử lý inbox + đề xuất next action. 5 mode: light (progress nhanh), deep (review sâu + delegate), closure (chốt project), inbox-only (chỉ xử lý inbox), trace (xem lại archive/log). Được kích hoạt từ orchestrator /okr. KHÔNG gọi trực tiếp trừ khi user gõ /okr-track."
---

# okr-track: Track + Review + Inbox Processing

Một skill duy nhất cho 3 use case: cập nhật progress nhanh (daily), review sâu (milestone/period-end), và xử lý inbox items. Tự chọn mode dựa trên context.

**Phân vai rõ ràng**:

- `okr-track` ghi đè **progress fields** (KR.current, KI.current, action.status) và ghi log.
- `okr-track` **xử lý inbox**: phân loại items → delegate hoặc tự apply tuỳ loại.
- Thay đổi **cấu trúc** (KR target, action mới, dời deadline, đổi PIC) → delegate sang `okr-init` hoặc `okr-plan` mode `update-*`. Track đề xuất, init/plan áp dụng.

> **Tiên quyết**: Skill `okr` (orchestrator) PHẢI load trước khi skill này chạy. Context từ orchestrator đã có sẵn (SOT ownership, shared schemas, quality gate), KHÔNG đọc lại.

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại (tối thiểu).
- `.okr/plan.md` + `.okr/actions/` nên có (nếu type=project). Thiếu → cảnh báo, vẫn cho track KR/KI + xử lý inbox.
- Ongoing type có thể track chỉ với `objective.md` (KI status).

## Flow

### Phase 1: Tính toán từ context + đọc thêm

SOT data đã có từ orchestrator (objective, plan frontmatter, actions frontmatter, resources). KHÔNG đọc lại.

Đọc thêm (orchestrator chưa load):

- `.okr/inbox/*.md` với status=pending: đọc frontmatter (orchestrator chỉ đếm)
- `.okr/log/`: mode light → 1 file mới nhất. Mode deep → tối đa 3 files mới nhất. Mode closure → tất cả.
- `.okr/log/reviews/`: mode light → 1 file mới nhất. Mode deep → tối đa 3 files mới nhất. Mode closure → tất cả.
- Mode trace: đọc log/reviews theo yêu cầu user.

Tính metrics (xem `references/metrics.md`):

- Project:
  - KR % đạt, trend so với log trước, timeline, actions tổng/done/doing/blocked/pending, tốc độ done/tuần
  - **Actions group by KR**: với mỗi KR, lọc `actions/*.md` có frontmatter `key_result: KR<N>`. Đếm theo status (done/doing/blocked/pending). Liệt kê IDs active (status ∈ {doing, blocked, pending}, ưu tiên blocked + doing trước, pending sau). Nếu >5 active actions → hiển thị 3 đầu + "(+N nữa)". Mỗi blocked action kèm lý do ngắn từ field `description` hoặc 1 dòng đầu của body.
- Ongoing: KI status (healthy/warning/critical), trend so với review trước. Practices streak hiện tại (đọc từ `plan.md` body).
- Inbox: số items pending

### Phase 2: Hiển thị dashboard

**Project type:**

Trước khi render dashboard, tính `period_overdue_days = max(0, today - end_date)`. Nếu `period_overdue_days > 0` AND `objective.status = active` → render dashboard kèm block cảnh báo period overdue ở vị trí ĐẦU TIÊN, trên cả Period progress (xem mẫu thứ 2 dưới).

Dashboard chuẩn (period chưa quá hạn):

```
Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (60% thời gian đã dùng)

Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
    Actions: 3 done | 2 doing | 1 blocked | 0 pending
    Active: A001 (doing), A003 (doing), A005 (blocked: chờ approve)
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
    Actions: 1 done | 1 doing | 0 blocked | 2 pending
    Active: A007 (doing), A009 (pending), A010 (pending)
  KR3: ██████░░░░ 3/5    (60%) > ahead
    Actions: 0 done | 0 doing | 0 blocked | 1 pending
    Active: A012 (pending)

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending
Tốc độ: 1.2 done/tuần (kế hoạch: 1.5)
Inbox: 3 items chưa xử lý

Cần chú ý
  - A005 blocked 5 ngày: chờ approve từ stakeholder
  - A007 quá hạn 3 ngày
  - KR2 cần +8 đơn vị/tháng để kịp target
```

Dashboard khi period đã qua (`period_overdue_days > 0` AND `status = active`):

```
⚠️ Period đã qua 12 ngày
End date 2026-12-31, hôm nay 2027-01-12. Status objective vẫn `active`.

KR chưa achieved:
  - KR1: 80/100 (80%) — còn thiếu 20 đơn vị
  - KR2: 35/50  (70%) — còn thiếu 15 đơn vị

Đề xuất:
  - Extend end_date qua `/okr init update-objective` nếu vẫn muốn theo đuổi target
  - Đổi status sang `completed` (chấp nhận kết quả hiện tại) hoặc `cancelled`
    (dừng theo đuổi) qua `/okr init update-objective`

Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (period đã đóng, +12 ngày)
... (phần còn lại giống dashboard chuẩn)
```

Logic chi tiết: xem `references/metrics.md` section "Period Overdue (Project type)".

**Ongoing type:**

```
Dashboard: [Tên Objective]
Review cycle: weekly (lần cuối: 2026-05-02, 7 ngày trước)

Key Indicators
  KI1: Tập thể dục    ≥3 lần/tuần   current: 4   ✅ healthy
  KI2: Ngủ đủ giấc    ≥7 giờ/đêm    current: 5.5 ⚠️ critical
  KI3: Khám định kỳ   ≥1 lần/6th    current: 1   ✅ healthy

Trend: declining (KI2 từ warning → critical)
Inbox: 1 item chưa xử lý

Cần chú ý
  - KI2 dưới ngưỡng 3 tuần liên tiếp
```

### Phase 3: Tự xác định mode + check inbox

Heuristic, sau đó hỏi user xác nhận:

| Tín hiệu                                                                   | Mode đề xuất                                   |
| -------------------------------------------------------------------------- | ---------------------------------------------- |
| Daily update, không blocker mới, KR on-track                               | `light`                                        |
| Đến milestone deadline, hoặc KR at-risk, hoặc nhiều blocked                | `deep`                                         |
| User nói "review", "tổng kết", "đánh giá", "lookback"                      | `deep`                                         |
| User nói "update", "xong rồi", "cập nhật"                                  | `light`                                        |
| Mọi action `done` → đề xuất                                                | `closure` (deep + chốt)                        |
| Ongoing + đến review_cycle                                                 | `deep`                                         |
| User nói "inbox", "xử lý inbox" hoặc vào từ `/okr inbox`                   | `inbox-only` (skip progress, vào Phase 5 ngay) |
| User nói "trace", "xem lại", "history", "lịch sử" hoặc vào từ `/okr trace` | `trace` (vào Trace Flow, skip progress)        |


```
Đề xuất mode: light (cập nhật nhanh, không phân tích sâu).
Inbox: 3 items chưa xử lý. Xử lý sau khi track xong.
Đổi mode? (light/deep/closure)
```

---

### Phase 4a: Mode LIGHT (cập nhật progress nhanh)

Phạm vi: CHỈ progress fields. Không sửa cấu trúc.

**Project type:**

1. Sync pull external (nếu action có `external_ids`)
2. Hỏi user update: KR current, action status, blocker mới
3. CONFIRM: ≤2 field → 1 dòng compact, ≥3 field → bảng đầy đủ
4. Ghi SOT (objective, plan counters, action status) + append log
5. Sync push + archive actions done + re-render Roadmap
6. Xử lý inbox (Phase 5) nếu có items pending
7. Đề xuất next action (1-7 ngày tới)

**Ongoing type:**
1. Hỏi update KI current + practice streak (plan.md `## Practices`)
2. Hỏi update action status (nếu có action files)
3. CONFIRM + ghi SOT + log (kèm streak thay đổi)
4. Xử lý inbox + đề xuất cải thiện KI nếu warning/critical

Chi tiết đầy đủ: `references/flow-light.md`

---

### Phase 4b: Mode DEEP (review sâu + delegate)

Phạm vi: phân tích root cause + ĐỀ XUẤT điều chỉnh. KHÔNG tự sửa cấu trúc.

1. Update progress nếu cần (giống light, kèm sync + archive)
2. Phân tích root cause (hỏi "tại sao?" ≥3 lần mỗi vấn đề)
3. Đề xuất điều chỉnh cấu trúc (bảng kèm skill đích: okr-init/okr-plan)
4. All-changes confirm: gom theo skill đích, user chọn áp dụng
5. Delegate sang skill phù hợp (kèm payload: changes, reason, pre_confirmed)
6. Xử lý inbox (Phase 5)
7. Ghi log review vào `log/reviews/YYYY-MM-DD.md`
8. Đề xuất next action

Chi tiết đầy đủ: `references/flow-deep.md`

---

### Phase 4c: Mode CLOSURE (mọi action done)

Đọc mở rộng: `actions/archive/**` + tất cả `log/reviews/**` (tổng kết toàn period).

Như deep + thêm:
1. Tổng kết toàn period (KR đạt vs target, % thời gian, lessons learned)
2. Hỏi user: completed hay tạo follow-up project?
3. Delegate `okr-init update-objective` đổi status nếu user đồng ý
4. Xử lý inbox còn lại
5. Ghi log review closure (kèm section `## Lessons`)

Chi tiết đầy đủ: `references/flow-closure.md`

---

### Phase 5: Inbox Processing Flow

Chạy sau update progress (light) hoặc sau delegate (deep).

**Mode inbox-only**: SOT đã có context từ orchestrator okr (preload Phase 1). Skip Phase 2 dashboard, Phase 3 detect, Phase 4 progress. Đi thẳng Phase 5.

Flow:

1. Đọc inbox pending + compute staleness (on-the-fly, xem Inbox Aging đã có trong context từ orchestrator)
2. Cảnh báo stale items (>30 ngày), hỏi user giữ/bỏ
3. Hiển thị bảng gợi ý xử lý (map vào KR/milestone/action)
4. Validate related IDs (KR, action, tool) trước khi xử lý
5. Xử lý từng item theo type: delegate hoặc tự apply
6. Gom delegate cùng skill → 1 lần gọi
7. Báo cáo: items đã xử lý + inbox còn lại

Inbox type → Delegate mapping: đã có trong context từ orchestrator.

Chi tiết đầy đủ: `references/flow-inbox.md`

---

## Trace Flow (mode `trace`)

Lazy loading: frontmatter trước, body khi user drill-down. Không update progress.

4 kiểu trace:

| Kiểu | Trigger | Đọc gì |
|------|---------|--------|
| Action cụ thể | "trace A003" | `actions/archive/A003-*.md` |
| Milestone | "trace M1" | `plan.md` + `actions/archive/` theo milestone |
| Theo thời gian | "actions done tháng 4" | `actions/archive/` theo `due_date` |
| Log | "xem log tuần trước" | `log/` hoặc `log/reviews/` theo ngày |

Chi tiết đầy đủ: `references/flow-trace.md`

---

## Schema

Đọc `references/data-format.md` cho schema:

- `log/YYYY-MM-DD.md` (tracking entries)
- `log/reviews/YYYY-MM-DD.md` (review entries)
- Progress fields được phép ghi đè
- Cấu trúc fields delegate sang skill khác
- Inbox processing rules

Đọc `references/metrics.md` cho cách tính KR %, KI status, trend, timeline.

## Quy tắc

- Dashboard luôn hiển thị TRƯỚC khi hỏi update.
- Mọi thay đổi qua phase confirm.
- Mode `light`: chỉ sửa progress fields (KR.current, KI.current, action.status, plan counters). Cấm sửa cấu trúc.
- Mode `deep`: KHÔNG tự sửa KR target / action mới / dời deadline / đổi PIC. Delegate sang `okr-init` / `okr-plan`.
- Phân tích root cause BẮT BUỘC ≥3 lần "tại sao", không dừng ở triệu chứng.
- Ghi đè SOT progress fields, append log. Không ngược lại.
- Log review (deep) ghi vào `log/reviews/`. Log thường ghi vào `log/`.
- Cuối flow LUÔN đề xuất next action cụ thể (file/người/thời điểm).
- Ongoing type: dashboard hiển thị KI status (healthy/warning/critical) + practices adherence thay vì % target.
- Inbox: xử lý SAU khi update progress. Không xử lý inbox trước khi dashboard hiển thị.
- Inbox items processed → giữ file, đổi status. Không xoá file.
