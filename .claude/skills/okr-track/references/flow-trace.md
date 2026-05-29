# Trace Flow (mode trace)

Chạy khi user gọi `/okr trace <ID>`, `/okr history`, hoặc nhắc đến "xem lại", "lịch sử". **Không** update progress. **Không** đọc file active bình thường.

**Nguyên tắc lazy loading**: Đọc dần, không đọc hết. Frontmatter trước, body khi user yêu cầu.

## Trace 1 action cụ thể

Trigger: "trace A003", "xem lại A003"

1. Tìm file `actions/archive/A003-*.md`
2. Đọc frontmatter → hiển thị tóm tắt (id, title, milestone, status, pic, due_date, effort)
3. User muốn xem chi tiết → đọc body (DoD, Output, Tiêu chí chất lượng)

```
Archive: A003
  Title     : Spec MVP
  Milestone : M1: Research
  PIC       : An
  Due date  : 2026-10-20
  Effort    : m
  Status    : done

Xem chi tiết? (y/n)
```

## Trace milestone

Trigger: "trace M1", "xem lại milestone Research"

1. Đọc frontmatter `plan.md` → lấy info milestone done (name, target_date, status)
2. Lọc `actions/archive/*.md` có `milestone: "M1: Research"` (chỉ frontmatter)
3. Hiển thị bảng tóm tắt: ID, Title, PIC, Due date, Effort
4. User chọn action nào cần xem → đọc body file đó

## Trace theo thời gian

Trigger: "xem lại actions tuần trước", "actions done tháng 4"

1. Lọc `actions/archive/*.md` theo `due_date` trong frontmatter
2. Hiển thị danh sách phù hợp (chỉ frontmatter)
3. User drill-down khi cần

## Trace log

Trigger: "xem log tuần trước", "log tháng 4", "review lần trước"

1. Lọc `log/*.md` theo filename (chứa ngày). Filter theo frontmatter `type` nếu user chỉ muốn xem review.
2. Hiển thị danh sách ngày có log
3. User chọn ngày → đọc nội dung file đó
