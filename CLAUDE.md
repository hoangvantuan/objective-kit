# Objective Kit

Bộ skill quản lý mục tiêu theo OKR. Mỗi thư mục `.okr/` chứa đúng **1 objective**.

## Entry point

User chỉ làm việc với **`/okr`** (orchestrator). Skill tự đánh giá trạng thái `.okr/` và kích hoạt skill con phù hợp. Không cần nhớ tên skill con.

Chi tiết routing, status dashboard, keyword mapping: xem `skills/okr/SKILL.md`.

## Skill con (4)

| Skill | Vai trò | Sub-mode |
|-------|---------|----------|
| `okr-init` | SOT objective + resource | `new`, `update-objective`, `update-resource` |
| `okr-plan` | SOT plan + actions | `new`, `update` |
| `okr-track` | Progress + review + inbox | `light`, `deep`, `closure`, `inbox-only` |
| `okr-capture` | Thu thập nhanh → inbox | n/a |

Tất cả skill con được kích hoạt qua orchestrator `/okr`. Không trigger trực tiếp.

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
├── inbox/             # Capture items chờ xử lý      (okr-capture → okr-track)
└── log/               # Append-only                  (okr-track)
```

Mỗi skill chứa schema chi tiết trong `references/` của riêng nó.

## Nguyên tắc

1. **1 objective / `.okr/`**: Mỗi thư mục chỉ chứa 1 mục tiêu. Muốn nhiều mục tiêu → tạo nhiều thư mục project.
2. **Single entry point**: User chỉ gọi `/okr`.
3. **Confirm trước khi ghi**: `init` và `plan` bắt buộc có bảng tóm tắt + xác nhận.
4. **SOT vs Log vs Inbox**: SOT ghi đè. Log append-only. Inbox status transition.
5. **Track đề xuất, init/plan áp dụng**: Track không sửa cấu trúc, chỉ progress fields.
