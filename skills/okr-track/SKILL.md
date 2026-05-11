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

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại (tối thiểu).
- `.okr/plan.md` + `.okr/actions/` nên có (nếu type=project). Thiếu → cảnh báo, vẫn cho track KR/KI + xử lý inbox.
- Ongoing type có thể track chỉ với `objective.md` (KI status).

## Flow

### Phase 1: Đọc state + tính toán

Đọc song song:

- `.okr/objective.md` (KR/KI hiện tại)
- `.okr/plan.md` (milestones) (nếu có)
- frontmatter `.okr/actions/*.md` (**không đệ quy**, bỏ qua `actions/archive/`) (status, due_date, pic) (nếu có)
- `.okr/resources.md` (PIC khả dụng) (nếu có)
- `.okr/inbox/*.md` với status=pending (đếm + đọc frontmatter)
- **Chỉ 1 file mới nhất** trong `.okr/log/reviews/` (nếu có, để so trend)

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

1. Hỏi user có thay đổi: KR current, action status (pending/doing/done/blocked), blocker mới.
2. CONFIRM trước khi ghi (đếm số field thay đổi rồi chọn UI):
   - **Nếu ≤2 field thay đổi** → confirm 1 dòng (giảm friction quick check-in):
     ```
     KR1: 40>50, A003 done. (y/sửa/huỷ)
     ```
     hoặc:
     ```
     A005 blocked (chờ approve). (y/sửa/huỷ)
     ```
   - **Nếu ≥3 field thay đổi** → confirm bảng đầy đủ:
     ```
     Thay đổi sắp ghi
     - KR1.current: 40 > 50
     - A003.status: doing > done
     - A005.status: doing > blocked, lý do: chờ approve
     Xác nhận? (y/sửa/huỷ)
     ```
   - Cả 2 nhánh: user trả lời `sửa` → hỏi field nào cần sửa, sửa xong CONFIRM lại. `huỷ` → bỏ toàn bộ thay đổi, kết thúc light mode.
3. Áp dụng:
  - Ghi đè progress: `objective.md` (KR.current, KR status), `plan.md` (counters), `actions/*.md` (frontmatter status).
  - Append log: `.okr/log/YYYY-MM-DD.md`. File ngày đã có → append section mới.
4. **Archive actions done** (nếu có action chuyển sang `done` trong lần này):
  - Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md` (tạo thư mục `archive/` nếu chưa có).
  - Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`.
  - Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter `plan.md` (với `status: done`).
  - Cập nhật counters frontmatter `plan.md` (`completed` +N).
5. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
6. Đề xuất next action: highlight việc cần làm trong 1-7 ngày tới.

**Ongoing type:**

1. Hiển thị KI hiện tại (tên, ngưỡng, current, status).
2. Hỏi user update từng KI: "KI1 (Tập thể dục, hiện tại: 2, ngưỡng: ≥3). Tuần này bao nhiêu?"
3. Tính status mới theo logic: healthy (≥ ngưỡng), warning (< ngưỡng, chênh < 20%), critical (< 80% ngưỡng). Xem `references/metrics.md`.
4. **Update practice streak**: đọc `plan.md` body section `## Practices` (nếu có). Với mỗi practice (`### PN: <Tên>` kèm field `frequency`, `target_count`, `current_streak`, `ki_link`):
   - Tính chu kỳ vừa qua dựa trên `frequency` (vd `weekly` = tuần vừa rồi, `daily` = hôm qua). Hỏi: "Practice **P1: Tập thể dục** (target ≥3 lần/tuần, streak hiện tại: 2). Tuần vừa rồi đạt target chưa? (y = đạt → streak +1 / n = không đạt → reset streak về 0 / skip = bỏ qua không update)".
   - User trả lời `y` → `current_streak += 1`. `n` → `current_streak = 0`. `skip` → giữ nguyên.
   - Nếu practice `ki_link` đã có KI tương ứng vừa update ở step 2-3, hiển thị inline để user thấy liên kết: `(P1 → KI1 healthy)`.
5. Nếu có action files (task cải thiện KI) → hỏi update status (pending/doing/done/blocked) như Project.
6. CONFIRM trước khi ghi (tương tự Project: ≤2 field → 1 dòng, ≥3 field → bảng). Bao gồm cả `current_streak` thay đổi vào diff list.
7. Áp dụng: ghi đè `objective.md` (KI current, status), ghi đè `plan.md` body section `## Practices` (chỉ field `current_streak` cho practice nào thay đổi). Nếu có actions → cập nhật `plan.md` counters + `actions/*.md`.
8. Append log: ngoài KI/action, ghi cả practice streak thay đổi (vd `## Thay đổi → P1.current_streak: 2 > 3`).
9. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
10. Đề xuất: nếu KI warning/critical → gợi ý tạo action cải thiện qua `/okr plan`. Nếu practice streak vừa reset về 0 → cảnh báo "Practice P1 đã reset streak, KI1 có thể giảm tuần tới. Cân nhắc điều chỉnh practice (giảm target_count, đổi thời gian) qua `/okr plan update`".

---

### Phase 4b: Mode DEEP (review sâu + delegate điều chỉnh cấu trúc)

Phạm vi: phân tích root cause + ĐỀ XUẤT điều chỉnh. KHÔNG tự sửa cấu trúc. Việc apply đẩy sang `okr-init` hoặc `okr-plan`.

#### Bước 1: Update progress nếu cần

Hỏi user có update progress nào trước phân tích (giống light). Nếu có → áp dụng + log + **archive actions done** như light (Phase 4a bước 4).

#### Bước 2: Phân tích root cause

Cho mỗi vấn đề (KR at-risk, KI critical/warning, blocker, quá hạn):

- Hỏi "tại sao?" tối thiểu 3 lần.
- Phân biệt nhân (gốc) vs duyên (điều kiện).
- Tách triệu chứng vs nguyên nhân.

Hiển thị:

```
Phân tích
## Đạt tốt
  - KR3 vượt kế hoạch vì [lý do]
## Cần cải thiện
  - KR2 trễ 15%: root cause là [...] (không phải triệu chứng [...])
  - A005 blocked 5 ngày: nguyên nhân là dependency [...]
```

#### Bước 3: Đề xuất điều chỉnh cấu trúc (kèm phân loại delegate)

Mỗi đề xuất gắn nhãn skill sẽ áp dụng:

```
Đề xuất điều chỉnh
| # | Đề xuất                            | Lý do            | Áp dụng qua            |
|---|------------------------------------|------------------|------------------------|
| 1 | Giảm KR2 target: 50 > 35           | Market shift     | okr-init update-objective |
| 2 | Thêm action A013 "Tăng marketing"  | Thiếu đẩy KR2    | okr-plan update        |
| 3 | Dời M2: 2026-11-15 > 2026-11-30    | A005 chậm        | okr-plan update        |
| 4 | Đổi PIC A007: An > Dũng            | An quá tải       | okr-init update-resource|
| 5 | Tăng khả dụng Bình: 50% > 80%      | Cần đẩy build    | okr-init update-resource|
```

#### Bước 4: User chọn áp dụng cái nào

```
Đồng ý đề xuất nào? (vd: 1,3,4 / không / sửa N: <new value>)
```

#### Bước 5: Delegate sang skill phù hợp

Gom đề xuất user đồng ý theo skill target:

- Đề xuất gắn `okr-init update-objective` → kích hoạt `okr-init` mode `update-objective` với danh sách thay đổi KR/period.
- Đề xuất gắn `okr-init update-resource` → kích hoạt `okr-init` mode `update-resource` với danh sách thay đổi PIC/khả dụng/tool.
- Đề xuất gắn `okr-plan update` → kích hoạt `okr-plan` mode `update` với danh sách thay đổi action/milestone.

Skill được delegate sẽ tự chạy phase confirm + ghi file (theo flow của riêng nó). Track CHỈ truyền context (lý do + giá trị mới), KHÔNG tự ghi SOT objective/plan.

#### Bước 6: Xử lý inbox (nếu có items pending)

Chạy Inbox Processing Flow (xem Phase 5).

#### Bước 7: Ghi log review

Sau khi tất cả delegate + inbox processing hoàn tất, append `.okr/log/reviews/YYYY-MM-DD.md`:

- Tổng kết KR/KI
- Phân tích root cause
- Đề xuất + cái nào đã apply (kèm skill nào áp dụng)
- Inbox items đã xử lý

Đồng thời append `log/YYYY-MM-DD.md` link sang file review.

#### Bước 8: Đề xuất next action

---

### Phase 4c: Mode CLOSURE (mọi action done)

Đọc mở rộng: **đọc cả `actions/archive/**` (tổng kết toàn bộ actions đã hoàn thành) + **đọc tất cả `log/reviews/**` (tổng kết toàn period).

Như deep + thêm:

1. Tính tổng kết toàn period: KR đạt vs target, % thời gian, lessons learned.
2. Hỏi user: chuyển status objective → `completed`, hay tạo follow-up project?
3. Nếu user đồng ý → delegate sang `okr-init` mode `update-objective` để đổi `status: completed`.
4. Xử lý inbox còn lại (nếu có).
5. Ghi log review closure (kèm section `## Lessons`).

---

### Phase 5: Inbox Processing Flow

Chạy sau khi update progress (light) hoặc sau delegate (deep). Cũng có thể chạy độc lập nếu user gọi `/okr track` chỉ để xử lý inbox.

#### Bước 1: Đọc inbox + compute staleness

Đọc tất cả `.okr/inbox/*.md` có `status: pending`. Nếu không có → skip toàn bộ Phase 5.

Với mỗi item, compute `staleness_days = today - captured_at` (xem `skills/okr-capture/references/data-format.md` section "Inbox Aging"). Phân loại:

- Mới (≤7 ngày): xử lý bình thường ở Bước 2.
- Đang chờ (7 < staleness ≤ 30): xử lý bình thường, nhưng sort lên đầu trong bảng Bước 2.
- Cũ (>30 ngày): chuyển sang Bước 1.5 xử lý riêng trước.

#### Bước 1.5: Cảnh báo stale items (chỉ chạy nếu có items >30 ngày)

Hiển thị block riêng TRƯỚC bảng Bước 2:

```
⚠️ Inbox cũ ≥30 ngày (3 items)
| # | Type     | Title                        | Captured   | Staleness |
|---|----------|------------------------------|------------|-----------|
| 1 | thought  | Thử framework X cho frontend | 2026-04-01 | 40 ngày   |
| 2 | action   | Viết blog post về Y          | 2026-03-25 | 47 ngày   |
| 3 | resource | Library Z hỗ trợ chart       | 2026-03-15 | 57 ngày   |

Còn relevant không? (vd: "1 giữ, 2 bỏ, 3 giữ" / all giữ / all bỏ)
```

User trả lời:
- `<N> giữ`: giữ `status: pending` (không thay đổi). Item đi tiếp vào Bước 2 xử lý bình thường.
- `<N> bỏ`: đổi `status: discarded` ngay (không cần qua Bước 2-3).
- `all giữ`: giữ tất cả, đẩy hết sang Bước 2.
- `all bỏ`: discard tất cả.

KHÔNG auto-discard. User quyết định cuối.

Sau Bước 1.5, đi tiếp Bước 2 với items đã filter (đã giữ + items ≤30 ngày).

#### Bước 2: Hiển thị inbox + gợi ý xử lý

Agent đọc context từng item, đối chiếu với SOT hiện tại (objective, plan, actions, resources), rồi gợi ý cách xử lý:

```
Inbox: 3 items chưa xử lý
| # | Type     | Title                         | Captured  | Gợi ý xử lý                    |
|---|----------|-------------------------------|-----------|----------------------------------|
| 1 | action   | Viết unit test cho API auth   | 05-09     | → Tạo A014, gán M2, due 05-15   |
| 2 | thought  | Thử framework X cho frontend  | 05-08     | → Giữ inbox (chưa rõ scope)     |
| 3 | blocker  | Server staging down           | 05-09     | → Block A007, ghi log            |

Xử lý items nào? (1,3 / all / skip / bỏ N)
```

Gợi ý xử lý dựa trên:

- `action`: map vào KR/milestone nào, capacity còn lại bao nhiêu, deadline nào hợp lý
- `blocker`: action nào bị ảnh hưởng?
- `resource`: resources.md cần thêm gì (Công cụ / Tài liệu)?
- `thought`: đủ rõ để thành action chưa? Nếu rõ → tạo action; nếu chưa rõ → giữ inbox; nếu chỉ là ghi chú → append log ngày

#### Bước 3: Xử lý từng item user chọn

| Inbox type | Xử lý                                                                  | Ai thực hiện                                 |
| ---------- | ---------------------------------------------------------------------- | -------------------------------------------- |
| `action`   | Chuyển thành action file trong `actions/`, cập nhật `plan.md`          | Delegate → `okr-plan` mode `update`          |
| `blocker`  | Đánh dấu action liên quan = `blocked` + ghi lý do                      | `okr-track` tự xử lý (progress field)        |
| `resource` | Thêm tool/tài liệu vào resources.md                                    | Delegate → `okr-init` mode `update-resource` |
| `thought`  | User chọn: giữ inbox / chuyển thành action / append log ngày / bỏ      | Tuỳ lựa chọn (`okr-track` append log nếu chọn) |

> **Migrate dữ liệu cũ**: Nếu file inbox có `type: idea` hoặc `type: note` (schema cũ), `okr-track` Phase 5 tự đổi sang `type: thought` khi xử lý xong (cùng lúc set `status: processed`). Không cần user thao tác thêm.


Với mỗi item xử lý xong → đổi `status: processed` trong file inbox.

User chọn "bỏ N" → đổi `status: discarded`.

Giữ inbox (thought chưa rõ) → giữ `status: pending`, không làm gì.

#### Bước 4: Gom delegate

Nếu nhiều items cùng delegate sang 1 skill (vd: 3 actions mới cùng sang `okr-plan update`) → gom thành 1 lần delegate. Skill được delegate sẽ tự chạy phase confirm + ghi file.

#### Bước 5: Báo cáo

```
Inbox đã xử lý: 2/3 items
  - #1 → A014 (tạo mới, gán M2)
  - #3 → A007 blocked (ghi log)
  - #2 → giữ inbox (chờ rõ hơn)
Inbox còn lại: 1 item pending
```

---

## Trace Flow (mode `trace`)

Chạy khi user gọi `/okr trace <ID>`, `/okr history`, hoặc nhắc đến "xem lại", "lịch sử". **Không** update progress. **Không** đọc file active bình thường.

**Nguyên tắc lazy loading**: Đọc dần, không đọc hết. Frontmatter trước, body khi user yêu cầu.

### Trace 1 action cụ thể

Trigger: "trace A003", "xem lại A003"

1. Tìm file `actions/archive/A003-*.md`
2. Đọc frontmatter → hiển thị tóm tắt (id, title, milestone, status, pic, due_date, effort)
3. User muốn xem chi tiết → đọc body (DoD, Output, Tiêu chí chất lượng)

```
Archive: A003
  Title     : Spec MVP
  Milestone : M1: Research
  PIC       : An
  Due date  : 2026-10-20
  Effort    : m
  Status    : done

Xem chi tiết? (y/n)
```

### Trace milestone

Trigger: "trace M1", "xem lại milestone Research"

1. Đọc frontmatter `plan.md` → lấy info milestone done (name, target_date, status)
2. Lọc `actions/archive/*.md` có `milestone: "M1: Research"` (chỉ frontmatter)
3. Hiển thị bảng tóm tắt: ID, Title, PIC, Due date, Effort
4. User chọn action nào cần xem → đọc body file đó

### Trace theo thời gian

Trigger: "xem lại actions tuần trước", "actions done tháng 4"

1. Lọc `actions/archive/*.md` theo `due_date` trong frontmatter
2. Hiển thị danh sách phù hợp (chỉ frontmatter)
3. User drill-down khi cần

### Trace log

Trigger: "xem log tuần trước", "log tháng 4", "review lần trước"

1. Lọc `log/*.md` hoặc `log/reviews/*.md` theo filename (chứa ngày)
2. Hiển thị danh sách ngày có log
3. User chọn ngày → đọc nội dung file đó

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
