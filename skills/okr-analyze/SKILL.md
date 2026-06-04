---
name: okr-analyze
description: "Phân tích OKR read-only: metrics, issues, priority, dashboard, trace."
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

**Preload Contract Tier 1** (`../okr-shared/references/preload.md`): trước khi đọc, đảm bảo nền Tier 1 đã có trong context (`objective.md`/`plan.md` frontmatter, `resources.md` full body, actions/inbox count, `lessons/index.md`, conditional `context/index.md` khi `context/` tồn tại). Idempotent.

**Tái dùng preload**: khi chạy qua orchestrator (mặc định), nền Tier 1 ĐÃ nằm trong context từ `okr-harness` Phase 1. KHÔNG Read lại (gồm cả `resources.md` đã có full body); chỉ đọc thêm phần body cần để render (KR/KI bảng, Roadmap body, `## Practices` Ongoing, actions frontmatter). Chạy ĐỘC LẬP (không qua orchestrator, data chưa có trong context) → tự nạp Tier 1 rồi đọc bảng dưới. Nhất quán với `okr-track` `flow-shared.md` Phase 1.

Đọc song song (nhiều Read call cùng lúc):

| File           | Đọc gì                                | Ghi chú                                                 |
| -------------- | ------------------------------------- | ------------------------------------------------------- |
| `objective.md` | Frontmatter + KR/KI bảng              | Loại mục tiêu (project/ongoing) quyết định metrics      |
| `plan.md`      | Frontmatter + Roadmap body + `## Practices` (Ongoing) | Counters + practices streak (render dashboard Ongoing)  |
| `resources.md` | TOÀN BỘ body                          | Capacity, skills, tools (data ở body, không phải frontmatter) |
| `actions/*.md` | Frontmatter only                      | Dùng Bash `ls` rồi đọc từng file, skip body trừ khi cần |
| `inbox/`       | Count + đọc frontmatter pending items | Compute staleness on-the-fly                            |
| `log/`         | Ongoing: review log gần nhất (tính trend). Project default: KHÔNG đọc | Deep/closure đọc nhiều hơn (xem okr-track Log Reading Rules) |
| `context/index.md` | TOÀN BỘ (conditional: chỉ khi `context/` tồn tại) | Reachability + "Áp dụng context": nạp body `<slug>.md` khi việc hiện tại khớp cột "Khi nào cần đọc" |


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
7. **Capacity / xung đột tài nguyên**: dùng bảng signals ở `../okr-shared/references/metrics.md` ("Capacity / xung đột tài nguyên"): quá tải, dồn deadline, skill gap, tool missing, capacity drop
8. **Reachability (chống file mồ côi)**: chạy "Reachability audit" (xem section dưới). Light = ls cấp 1 (cảnh báo nhẹ). Deep/closure = thuật toán đầy đủ (báo mồ côi + link chết + context lệch).

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
[Layout theo loại mục tiêu, xem "Dashboard layout" bên dưới]

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

### Dashboard layout: Project

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
    Active: 2 doing | 1 blocked | 0 pending
    → A001 (doing), A003 (doing), A005 (blocked: [lý do])

Actions: N tổng | X done | Y doing | Z blocked | W pending
Inbox: [N] items chưa xử lý

Cần chú ý
  - [issues: blocked, overdue, at-risk]
```

> **Nguồn done count**: dòng tổng `X done` lấy từ counter `plan.md` (`completed`). Per-KR CHỈ hiển thị active (doing/blocked/pending) vì action done đã archive, không đếm được per KR từ active. Tốc độ done/tuần là metric của `deep`/`closure` (đọc log/archive), KHÔNG hiển thị ở dashboard light.

Block period overdue (render trước phần còn lại khi overdue, chi tiết hành vi: metrics.md "Period Overdue"):

```
⚠️ Period đã qua [N] ngày
End date [date], hôm nay [date]. Status vẫn `active`.

KR chưa achieved:
  - KR1: [current]/[target] ([%]), còn thiếu [N]

Đề xuất:
  - Extend end_date (chạy `okr-init` mode update-objective)
  - Đổi status sang completed/cancelled

Dashboard: [phần còn lại]
```

### Dashboard layout: Ongoing

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
- **Tốc độ hoàn thành**: đếm action done/tuần kể từ review trước (đọc `log/` dòng `status: doing > done` theo ngày, hoặc `completed_date`). Xem `../okr-shared/references/metrics.md` "Tốc độ hoàn thành".

## Mode trace (xem lại lịch sử, read-only)

Mode đọc dữ liệu lịch sử khi user muốn "xem lại / trace / history". Vẫn read-only (analyze không bao giờ ghi). Khác default/deep: đọc `actions/archive/` + `log/` thay vì state active, theo nguyên tắc **lazy loading** (frontmatter trước, body khi user yêu cầu). 4 kiểu trace (action / milestone / theo thời gian / log) + quy trình chi tiết: xem `references/flow-trace.md`.

Trigger: "trace A003", "xem lại milestone M1", "actions done tháng 4", "xem log tuần trước".

## Reachability audit (read-only, chống file mồ côi)

Backstop cho "Reachability khi ghi" (canonical `../okr-shared/references/preload.md`). Read-only: CHỈ báo + đề xuất; việc neo do `okr-init`/`okr-plan`/`okr-track` làm (giữ SOT ownership, đúng "track đề xuất, init/plan áp dụng").

### Light (dashboard, mỗi phiên có plan)

Tái dùng kết quả `ls -1 .okr/` (đã chạy ở Preload Contract Bước 0). Chỉ soi CẤP 1, KHÔNG đệ quy, KHÔNG đọc link-set, KHÔNG khẳng định mồ côi. Whitelist đã biết ở root:

- File: `objective.md`, `resources.md`, `plan.md`.
- Thư mục: `actions`, `inbox`, `log`, `lessons`, `context`.

Bất kỳ file lẻ / thư mục con NGOÀI whitelist trên:

````
ℹ️ Có file/thư mục ở vị trí lạ: <tên>. Chạy review sâu để xác minh + cách neo.
````

Giữ claim token-cheap (1 lệnh ls cấp 1, output ngắn).

### Deep / closure (thuật toán reachable set đầy đủ)

`closure` ở đây là pha tái dùng thuật toán trong flow khác, không phải mode riêng của `okr-analyze`.

**Bước 1. Liệt kê thực tế:** `ls -R .okr/` (đệ quy). Chỉ lấy TÊN file, KHÔNG đọc body của `archive/`, `log/` (tôn trọng invisible-by-default: audit chỉ kiểm tồn tại path).

**Bước 2. Tập reachable theo vị trí:** 3 file SOT đơn ở root + các thư mục cấu trúc đã biết (`actions/`, `actions/archive/`, `inbox/`, `log/`, `lessons/`, `context/`). File theo độ sâu chuẩn của từng thư mục = reachable theo vị trí. Riêng `context/` vẫn phải qua Bước 5 để kiểm entry index, dù không bị gọi là mồ côi ở Bước 4.

**Bước 3. Tập reachable qua link/đăng ký** (bắt file ngoài thư mục đã biết + để check link chết):
- Cột `Resource` của `resources.md` `## Tài liệu & Knowledge Base`.
- Dòng `Path:` trong `## Output/Deliverable` của action ACTIVE **và** action trong `actions/archive/` (deep/closure vốn đọc archive).
- `context/index.md` entries (cột `Path`).
- Link trong `lessons/index.md` của mục còn hiệu lực.
- Link lesson `ported/obsolete` đọc thành tập riêng chỉ để phân loại cảnh báo nhẹ ở Bước 6, KHÔNG đưa vào reachable set.
- Roadmap link trong `plan.md` body (đọc on-demand) để check Roadmap link chết.

**Bước 4. Mồ côi = file thực tế KHÔNG thuộc Bước 2 và KHÔNG thuộc Bước 3.** Điển hình: file ở `.okr/` root ngoài 3 SOT, hoặc trong thư mục con lạ (vd `.okr/research/`). Báo path + gợi ý neo theo bản đồ neo (tài liệu DÙNG → resources; cross-cutting → context/).

**Bước 5. Kiểm tra nội bộ context/:** nếu `context/` tồn tại mà thiếu `context/index.md` → báo "context lệch". Mọi `context/<slug>.md` (trừ `index.md`) phải có entry trong `index.md`; mọi entry phải trỏ file tồn tại. Lệch → báo "context lệch".

**Bước 6. Kiểm tra link chết:** mọi path Bước 3 và tập lesson `ported/obsolete` riêng phải trỏ file tồn tại.
- Roadmap link trỏ `actions/*` không tồn tại, entry context trỏ file đã xóa, cột Resource trỏ file `.okr/` không tồn tại → **lỗi cứng**.
- File NGOÀI `.okr/` không tồn tại (deliverable ngoài chưa tạo), lesson ported/obsolete đã xóa khỏi `.okr/` (hàng đợi port hợp lệ) → **cảnh báo nhẹ**, không lỗi cứng.

Output mỗi vấn đề: path + loại (mồ côi / link chết / context lệch) + gợi ý neo + skill áp dụng. KHÔNG tự sửa.

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

  
