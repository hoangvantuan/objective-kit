---
name: okr-resource
description: "Quản lý tài nguyên, vai trò và trách nhiệm trong dự án. Mapping resource vào actions, phát hiện thiếu hụt và xung đột. Trigger: /okr-resource"
---

# okr-resource: Quản lý tài nguyên & vai trò

Tạo/cập nhật `.okr/resources.md`. Mapping nhân sự, công cụ, ngân sách vào actions cụ thể.

## Điều kiện tiên quyết

- `.okr/plan.md` và `.okr/actions/` phải tồn tại. Nếu chưa, hướng user chạy `/okr-plan`.

## Checklist

1. Đọc `.okr/plan.md` và frontmatter tất cả `.okr/actions/*.md`
2. Liệt kê actions chưa có PIC
3. Hỏi user về nhân sự: tên, vai trò, trách nhiệm, khả dụng
4. Mapping PIC vào actions (cập nhật frontmatter `pic` trong action files)
5. Hỏi user về công cụ, tài liệu, ngân sách
6. Kiểm tra xung đột (đọc `references/role-matrix.md` phần phát hiện xung đột)
7. Tạo/cập nhật `.okr/resources.md`

## Format output

Đọc `references/data-format.md` để lấy schema cho `resources.md`.

## Quy tắc

- Khi cập nhật PIC, sửa cả frontmatter trong `actions/*.md` (SOT)
- Mỗi action có deadline mà PIC khả dụng < 50% cần cảnh báo
- Nếu phát hiện xung đột, đề xuất giải pháp cụ thể (dời deadline, tách task, thêm người)
- File resources.md là SOT: luôn phản ánh trạng thái mới nhất, không chứa lịch sử
