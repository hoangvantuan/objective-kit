# Data Format: resources.md

Schema cho `.okr/resources.md`. SOT (Source of Truth), ghi đè khi cập nhật.

```yaml
---
last_updated: YYYY-MM-DD
---
```

Body chứa các sections:

- `## Vai trò & Trách nhiệm` (bảng: Tên, Vai trò, Trách nhiệm, Actions, Khả dụng)
- `## Công cụ & Tài liệu` (bảng: Tài nguyên, Mục đích, Status, Người quản lý)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách)

## Tham chiếu actions/

Skill đọc frontmatter từ `.okr/actions/*.md`. Cập nhật field `pic` khi assign:

```yaml
---
id: AXXX
pic: "Tên người"
---
```
