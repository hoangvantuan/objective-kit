# Archive Actions Done: Tiết kiệm token bằng cách ẩn task hoàn thành

## Bối cảnh

Mỗi lần orchestrator `/okr` chạy, agent đọc toàn bộ `.okr/` (objective, plan, actions, resources, inbox, log). Hai nguồn tốn token lớn nhất:

1. **Actions done** vẫn nằm trong `actions/`, agent vẫn đọc hết
2. **Log cũ** tích lũy theo thời gian, agent đọc hết mỗi lần track

**Mục tiêu**: Dữ liệu đã hoàn thành (actions done, log cũ) trở thành invisible by default. Chỉ đọc khi user yêu cầu trace.

## Thiết kế

### 1. Cơ chế Archive

Khi `okr-track` (light hoặc deep) đánh `status: done` cho action, **trong cùng lần track, sau phase confirm**:

1. Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md`
2. Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`
3. Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter plan.md (với `status: done`)
4. Cập nhật counters frontmatter `plan.md` (`completed` +1)

### 2. Invisible by Default

Các skill đọc `.okr/` bỏ qua `actions/archive/` và log cũ:

| Nguồn dữ liệu | Mặc định đọc | Khi nào đọc thêm |
|----------------|-------------|-------------------|
| `actions/*.md` | Có (chỉ active) | Luôn đọc |
| `actions/archive/` | **Không** | User trace hoặc mode closure |
| `log/` latest (1 file gần nhất) | Có (chỉ file mới nhất) | Luôn đọc |
| `log/` cũ hơn | **Không** | User trace theo ngày |
| `log/reviews/` latest | Có (chỉ file mới nhất) | Luôn đọc |
| `log/reviews/` cũ hơn | **Không** | User trace theo ngày |

Quy tắc theo skill:

| Skill | Actions archive | Log cũ |
|-------|----------------|--------|
| Orchestrator `/okr` | Không đọc | Chỉ latest log |
| `okr-track` light | Không đọc | Chỉ latest log (để so trend) |
| `okr-track` deep | Không đọc | Chỉ latest log + latest review |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả reviews** (tổng kết) |
| `okr-plan` update | Không đọc, không sửa | Không đọc |


### 3. Log: chỉ đọc mới nhất

Log tích lũy theo thời gian (mỗi lần track = 1 entry). Đọc hết log cũ là lãng phí token.

**Quy tắc**:
- Orchestrator `/okr` Bước 1: chỉ đọc **1 file mới nhất** trong `log/` (sorted by filename desc, lấy đầu tiên)
- `okr-track` Phase 1: chỉ đọc **1 file mới nhất** trong `log/` và **1 file mới nhất** trong `log/reviews/` (để so trend)
- Log cũ hơn: KHÔNG đọc, trừ khi user yêu cầu trace
- `okr-track` closure: đọc tất cả `log/reviews/` (cần tổng kết period)

Ví dụ: `log/` có 15 files từ 2026-04-01 đến 2026-05-10. Agent chỉ đọc `log/2026-05-10.md`. 14 files còn lại invisible.

### 4. Trace: đọc archive và log cũ khi cần

**Trigger**: User nói "trace", "xem lại", "history", "lịch sử".

**Nguyên tắc lazy loading**: Đọc dần, không đọc hết.

#### Trace 1 action cụ thể

Ví dụ: "trace A003"

1. Tìm file `actions/archive/A003-*.md`
2. Đọc frontmatter → hiển thị tóm tắt (id, title, milestone, status, pic, due_date)
3. User muốn xem chi tiết → đọc body (DoD, Output, Tiêu chí chất lượng)

#### Trace milestone

Ví dụ: "trace M1", "xem lại milestone Research"

1. Đọc frontmatter plan.md → lấy info milestone done (name, target_date, status)
2. Lọc `actions/archive/*.md` có `milestone: "M1: Research"` (chỉ frontmatter)
3. Hiển thị bảng tóm tắt: ID, Title, PIC, Due date, Effort
4. User chọn action nào cần xem → đọc body file đó

#### Trace theo thời gian

Ví dụ: "xem lại actions tuần trước", "actions done tháng 4"

1. Lọc `actions/archive/*.md` theo `due_date` trong frontmatter
2. Hiển thị danh sách phù hợp (chỉ frontmatter)
3. User drill-down khi cần

#### Trace log

Ví dụ: "xem log tuần trước", "log tháng 4", "review lần trước"

1. Lọc `log/*.md` hoặc `log/reviews/*.md` theo filename (chứa ngày)
2. Hiển thị danh sách ngày có log
3. User chọn ngày → đọc nội dung file đó

### 5. Cấu trúc `.okr/` sau thay đổi

```
.okr/
├── objective.md
├── resources.md
├── plan.md
├── actions/
│   ├── A005-marketing.md      ← active (doing/pending/blocked)
│   ├── A006-build-api.md      ← active
│   └── archive/               ← MỚI: chứa actions done
│       ├── A001-khao-sat.md
│       ├── A002-doi-thu.md
│       └── A003-spec-mvp.md
├── inbox/
└── log/
```

### 6. Ví dụ flow hoàn chỉnh

**Trước khi archive** (track light):

```
Dashboard: Coaching AI Program
Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending

Thay đổi sắp ghi
- A003.status: doing > done
- A004.status: doing > done
Xác nhận? (y/sửa/huỷ)
```

User chọn `y`, agent:

1. Ghi `status: done` vào frontmatter A003, A004
2. Dời `actions/A003-spec-mvp.md` → `actions/archive/A003-spec-mvp.md`
3. Dời `actions/A004-wireframe.md` → `actions/archive/A004-wireframe.md`
4. Xóa dòng A003, A004 khỏi Roadmap body trong plan.md
5. Kiểm tra: M1 còn action active không? Nếu không → xóa heading M1 khỏi Roadmap body, đánh `status: done` cho M1 trong frontmatter
6. Cập nhật counters: `completed: 6` (was 4)
7. Append log

```
Đã cập nhật + archived 2 actions (A003, A004).
M1: Research hoàn thành → archived khỏi Roadmap.
```

**Lần track tiếp**: Agent chỉ đọc 6 active actions thay vì 12. Tiết kiệm ~50% token cho phần actions.

**Khi trace**: User nói "trace A003"

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

## Files cần thay đổi

### `skills/okr/SKILL.md`

- Bước 1 (đọc trạng thái): glob `actions/*.md` (không đệ quy). Ghi rõ KHÔNG đọc `archive/`. Log chỉ đọc latest 1 file.
- Bước 3 (suy luận): thêm keyword routing: "trace / xem lại / history / lịch sử" → `okr-track` mode trace
- Bảng lệnh: thêm `/okr trace <ID>` và `/okr history`

### `skills/okr-track/SKILL.md`

- Phase 1: ghi rõ glob `actions/*.md` (không đệ quy). Log chỉ đọc latest 1 file `log/` + latest 1 file `log/reviews/`
- Phase 4a (light): thêm bước archive sau ghi progress (dời file, xóa Roadmap, cập nhật counters)
- Phase 4b (deep): tương tự light cho phần archive
- Phase 4c (closure): đọc cả `actions/archive/` + tất cả `log/reviews/` để tổng kết
- Thêm section mới: **Trace Flow** (lazy loading, 4 kiểu trace: action, milestone, thời gian, log)

### `skills/okr-plan/SKILL.md`

- Mode UPDATE: ghi rõ chỉ thao tác `actions/*.md`, không sửa `archive/`
- Phase 4 (ghi file): không tạo file trong `archive/`

### `skills/okr-plan/references/data-format.md`

- Thêm section `## actions/archive/`: cùng schema với `actions/*.md`, chỉ khác vị trí
- Ghi rõ: archive files KHÔNG bị sửa sau khi archive (read-only)

### `skills/okr-track/references/data-format.md`

- Thêm section `## Archive Rules`: trigger, flow, invisible by default
- Thêm section `## Log Reading Rules`: chỉ latest, cũ khi trace
- Thêm section `## Trace Flow`: lazy loading, 4 kiểu trace (action, milestone, thời gian, log)

### `CLAUDE.md`

- Cập nhật sơ đồ cấu trúc `.okr/`: thêm `actions/archive/`
- Thêm nguyên tắc: "Actions done tự động archive. Log chỉ đọc mới nhất. Trace khi cần."
