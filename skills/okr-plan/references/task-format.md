# Task Format: Template action file

## Tạo file action

Mỗi action = 1 file trong `.okr/actions/`. Tên file: `AXXX-slug.md` (XXX = số thứ tự 3 chữ số, slug = tên ngắn dùng dấu gạch ngang).

Ví dụ: `A001-thiet-ke-database.md`, `A002-viet-api-auth.md`

## Template

```yaml
---
id: AXXX
title: "Tên task rõ ràng"
description: "1-2 câu mô tả task cần làm"
key_result: KR1
milestone: "Tên milestone"
status: pending
priority: high
effort: m
pic: ""
due_date: YYYY-MM-DD
depends_on: []
notes: ""
external_ids: {}
---
```

```markdown
## Definition of Done

- [ ] [Tiêu chí 1: cụ thể, verify được]
- [ ] [Tiêu chí 2]

## Output/Deliverable

[Mô tả sản phẩm đầu ra khi task hoàn thành. Phải cụ thể: file gì, ở đâu, chứa gì]

## Tiêu chí chất lượng

[Output đạt khi nào? Đo bằng gì? Vd: "Report cover đủ 5 đối thủ, mỗi đối thủ có ≥3 điểm so sánh"]

## Checkpoints (BẮT BUỘC khi effort=xl, optional khác)

- [ ] [Mốc 1: phase/phần đầu xong vào ngày YYYY-MM-DD]
- [ ] [Mốc 2: phase/phần giữa xong vào ngày YYYY-MM-DD]
- [ ] [Mốc 3+ (optional, xoá nếu không dùng): phase/phần cuối xong vào ngày YYYY-MM-DD]

## Ghi chú

[Context bổ sung, ràng buộc, link tham khảo]
```

## Quy tắc đặt tên

- ID bắt đầu từ A001, tăng dần
- Slug dùng tiếng Việt không dấu hoặc tiếng Anh, dấu gạch ngang
- Mỗi task phải map vào đúng 1 Key Result
- Mỗi task phải thuộc 1 milestone

## Priority values

| Value | Khi nào dùng |
|-------|-------------|
| critical | Chặn toàn bộ dự án nếu không xong |
| high | Cần xong sớm, ảnh hưởng nhiều task khác |
| medium | Quan trọng nhưng không chặn ai |
| low | Làm khi rảnh, nice-to-have |

## Effort values

| Value | Thời gian ước lượng | Quy tắc đặc biệt |
|-------|---------------------|------------------|
| xs | < 1 giờ | - |
| s | 1-4 giờ | - |
| m | 1-2 ngày | - |
| l | 3-5 ngày | - |
| xl | > 1 tuần | **BẮT BUỘC body có `## Checkpoints` ≥2 mục, mỗi mục có ngày deadline cụ thể**. Track đọc Checkpoints để cảnh báo trượt giữa kỳ. Nếu không có Checkpoints, action không hợp lệ. |
