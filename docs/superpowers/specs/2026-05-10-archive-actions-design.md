# Archive Actions Done: Tiết kiệm token bằng cách ẩn task hoàn thành

## Bối cảnh

Mỗi lần orchestrator `/okr` chạy, agent đọc toàn bộ `.okr/` (objective, plan, actions, resources, inbox, log). Actions đã `done` vẫn nằm trong `actions/`, agent vẫn đọc hết → tốn token không cần thiết.

**Mục tiêu**: Actions done trở thành invisible by default. Chỉ đọc khi user yêu cầu trace.

## Thiết kế

### 1. Cơ chế Archive

Khi `okr-track` (light hoặc deep) đánh `status: done` cho action, **trong cùng lần track, sau phase confirm**:

1. Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md`
2. Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`
3. Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter plan.md (với `status: done`)
4. Cập nhật counters frontmatter `plan.md` (`completed` +1)

### 2. Invisible by Default

Các skill đọc `.okr/` bỏ qua `actions/archive/`:

| Skill | Hành vi với archive |
|-------|-------------------|
| Orchestrator `/okr` | Glob `actions/*.md` (không đệ quy vào `archive/`). Không đếm archive |
| `okr-track` light | Chỉ đọc `actions/*.md`. Archive invisible |
| `okr-track` deep | Chỉ đọc `actions/*.md`. Archive invisible |
| `okr-track` closure | **Đọc archive** (cần tổng kết toàn bộ dự án) |
| `okr-plan` update | Chỉ thao tác `actions/*.md`. Không sửa archive |

### 3. Trace: đọc archive khi cần

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

### 4. Cấu trúc `.okr/` sau thay đổi

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

### 5. Ví dụ flow hoàn chỉnh

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

- Bước 1 (đọc trạng thái): glob `actions/*.md` thay vì đệ quy. Ghi rõ KHÔNG đọc `archive/`
- Bước 3 (suy luận): thêm keyword routing: "trace / xem lại / history / lịch sử" → `okr-track` mode trace
- Bảng lệnh: thêm `/okr trace <ID>` và `/okr history`

### `skills/okr-track/SKILL.md`

- Phase 1: ghi rõ glob `actions/*.md` (không đệ quy)
- Phase 4a (light): thêm bước archive sau ghi progress (dời file, xóa Roadmap, cập nhật counters)
- Phase 4b (deep): tương tự light cho phần archive
- Phase 4c (closure): đọc cả `actions/archive/` để tổng kết
- Thêm section mới: **Trace Flow** (lazy loading, 3 kiểu trace)

### `skills/okr-plan/SKILL.md`

- Mode UPDATE: ghi rõ chỉ thao tác `actions/*.md`, không sửa `archive/`
- Phase 4 (ghi file): không tạo file trong `archive/`

### `skills/okr-plan/references/data-format.md`

- Thêm section `## actions/archive/`: cùng schema với `actions/*.md`, chỉ khác vị trí
- Ghi rõ: archive files KHÔNG bị sửa sau khi archive (read-only)

### `skills/okr-track/references/data-format.md`

- Thêm section `## Archive Rules`: trigger, flow, invisible by default
- Thêm section `## Trace Flow`: lazy loading, 3 kiểu trace

### `CLAUDE.md`

- Cập nhật sơ đồ cấu trúc `.okr/`: thêm `actions/archive/`
- Thêm dòng giải thích: "Actions done tự động archive. Trace khi cần."
