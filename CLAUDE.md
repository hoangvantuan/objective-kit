# Objective Kit

Bộ skill quản lý mục tiêu theo OKR. Mỗi thư mục `.okr/` chứa đúng **1 objective**.

## Entry point

User chỉ làm việc với `**/okr**` (orchestrator). Skill tự đánh giá trạng thái `.okr/` và kích hoạt skill con phù hợp. Không cần nhớ tên skill con.

Chi tiết routing, status dashboard, keyword mapping: xem `skills/okr/SKILL.md`.

## Skill con (4)

| Skill         | Vai trò                   | Sub-mode                                          |
| ------------- | ------------------------- | ------------------------------------------------- |
| `okr-init`    | SOT objective + resource  | `new`, `update-objective`, `update-resource`      |
| `okr-plan`    | SOT plan + actions        | `new`, `update`                                   |
| `okr-track`   | Progress + review + inbox | `light`, `deep`, `closure`, `inbox-only`, `trace` |
| `okr-capture` | Thu thập nhanh → inbox    | n/a                                               |


Tất cả skill con được kích hoạt qua orchestrator `/okr`. Không trigger trực tiếp.

## Phân vai SOT

Mỗi field SOT chỉ được sửa bởi đúng 1 skill (`okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để apply).

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


> Bảng này là **single source**. Các file `skills/okr/SKILL.md`, `skills/okr-track/references/data-format.md`, `docs/okr-system-review.md` link sang đây.

## Hai loại mục tiêu

- **Project**: có deadline, đo bằng Key Results (baseline → target), đạt target rồi kết thúc.
- **Ongoing**: duy trì liên tục (giống lĩnh vực), đo bằng Key Indicators (ngưỡng tối thiểu), không "xong".

Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI cụ thể.

## Dữ liệu

```
.okr/
├── objective.md       # SOT mục tiêu + KR/KI       (okr-init)
├── resources.md       # SOT người + tool + ngân sách (okr-init)
├── plan.md            # SOT milestones + counters    (okr-plan)
├── actions/           # SOT 1 file/action            (okr-plan + okr-track)
│   └── archive/       # Actions done (read-only)      (okr-track archive)
├── inbox/             # Capture items chờ xử lý      (okr-capture → okr-track)
└── log/               # Append-only                  (okr-track)
    └── reviews/       # Deep/closure reviews          (okr-track deep/closure)
```

Mỗi skill chứa schema chi tiết trong `references/` của riêng nó.

## Nguyên tắc

1. **1 objective / `.okr/**`: Mỗi thư mục chỉ chứa 1 mục tiêu. Muốn nhiều mục tiêu → tạo nhiều thư mục project.
2. **Single entry point**: User chỉ gọi `/okr`.
3. **Confirm trước khi ghi**: `init` và `plan` bắt buộc có bảng tóm tắt + xác nhận.
4. **SOT vs Log vs Inbox**: SOT ghi đè. Log append-only. Inbox status transition.
5. **Track đề xuất, init/plan áp dụng**: Track không sửa cấu trúc, chỉ progress fields.
6. **Archive + Log tiết kiệm token**: Actions done tự động archive vào `actions/archive/` (read-only, invisible by default). Log chỉ đọc file mới nhất. Trace khi cần xem lại dữ liệu cũ.
