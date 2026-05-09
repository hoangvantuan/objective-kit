# Task Format: Template action file

## Tạo file action

Mỗi action = 1 file trong `.ok/actions/`. Tên file: `AXXX-slug.md` (XXX = số thứ tự 3 chữ số, slug = tên ngắn dùng dấu gạch ngang).

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
---
```

```markdown
## Definition of Done

- [ ] [Tiêu chí 1: cụ thể, verify được]
- [ ] [Tiêu chí 2]

## Output/Deliverable

[Mô tả sản phẩm đầu ra khi task hoàn thành. Phải cụ thể: file gì, ở đâu, chứa gì]

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

| Value | Thời gian ước lượng |
|-------|-------------------|
| xs | < 1 giờ |
| s | 1-4 giờ |
| m | 1-2 ngày |
| l | 3-5 ngày |
| xl | > 1 tuần |
