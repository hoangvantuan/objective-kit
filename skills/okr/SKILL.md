---
name: okr
description: "Entry point chính cho mọi tương tác OKR. LUÔN dùng skill này đầu tiên khi user nhắc đến mục tiêu, dự án, OKR, tracking, kế hoạch, tài nguyên, capacity, skill, inbox, capture, ghi nhanh, hoặc gọi `/okr`. Skill tự đánh giá trạng thái `.okr/` hiện tại và chủ động kích hoạt skill phù hợp (init/plan/track/capture) với context user cung cấp. User KHÔNG cần biết tên skill con, chỉ làm việc với `/okr`."
---

# okr: Orchestrator quản lý OKR

Skill điều phối trung tâm. User mặc định luôn vào skill này. Phân tích state + intent → kích hoạt skill phù hợp + truyền context.

## Nguyên tắc cốt lõi

1. **Single entry point**: User chỉ gọi `/okr` (kèm hoặc không kèm câu lệnh tự nhiên). Không cần nhớ tên skill con.
2. **Tự kích hoạt**: Sau khi xác định phase, gọi thẳng skill con (đọc SKILL.md tương ứng và thực thi), KHÔNG bắt user gõ thêm lệnh.
3. **Truyền context**: Mọi câu chữ user nói khi gọi `/okr` phải forward sang skill con (vd: "/okr tôi muốn làm app X" → kích hoạt `okr-init` mode `new` với context dự án app X).
4. **Không tự ghi file**: Skill này CHỈ phân tích, route, gọi skill con. Mọi thay đổi file do skill con xử lý (kèm phase confirm với user).

## Skill con (4)

| Skill         | Vai trò                                                                                   | Sub-mode                                          |
| ------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `okr-init`    | Quản lý SOT objective + resource                                                          | `new`, `update-objective`, `update-resource`      |
| `okr-plan`    | Quản lý SOT plan + actions                                                                | `new`, `update`                                   |
| `okr-track`   | Đánh giá state + cập nhật progress + xử lý inbox + đề xuất + delegate điều chỉnh cấu trúc | `light`, `deep`, `closure`, `inbox-only`, `trace` |
| `okr-capture` | Thu thập nhanh ý tưởng/ghi chú → inbox                                                    | n/a                                               |


## Phân vai SOT

Mỗi field SOT chỉ được sửa bởi đúng 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa, delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| KR.current, KI.current, action.status, plan counters              | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |

> Chi tiết: `references/sot-ownership.md` (load cùng skill này).

## Flow xử lý

### Bước 1: Đọc trạng thái `.okr/`

**Preload SOT data**: Orchestrator đọc sẵn frontmatter + body thiết yếu của 3 file SOT dưới đây. Context này available cho mọi skill con khi được kích hoạt, skill con KHÔNG cần đọc lại.

Đọc song song nếu file tồn tại:

- `.okr/objective.md` (frontmatter + KR/KI)
- `.okr/plan.md` (frontmatter)
- `.okr/resources.md` (frontmatter)
- glob `.okr/actions/*.md` (chỉ frontmatter, **không đệ quy**, bỏ qua `actions/archive/`)
- glob `.okr/inbox/*.md` có status=pending (đếm)
- `.okr/log/`: **KHÔNG đọc**. Log thô không cần cho state assessment.
- **Chỉ 1 file mới nhất** trong `.okr/log/reviews/` (sorted by filename desc, lấy đầu tiên). KHÔNG đọc review cũ hơn.

### Bước 2: Hiển thị status ngắn (luôn luôn)

```
Trạng thái OKR
  Objective : [tên hoặc "(chưa có)"]
  Type      : [project / ongoing]
  Period    : [start > end hoặc "review cycle: weekly"]
  KR/KI     : X/Y đạt | Y total [hoặc N healthy / M warning / K critical]
  Actions   : a done / b doing / c blocked / d pending
  Resource  : [đã có (capacity Xh/tuần, N skills, M tool) / thiếu Solo Profile / thiếu capacity]
  Inbox     : N items chưa xử lý
  Log gần   : YYYY-MM-DD ([X ngày trước])
```

**Gợi ý tiếp theo** (1 dòng, render sau block status):

| Tín hiệu (ưu tiên từ trên xuống) | Gợi ý |
|-----------------------------------|-------|
| Action quá hạn | "Có N action quá hạn. Update status hoặc dời deadline qua track/plan update." |
| KR at-risk | "KR<N> at-risk. Chạy deep review để phân tích." |
| Inbox ≥5 items | "Inbox có N items. Xử lý qua track hoặc inbox-only." |
| Mọi action done | "Tất cả action done. Chạy closure để chốt." |
| Default | "Chạy track light để cập nhật tiến độ." |

### Bước 3: Suy luận next action

Bảng routing dưới đây gộp 2 nguồn tín hiệu: (a) **State** = trạng thái `.okr/` đọc được ở Bước 1, (b) **Keyword** = từ khoá user nói khi gọi `/okr`. Đọc theo thứ tự State trước, Keyword sau. Nếu State + Keyword chỉ về cùng skill → đi thẳng. Nếu mâu thuẫn → ưu tiên Keyword (user đang nói rõ ý) và xác nhận lại ở Bước 4.

| Trigger (State hoặc Keyword)                                                                       | Skill         | Mode               |
| -------------------------------------------------------------------------------------------------- | ------------- | ------------------ |
| State: `.okr/` chưa có HOẶC `objective.md` thiếu                                                   | `okr-init`    | `new`              |
| Keyword: "khởi tạo / mục tiêu mới / dự án mới"                                                     | `okr-init`    | `new`              |
| Keyword: "đổi target / sửa KR / sửa KI / đổi ngưỡng / đổi deadline objective / objective / period" | `okr-init`    | `update-objective` |
| Keyword: "tài liệu / tài nguyên / capacity / skill / công cụ / ngân sách / Solo Profile"           | `okr-init`    | `update-resource`  |
| State: có objective + resource, thiếu `plan.md`                                                    | `okr-plan`    | `new`              |
| Keyword: "kế hoạch / plan / lộ trình / milestone mới" (chưa có `plan.md`)                          | `okr-plan`    | `new`              |
| Keyword: "thêm action / sửa action / dời deadline / xoá action / sửa milestone"                    | `okr-plan`    | `update`           |
| State: đủ SOT + actions còn mở + keyword tiến độ/cập nhật/xong/blocked                             | `okr-track`   | `light`            |
| Keyword: "review / tổng kết / lookback / đánh giá sâu"                                             | `okr-track`   | `deep`             |
| State: mọi action `done`                                                                           | `okr-track`   | `closure`          |
| Keyword: "thêm nhanh / ghi lại / note / capture / nhớ cái này / inbox / add"                       | `okr-capture` | n/a                |
| Keyword: "trace / xem lại / history / lịch sử / xem action cũ / xem log cũ"                        | `okr-track`   | `trace`            |
| Keyword: "inbox / xử lý inbox" (skip update progress)                                              | `okr-track`   | `inbox-only`       |

### Bước 4: Xác nhận với user (1 câu)

```
> Đề xuất: chạy [skill] mode [mode] vì [lý do ngắn]. Tiếp tục? (y/sửa)
```

User reply `y` → kích hoạt skill con. User reply context khác → re-route.

### Bước 5: Kích hoạt skill con

Đọc `skills/<skill-con>/SKILL.md` rồi thực thi theo hướng dẫn của skill đó. Truyền context user đã nói khi gọi `/okr` vào prompt khởi đầu của skill con.

## Khi user gọi `/okr <subcommand>` rõ tên

| Lệnh                                             | Hành động                                                         |
| ------------------------------------------------ | ----------------------------------------------------------------- |
| `/okr init`                                      | Bỏ qua state-check, gọi `okr-init` (mode tự detect)               |
| `/okr init new`                                  | `okr-init` mode `new` (cảnh báo nếu sẽ ghi đè)                    |
| `/okr init update-objective`                     | `okr-init` mode `update-objective`                                |
| `/okr init update-resource`                      | `okr-init` mode `update-resource`                                 |
| `/okr plan`                                      | Gọi `okr-plan` (mode tự detect)                                   |
| `/okr plan new`                                  | `okr-plan` mode `new` (cảnh báo nếu sẽ ghi đè)                    |
| `/okr plan update`                               | `okr-plan` mode `update`                                          |
| `/okr resource`                                  | Alias cho `okr-init` mode `update-resource`                       |
| `/okr track`                                     | Gọi `okr-track` (mode tự detect)                                  |
| `/okr track light|deep|closure|inbox-only|trace` | Gọi `okr-track` mode tương ứng                                    |
| `/okr capture` hoặc `/okr add`                   | Gọi `okr-capture`                                                 |
| `/okr inbox`                                     | Gọi `okr-track` chỉ xử lý inbox (skip update progress)            |
| `/okr trace <ID>`                                | Gọi `okr-track` mode `trace`, truyền ID (vd: A003, M1)            |
| `/okr history`                                   | Gọi `okr-track` mode `trace`, hiển thị danh sách archive + log cũ |
| `/okr status`                                    | Chỉ hiển thị status (Bước 2), không kích hoạt skill               |


## Quy tắc

- KHÔNG tạo/sửa file. Chỉ đọc + route.
- Đọc tối thiểu cần thiết: frontmatter trước, body chỉ khi cần.
- Status hiển thị bắt buộc trước mọi đề xuất.
- State mơ hồ → hỏi user thay vì đoán.
- Sau khi skill con xong, quay về `okr` để hiển thị status mới + đề xuất bước tiếp theo.
