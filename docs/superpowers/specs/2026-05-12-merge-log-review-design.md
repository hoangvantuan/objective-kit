# Gộp Log + Review: Design Spec

## Mục tiêu

Gộp `log/` và `log/reviews/` thành 1 folder `log/` duy nhất. Phân biệt loại entry bằng frontmatter `type` (array). Đơn giản hoá rule đọc. Bỏ trùng lặp khi deep mode ghi cả 2 file.

## Bối cảnh

Hiện tại `okr-track` ghi 2 loại log vào 2 folder:

- `log/YYYY-MM-DD.md`: tracking entries (light mode). Ghi sự kiện thay đổi.
- `log/reviews/YYYY-MM-DD.md`: review entries (deep/closure mode). Ghi phân tích + đề xuất.

Vấn đề:

1. **Trùng lặp**: deep mode ghi 1 review + 1 log entry (tóm tắt + link). Log entry gần như vô dụng.
2. **Rule đọc phức tạp**: 5 rule khác nhau cho 5 mode, phân biệt 2 folder.
3. **Log giữ vai trò audit trail**: cross-check SOT có update đúng không. Giá trị này cần giữ.

## Thiết kế

### Cấu trúc folder mới

```
.okr/log/
  2026-05-10.md   type: [tracking]
  2026-05-12.md   type: [tracking, review]
  2026-05-15.md   type: [tracking]
  2026-05-18.md   type: [review]
  2026-12-31.md   type: [closure]
```

Bỏ hẳn `log/reviews/` subfolder.

### Schema file log

```yaml
---
date: YYYY-MM-DD
type: [tracking]          # array, giá trị hợp lệ: tracking | review | closure
---
```

`type` là array vì cùng ngày có thể có light sáng (tracking) + deep chiều (review) → `type: [tracking, review]`.

### Body theo type

**tracking** (từ light hoặc deep Bước 1 update progress):

```markdown
## Thay đổi
- KR1.current: 40 > 50
- A003.status: doing > done

## Ghi chú
- A005 vẫn blocked
```

**review** (từ deep): gồm cả `## Thay đổi` (nếu deep có update progress) + phân tích.

```markdown
## Thay đổi
- KR1.current: 40 > 50

## Tổng kết
(bảng KR: Target, Current, %, Trend)

## Phân tích
### Đạt tốt
### Cần cải thiện (root cause ≥3 lần "tại sao")

## Đề xuất điều chỉnh
(bảng: #, đề xuất, lý do, skill áp dụng, đã apply?)
```

**closure** (từ closure): giống review + thêm section `## Lessons`.

```markdown
## Thay đổi
...
## Tổng kết
...
## Phân tích
...
## Đề xuất điều chỉnh
...
## Lessons
(chỉ closure mới có section này)
```

### Append rule

- Cùng ngày, cùng type: append section mới `## Thay đổi (lần N, HH:MM)`.
- Cùng ngày, khác type (light sáng + deep chiều): append deep sections vào file đã có, update frontmatter `type` thành union (vd `[tracking]` → `[tracking, review]`).

### Rule đọc log

| Mode                | Đọc log                 |
| ------------------- | ----------------------- |
| Orchestrator `/okr` | Không đọc               |
| Light               | Không đọc               |
| Deep                | Adaptive (xem bên dưới) |
| Closure             | Tất cả files            |
| Trace               | Lazy theo yêu cầu user  |


**Deep adaptive rule:**

1. Scan `log/` sorted desc by filename (date).
2. Tìm file deep gần nhất = file có `review` hoặc `closure` trong `type` array.
3. **Nếu file mới nhất đã là deep**: đọc 3 file deep gần nhất (để nắm xu hướng).
4. **Nếu file mới nhất là light**: đọc tất cả light từ deep cuối đến nay + 3 file deep gần nhất.

Mục đích: deep luôn nắm (a) xu hướng dài hạn qua 3 review trước, (b) mọi thay đổi nhỏ kể từ review trước.

**Light không đọc log**: light là quick update, không cần historical context. SOT (objective.md, plan.md, actions/) đã đủ.

### Tác động đến các file

| File                                  | Thay đổi                                                                                                                                                                                         | Effort |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ |
| `okr/SKILL.md`                        | Bước 1: bỏ đọc `log/reviews/`, sửa thành "KHÔNG đọc log"                                                                                                                                         | xs     |
| `okr-track/SKILL.md`                  | Phase 1: đổi rule đọc (bỏ phân biệt log vs review). Phase 4a: ghi `log/` (giữ nguyên). Phase 4b Bước 7: ghi `log/` thay vì `log/reviews/`, bỏ append log entry riêng. Closure: đọc tất cả `log/` | s      |
| `okr-track/references/data-format.md` | Gộp schema 2 loại log thành 1. Sửa Log Reading Rules. Bỏ section `log/reviews/` riêng                                                                                                            | s      |
| `CLAUDE.md`                           | Sửa cây thư mục: bỏ `reviews/` subfolder                                                                                                                                                         | xs     |
| `okr-plan/SKILL.md`                   | Không ảnh hưởng                                                                                                                                                                                  | -      |
| `okr-init/SKILL.md`                   | Không ảnh hưởng                                                                                                                                                                                  | -      |
| `okr-capture/SKILL.md`                | Không ảnh hưởng                                                                                                                                                                                  | -      |


### Không làm

- Không migrate dữ liệu cũ (`log/reviews/` hiện có giữ nguyên, schema mới chỉ cho `.okr/` tạo mới)
- Không đổi format nội dung (giữ heading structure hiện tại)
- Không đổi logic phân tích root cause hay đề xuất
- Không đổi archive rules hay inbox processing

## Verification

1. Grep toàn bộ `skills/` tìm `log/reviews` → phải trả về 0 kết quả (trừ legacy migration note nếu có).
2. Kiểm tra `okr-track/SKILL.md` không còn đề cập `log/reviews/` trong flow ghi file.
3. Kiểm tra `okr/SKILL.md` Bước 1 không đọc log.
4. Kiểm tra `CLAUDE.md` cây thư mục không còn `reviews/`.
5. Test flow: light → ghi `log/YYYY-MM-DD.md` type [tracking]. Deep cùng ngày → append, type [tracking, review]. Deep ngày mới → ghi file mới type [review].
