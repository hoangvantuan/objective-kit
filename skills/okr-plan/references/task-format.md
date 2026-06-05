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
key_result: KR1  # Ongoing: ghi KI ID (vd KI1)
milestone: "Tên milestone"  # Ongoing: có thể để ""
status: pending
priority: high
effort: m
pic: ""  # Mặc định "self". Có thể gán tên người khác khi delegate việc
due_date: YYYY-MM-DD
completed_date: ""  # okr-plan để trống; okr-track set khi status→done
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

Path: [chỉ khi deliverable là FILE riêng: đường dẫn file, vd `docs/report.md` hoặc `.okr/context/...`. Không phải file riêng thì bỏ dòng Path]
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

> **Dòng `Path:` để reachability** (`../../okr-shared/references/preload.md` "Giữ path deliverable bền"): nếu deliverable là file riêng, dòng `Path: <đường dẫn>` đặt ở ĐẦU section. `okr-track` khi ghi đè output thực tế PHẢI giữ/cập nhật dòng này, không làm đứt link. Audit Deep dựa vào dòng `Path:`. Deliverable không phải file (vd "KR1 đạt 5k DAU") thì không có dòng `Path:`.

## Quy tắc đặt tên

- ID bắt đầu từ A001, tăng dần
- Slug dùng tiếng Việt không dấu hoặc tiếng Anh, dấu gạch ngang
- Mỗi task phải map vào đúng 1 Key Result (Project) hoặc Key Indicator (Ongoing)
- Project: mỗi task phải thuộc 1 milestone. Ongoing: milestone không bắt buộc (để trống hoặc gán tên gợi nhớ)

## Priority và Effort

Giá trị hợp lệ + quy tắc đặc biệt (xl Checkpoints): xem `action-guide.md`.
