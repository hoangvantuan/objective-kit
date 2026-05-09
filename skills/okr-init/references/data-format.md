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

### Habit type

```yaml
---
type: habit
objective: "string"
start_date: YYYY-MM-DD
frequency: daily | weekly | monthly
status: active | paused
---
```

Body:
- `## Objective` (WHY)
- `## Key Indicators` (bảng: #, Chỉ số, Tần suất, Streak hiện tại, Streak dài nhất)
- `## Checklist lặp lại`

## resources.md (SOT)

```yaml
---
last_updated: YYYY-MM-DD
---
```

Body sections (giữ section header dù rỗng):
- `## Vai trò & Trách nhiệm` (bảng: Tên, Vai trò, Trách nhiệm, Actions, Khả dụng %)
- `## Công cụ & Tài liệu` (bảng: Tài nguyên, Mục đích, Status, Người quản lý)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách bullet)

Tại thời điểm `mode: new`, cột `Actions` trong section "Vai trò" để trống (plan chưa có). Mode `update-resource` sẽ map sau khi `okr-plan` tạo actions.

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
