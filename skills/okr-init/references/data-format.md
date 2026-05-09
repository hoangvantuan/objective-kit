# Data Format: objective.md + resources.md

`okr-init` tạo và cập nhật CẢ HAI file.

## objective.md (SOT)

### Project type

```yaml
---
type: project
objective: "string"
quarter: "Q1-2026"
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
status: active | completed | paused | cancelled
---
```

Body:
- `## Objective` (mô tả WHY chi tiết)
- `## Key Results` (bảng: #, Key Result, Baseline, Target, Current, Status)
- `## Context` (bối cảnh, ràng buộc)

KR Status: `pending` | `in-progress` | `achieved` | `missed`

### Ongoing type

Mục tiêu liên tục, không có deadline. Giống "lĩnh vực" cần duy trì (sức khoẻ, chất lượng code, tài chính...). Không bao giờ "xong", chỉ duy trì ở trạng thái khoẻ.

```yaml
---
type: ongoing
objective: "string"
start_date: YYYY-MM-DD
review_cycle: weekly | biweekly | monthly
status: active | paused | archived
---
```

Body:
- `## Objective` (WHY: tại sao lĩnh vực này quan trọng)
- `## Key Indicators` (bảng: #, Chỉ số, Ngưỡng tối thiểu, Current, Status)
- `## Practices` (danh sách hành động thường xuyên để duy trì)

KI Status: `healthy` | `warning` | `critical`

Phân biệt KR (Project) vs KI (Ongoing):
- KR có target cụ thể, đạt rồi thì done. KI có ngưỡng tối thiểu, duy trì liên tục.
- KR đo outcome một lần. KI đo trạng thái hiện tại so với ngưỡng.
- Ví dụ KR: "DAU 1k → 5k". Ví dụ KI: "Tập thể dục ≥3 lần/tuần" (ngưỡng = 3, current = 2 → warning).

## resources.md (SOT)

```yaml
---
last_updated: YYYY-MM-DD
---
```

Body sections (giữ section header dù rỗng):
- `## Nhân sự (Vai trò & Trách nhiệm)` (bảng: Họ Tên, Liên lạc (Zalo/FB/SĐT/Địa chỉ), Vai trò & Trách nhiệm, Ngày tham gia, Khả dụng %, Actions)
- `## Công cụ` (bảng: Tên công cụ, Khi nào dùng, Mục đích, Resource (URL/Account), Người quản lý)
- `## Tài liệu & Knowledge Base` (bảng: Tên/Loại, Vị trí (Link/Folder/File), Mục đích, Status)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách bullet)

Tại thời điểm `mode: new`, cột `Actions` trong section "Nhân sự" để trống (plan chưa có). Mode `update-resource` sẽ map sau khi `okr-plan` tạo actions.

## Tham chiếu actions/

Khi assign PIC trong mode `update-resource`, sửa frontmatter `.okr/actions/AXXX-*.md`:

```yaml
---
id: AXXX
pic: "Tên người"
---
```

Sync 2 chiều: `resources.md` (cột Actions của người đó) và frontmatter `pic` của action.

## Phát hiện xung đột

Áp dụng khi update resource hoặc khi `okr-plan` gọi vào để map PIC:

| Tín hiệu | Cảnh báo |
|----------|----------|
| Cùng PIC có ≥3 actions cùng deadline (±2 ngày) | Quá tải, đề xuất tách task hoặc dời |
| PIC khả dụng <50% có >5 actions trong period | Capacity không đủ, đề xuất thêm người |
| Action có deadline trước ngày PIC sẵn sàng | Bất khả thi, đề xuất dời deadline |
| Action không có PIC mà deadline <7 ngày | Cần assign gấp |
| Tool/tài liệu status `missing` mà có action phụ thuộc | Block, đề xuất bổ sung trước |

Mỗi cảnh báo phải kèm đề xuất giải pháp cụ thể: dời deadline, tách task, thêm người, hoặc giảm scope.

## Quy tắc chung

- SOT: ghi đè khi cập nhật, không append. Lịch sử thay đổi đi vào log của `okr-track`.
- Section trống vẫn giữ header để mode `update-*` có chỗ chèn.
- Mọi sửa đổi `resources.md` phải cập nhật `last_updated`.
- Update PIC PHẢI sync cả `resources.md` lẫn frontmatter `actions/*.md`.
