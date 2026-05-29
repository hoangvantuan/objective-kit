---
name: okr-harness
description: "Quản lý mục tiêu OKR (skill-only, chạy inline): dashboard, init, plan, track, review, inbox, capture, closure, trace. Trigger khi user nhắc: OKR, mục tiêu, dự án, kế hoạch, tracking, review sâu, tài nguyên, capacity, inbox, capture, ghi nhanh, dashboard, tiến độ, quá hạn, at-risk, blocker, tổng kết, rút bài học, retro, bài học. Cũng trigger khi: 'chạy lại', 'cập nhật tiến độ', 'xem tiến độ', 'hôm nay làm gì', 'review mục tiêu', 'sửa plan'. Gọi trực tiếp: /okr-harness."
---

# OKR Harness: Orchestrator (skill-only)

Entry point quản lý mục tiêu OKR. Đọc state, route, **chạy skill inline**, tổng hợp kết quả.

## Bản đồ skill

| Skill         | Vai trò                                                                            | Ghi          |
| ------------- | ---------------------------------------------------------------------------------- | ------------ |
| `okr-analyze` | Phân tích read-only: metrics, issues, priority, dashboard, trace (xem lại history) | Không        |
| `okr-init`    | Tạo/sửa objective, KR/KI, resources                                                | Có           |
| `okr-plan`    | Tạo/sửa plan, milestones, actions                                                  | Có           |
| `okr-track`   | Track progress, deep review, inbox, closure, sync                                  | Có           |
| `okr-capture` | Ghi nhanh vào inbox                                                                | Có (inbox)   |
| `okr-retro`   | Rút bài học từ phiên, ghi `.okr/lessons/` (record-only)                            | Có (lessons) |
| `okr-shared`  | Quy tắc chung: SOT, schemas, quality gate, delegate, priority                      | Không        |


## Phase 0: Context check

```
if .okr/ không tồn tại:
    → chạy okr-init mode new (inline)
    return
else:
    → Phase 1 (đọc state + xác định intent)
```

## Phase 1: Đọc state + xác định intent

### Preload

Orchestrator tự đọc nhanh (không gọi skill) để xác định state:

```
objective.md      → type (project/ongoing), status, period
plan.md           → có/không, counters
actions/          → count active, có overdue/blocked?
inbox/            → count pending
lessons/index.md  → đọc TOÀN BỘ (bài học, auto-load mỗi phiên)
```

Chỉ đọc frontmatter (trừ `lessons/index.md` đọc toàn bộ vì nhỏ, chứa câu lõi bài học). Không đọc body action, log, archive, hay file detail bài học (load on-demand khi cần).

Lessons đã nạp → dùng làm **context định hướng** khi skill đề xuất/điều chỉnh trong phiên (đối chiếu area/target liên quan). Quy tắc áp dụng: `okr-shared` SKILL "Áp dụng lessons".

### Intent routing

| State                   | User intent                            | Chạy skill (inline)                   |
| ----------------------- | -------------------------------------- | ------------------------------------- |
| Chưa có .okr/           | Bất kỳ                                 | `okr-init` new                        |
| Có objective, chưa plan | Mặc định                               | `okr-plan` new                        |
| Có objective + plan     | **Mặc định / "hôm nay"**               | `okr-analyze` (dashboard)             |
|                         | "sửa mục tiêu / KR / KI"               | `okr-init` update-objective           |
|                         | "tài nguyên / capacity / skill / tool" | `okr-init` update-resource            |
|                         | "thêm action / sửa plan / sửa action"  | `okr-plan` update                     |
|                         | "track / cập nhật / update"            | `okr-analyze` → `okr-track` light     |
|                         | "review sâu / đánh giá / phân tích"    | `okr-analyze` deep → `okr-track` deep |
|                         | "capture / ghi nhanh / note"           | `okr-capture`                         |
|                         | "inbox / xử lý inbox"                  | `okr-track` inbox-only                |
|                         | "tổng kết / closure / kết thúc"        | `okr-track` closure                   |
|                         | "trace / history / xem lại"            | `okr-analyze` trace                   |
|                         | "rút bài học / tổng kết phiên / retro" | `okr-retro`                           |


**Collision**: state gợi ý khác user intent → ưu tiên user intent + xác nhận lại.

## Phase 2: Thực thi inline

Đọc SKILL.md của skill đích, theo đúng flow của nó. Khi một flow cần chuyển sang skill khác (vd track deep cần áp dụng thay đổi cấu trúc), đọc tiếp SKILL.md kia và thực thi, mang theo context đã có.

### Dashboard (mặc định khi có plan)

Chạy `okr-analyze` (focus=full, max_items=2, horizon_days=3). Render dashboard + nhắc review nếu cần (xem "Nhắc review").

### Track light

1. Chạy `okr-analyze` (focus: progress, overdue, blocked) → thu được analysis.
2. Chạy `okr-track` light, dùng analysis làm input. Tương tác user, cập nhật progress, archive, next action.

### Deep review (chuỗi tuần tự inline, KHÔNG agent team)

1. Chạy `okr-analyze` mode deep: đọc toàn bộ `.okr/` + log history, root cause mỗi issue (≥3 "tại sao?").
2. Chạy `okr-track` deep, mang theo analysis: trình bày root cause → đề xuất điều chỉnh → all-changes confirm. User chọn áp dụng.
3. Ghi progress fields (việc của okr-track).
4. **Áp dụng thay đổi cấu trúc**: với mỗi thay đổi đã confirm thuộc cấu trúc, đọc tiếp `okr-init`/`okr-plan` và thực thi với `pre_confirmed: true` (xem `okr-shared` delegate-protocol). Cùng agent thực hiện, không spawn.
5. Ghi log review.

> Trước đây deep review chạy bằng agent team (analyst + tracker qua SendMessage). Skill-only gộp thành chuỗi tuần tự một agent: phân tích → track → áp dụng. Mất tính song song, đổi lại chạy được ở mọi nền tảng.

### Init / Plan

Chạy `okr-init` (mode new/update-objective/update-resource) hoặc `okr-plan` (new/update). Tương tác user theo flow. Confirm trước ghi.

### Capture

Chạy `okr-capture`: phân loại type, ghi `inbox/YYYY-MM-DD-HHmm-slug.md`. Xem `okr-capture` SKILL.md.

### Inbox / Closure

Chạy `okr-track` với mode tương ứng (inbox-only / closure).

### Trace

Chạy `okr-analyze` mode trace (read-only: xem lại history/archive/log). Xem `okr-analyze/references/flow-trace.md`.

## Phase 3: Tổng hợp

- Gom kết quả các bước.
- Render tóm tắt cho user: thay đổi gì, next step gì.
- Gợi ý rút bài học nếu flow có thực chất (xem `## Rút bài học`).

## Error handling

| Lỗi                       | Xử lý                                                    |
| ------------------------- | -------------------------------------------------------- |
| `.okr/` không tồn tại     | Route `okr-init` new                                     |
| File corrupt              | Báo cụ thể file nào, đề xuất sửa                         |
| Skill flow lỗi giữa chừng | Báo bước nào lỗi, giữ thay đổi đã ghi, hỏi user tiếp     |
| Phân tích thiếu dữ liệu   | Tiếp tục với dữ liệu có, ghi rõ phần thiếu trong báo cáo |


## Nhắc review

Khi hiển thị dashboard, áp quy tắc "Nhắc review" canonical ở `okr-shared/references/metrics.md` (first match, phân Project/Ongoing). Period overdue (Project) là cảnh báo riêng, ưu tiên cao hơn, render block đầu dashboard. KHÔNG định nghĩa ngưỡng riêng ở đây để tránh lệch với analyze/track.

## Test scenarios

### Happy path: Daily check-in

1. User: `/okr-harness`
2. Orchestrator đọc state → có objective + plan + actions active
3. Chạy `okr-analyze` inline → dashboard
4. Render: KR progress, action summary, top 2 priority, review reminder
5. User: "track"
6. Chạy `okr-analyze` (analysis) → `okr-track` light inline
7. Tương tác user, cập nhật, archive, next action
8. Render tóm tắt

### Happy path: Deep review

1. User: "review sâu"
2. Chạy `okr-analyze` deep: root cause mọi issue
3. Chạy `okr-track` deep: trình bày root cause + đề xuất → all-changes confirm
4. User approve → ghi progress fields
5. Có thay đổi cấu trúc → đọc tiếp `okr-init`/`okr-plan` (pre_confirmed) áp dụng inline
6. Ghi log review
7. Render tóm tắt toàn bộ thay đổi

### Error path: Chưa init

1. User: `/okr-harness`
2. `.okr/` không tồn tại
3. Chạy `okr-init` new → hướng dẫn tạo objective + resource

### Error path: Thiếu dữ liệu phân tích

1. `okr-analyze` gặp file thiếu frontmatter
2. Báo cụ thể file/field thiếu
3. Tiếp tục track với phần dữ liệu có, ghi chú phần thiếu trong tóm tắt

## Rút bài học

Thay cho cơ chế tự cải tiến cũ. Việc rút bài học do skill `okr-retro` đảm nhiệm, **user chủ động**.

**Gợi ý cuối flow**: sau khi tổng hợp (Phase 3), với flow có thực chất (track, deep review, init, plan, inbox; trừ dashboard/capture/trace đơn giản), thêm đúng 1 dòng:

> "Muốn rút bài học phiên này? (chạy okr-retro)"

KHÔNG tự chạy. User đồng ý hoặc tự gõ "rút bài học" → chạy `okr-retro` inline.

`okr-retro` ghi 2 loại bài học vào `.okr/lessons/`:

- **Loại A (cải tiến skill)**: hàng đợi, user port thủ công về repo gốc `objective-kit` để cải tiến harness.
- **Loại B (project cụ thể)**: tri thức tại chỗ, auto-load mỗi phiên qua index.

Record-only: `okr-retro` KHÔNG tự sửa file skill, KHÔNG ghi CHANGELOG. Chi tiết: `okr-retro` SKILL.md.

## Tham khảo

- I/O từng skill: `references/skill-contract.md`
- Sơ đồ luồng: `references/flows.md`
