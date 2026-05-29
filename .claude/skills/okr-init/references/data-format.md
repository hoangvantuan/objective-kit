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

> **Practices không nằm ở `objective.md`**. Hành động lặp lại để duy trì KI nằm ở `plan.md` body section `## Practices`. Schema xem `../../okr-plan/references/data-format.md`.

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
- `## Solo Profile` (bảng 1 dòng: Tên, Capacity (giờ/tuần), Skills)
- `## Công cụ` (bảng 6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú)
- `## Tài liệu & Knowledge Base` (bảng 6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách bullet)

### Schema 6 cột thống nhất (Công cụ + Tài liệu/KB)

Cả 2 section dùng cùng 6 cột:

| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|

Ví dụ `## Công cụ`:

| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| Things 3 | Quản lý task hàng ngày | Daily planning, tracking | skill: things-mac | macOS app | Sync 2 chiều với OKR |
| Cursor | Code editor AI | Khi code, debug | Mở project → chat | cursor.sh | License Pro |

Ví dụ `## Tài liệu & Knowledge Base`:

| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| API Docs v2 | Reference backend endpoints | Khi tích hợp API | Mở link → search endpoint | https://docs.example.com | Cần VPN |
| Design System | UI components reference | Khi build frontend | Figma → Library → Components | figma.com/file/xxx | v2.1 |

Quy tắc cột "Cách dùng":

| Loại resource | Nội dung cột "Cách dùng" |
|--------------|-------------------------|
| Tool có tích hợp OKR | `skill: <tên-skill>` hoặc `mcp: <tên-mcp-server>` |
| Tool không tích hợp | Quy trình ngắn 1 dòng (vd: "Mở project → chat") |
| Tài liệu online | Cách truy cập (vd: "Mở link → search endpoint") |
| Tài liệu local | Path hoặc cách mở (vd: "GDrive → Folder meetings") |
| Knowledge base | Cách query (vd: "skill: deepwiki → search topic") |

### Migration schema cũ → 6 cột

`okr-init` mode `update-resource` lần đầu gặp schema cũ:

1. Detect bảng Công cụ có 4 cột (Tên công cụ, Khi nào dùng, Mục đích, Resource) → schema cũ.
2. Auto-migrate: map cột cũ → cột mới, thêm 2 cột trống (Cách dùng, Ghi chú). Báo user điền sau.
3. Detect bảng KB có 4 cột (Tên/Loại, Vị trí, Mục đích, Status) → map: Tên/Loại → Tên, Vị trí → Resource, Mục đích giữ, Status → gộp vào Ghi chú. Thêm Khi nào dùng + Cách dùng trống.
4. Không mất dữ liệu, chỉ restructure cột.

Skill OKR phục vụ persona **solo only** (1 user, 1 objective). Mọi action mặc định `pic: self`. Section `## Solo Profile` chứa duy nhất 1 dòng (chính user). KHÔNG còn các field "Liên lạc", "Vai trò", "Khả dụng %", "Actions" (bảng mapping PIC → action ID): đã bỏ vì không cần với solo. Mode `update-resource` chỉ sửa Solo Profile (capacity, skills) hoặc Công cụ/Tài liệu/Ngân sách.

> **Lưu ý dữ liệu cũ**: Nếu gặp `resources.md` từ schema cũ có section `## Nhân sự (Vai trò & Trách nhiệm)` hoặc cột "Người quản lý" / "Liên lạc", coi là legacy. Mode `update-resource` lần đầu sẽ migrate: tự lấy dòng đầu tiên (hoặc dòng có khả dụng cao nhất) làm Solo Profile, bỏ các cột không còn schema.

## Phát hiện xung đột (solo)

Bảng tín hiệu xung đột capacity/tài nguyên là canonical ở `../../okr-shared/references/metrics.md` (section "Capacity / xung đột tài nguyên"): quá tải, dồn deadline, skill gap, tool missing, capacity drop.

Áp dụng: `okr-init update-resource` khi hoàn tất; `okr-plan` khi check fit lúc tạo/sửa plan; `okr-analyze` khi phát hiện issue runtime. Mỗi cảnh báo kèm đề xuất giải pháp cụ thể (định nghĩa canonical ở `metrics.md`, không lặp ở đây).

## Quy tắc chung

- SOT: ghi đè khi cập nhật, không append. Lịch sử thay đổi đi vào log của `okr-track`.
- Section trống vẫn giữ header để mode `update-*` có chỗ chèn.
- Mọi sửa đổi `resources.md` phải cập nhật `last_updated`.
- Field `pic` trong frontmatter `actions/*.md` mặc định `self`. Có thể gán tên người khác khi delegate việc. Không cần sync 2 chiều với `resources.md`.
