# Design: Output Tracking trong okr-track

## Vấn đề

Khi action hoàn thành, hệ thống chỉ ghi `status: doing > done`. Không ghi lại output thực tế (file path, link, text). User muốn check lại task đã done thì không biết output nằm ở đâu, phải tự đi tìm.

## Giải pháp

Mở rộng okr-track để ghi output thực tế vào `## Output/Deliverable` của action file. Ghi đè nội dung dự kiến (do okr-plan tạo lúc planning) bằng output thực. Đồng thời log output vào log file để tra cứu nhanh.

## Quyết định thiết kế

| Quyết định | Chọn | Lý do |
|-----------|------|-------|
| Output types | Bất kỳ (file, link, text, mixed) | User có output đa dạng, không cố định |
| Thời điểm ghi | Bất kỳ lúc nào + nhắc khi mark done | Linh hoạt, không bỏ sót |
| Vị trí lưu | Ghi đè `## Output/Deliverable` trong action file | Đơn giản, 1 chỗ duy nhất, không thêm section mới |
| Skill sở hữu | okr-track | Output là phần tự nhiên của tracking progress |
| Bắt buộc? | Không. Nhắc nhở, cho skip | Không ép, tránh friction |
| Log | Có. Append output vào log entry | Cross-check, tra cứu nhanh không cần mở action file |

## SOT ownership thay đổi

| Field | Hiện tại | Sau thay đổi |
|-------|----------|-------------|
| `## Output/Deliverable` (dự kiến) | okr-plan | okr-plan (tạo lần đầu) |
| `## Output/Deliverable` (thực tế) | không ai | okr-track (ghi đè khi có output) |

## Flow ghi output

### Cách 1: Chủ động ghi giữa chừng

User nói: "output A003: file report ở docs/report.md"

okr-track ghi đè section `## Output/Deliverable` của A003. Action không cần đang `done`, `doing` cũng ghi được.

### Cách 2: Nhắc khi mark done

1. okr-track chuyển action sang `done`
2. Kiểm tra section `## Output/Deliverable`: nếu chưa bị ghi đè (vẫn là mô tả dự kiến) thì hỏi: "Action này có output gì? (path, link, text... hoặc skip)"
3. User skip: mark done bình thường, section giữ nguyên mô tả dự kiến
4. User ghi: ghi đè section, rồi mark done

## Format output trong action file

Free-form markdown trong section `## Output/Deliverable`:

```markdown
## Output/Deliverable

- [report.md](docs/report.md)
- Google Sheets: https://docs.google.com/spreadsheets/d/xxx
- Kết luận: chi phí tăng 15%, cần cắt 2 feature
```

## Format output trong log file

Append vào log entry cùng lúc ghi vào action file:

```markdown
## Thay đổi
- A003.status: doing > done
- A003.output:
  - [report.md](docs/report.md)
  - https://docs.google.com/spreadsheets/d/xxx
```

## File cần sửa

1. `skills/okr/references/sot-ownership.md`: thêm dòng output cho okr-track
2. `skills/okr-track/SKILL.md`: thêm capability ghi output
3. `skills/okr-track/references/flow-light.md`: thêm bước nhắc output khi mark done
4. `skills/okr-track/references/flow-deep.md`: tương tự
5. `skills/okr-track/references/data-format.md`: thêm `output` vào schema log entry (section `## Thay đổi`)
6. `CLAUDE.md`: cập nhật bảng SOT

## Không thay đổi

- Schema frontmatter action (không thêm field mới vào YAML)
- okr-plan flow (vẫn tạo `## Output/Deliverable` dự kiến như cũ)
- okr-capture (không liên quan)
- okr-init (không liên quan)
