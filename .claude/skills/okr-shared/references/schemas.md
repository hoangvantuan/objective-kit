# Shared Schemas

Schema dùng chung giữa nhiều skill. Load 1 lần qua orchestrator `okr`.

## Roadmap format

> Canonical. Các skill khác (okr-plan render, okr-track re-render) link về đây, không chép lại.

Mỗi milestone heading chứa bảng action bên dưới:

````markdown
## Roadmap

### M1: Research (target: 2026-05-20)

| ID | Task | Deadline | Priority | Notes |
|----|------|----------|----------|-------|
| [A001](actions/A001-research-market.md) | Nghiên cứu thị trường | 2026-05-15 | high | Cần xong trước A003 |
| [A002](actions/A002-competitor-analysis.md) | Phân tích đối thủ | 2026-05-18 | medium | |

### M2: Development (target: 2026-06-15)

| ID | Task | Deadline | Priority | Notes |
|----|------|----------|----------|-------|
| [A003](actions/A003-build-mvp.md) | Xây MVP | 2026-06-01 | critical | Depends: A001 |
| [A004](actions/A004-testing.md) | Viết test | 2026-06-10 | high | |
````

Cột bảng (5 cột, solo mode):

| Cột | Nguồn dữ liệu | Ghi chú |
|-----|---------------|---------|
| ID | `id` frontmatter, link tới file action | Relative link: `actions/AXXX-slug.md` |
| Task | `title` frontmatter | Hiển thị nguyên văn |
| Deadline | `due_date` frontmatter | Format YYYY-MM-DD |
| Priority | `priority` frontmatter | critical, high, medium, low |
| Notes | `notes` frontmatter (optional) | Không có thì cell trống |

Cột Assignee: hiển thị khi có ít nhất 1 action active với `pic` khác `self`. Vị trí: giữa Task và Deadline. Nguồn: `pic` frontmatter. Nếu mọi action `pic: self` → ẩn cột để gọn.

Quy tắc sinh bảng:

- **Ai render**: `okr-plan` (khi tạo/update action) và `okr-track` (khi update status/archive).
- **Sắp xếp**: Trong mỗi milestone, sắp theo Priority (critical > high > medium > low), rồi Deadline (sớm trước).
- **Lọc**: Chỉ hiển thị action chưa done. Action done đã archive, không xuất hiện trong bảng.
- **SOT**: Action files là SOT. Bảng là view được sinh lại mỗi lần update. Không sửa bảng trực tiếp.
- **Ongoing type**: Ongoing có thể có action không thuộc milestone nào. Nhóm dưới heading `### Chưa phân milestone`. Nếu Ongoing không có action nào, section `## Roadmap` chỉ chứa `## Practices` (không có bảng).

Ví dụ Ongoing Roadmap (có actions):

````markdown
## Practices

### P1: Tập thể dục
- frequency: weekly
- target_count: 3
- current_streak: 5
- ki_link: KI1

## Roadmap

### Chưa phân milestone

| ID | Task | Deadline | Priority | Notes |
|----|------|----------|----------|-------|
| [A001](actions/A001-mua-do-tap.md) | Mua đồ tập gym | 2026-05-20 | medium | Cải thiện KI1 |
````

---

## Inbox Aging

> Canonical. Các skill khác (okr-capture, okr-track) link về đây, không chép lại.

Trường `staleness_days = today - captured_at` được compute **on-the-fly** khi đọc, KHÔNG lưu vào frontmatter.

Ngưỡng:

| Phạm vi | Nghĩa | Hành vi của okr-track |
|---------|-------|------------------------|
| `staleness_days ≤ 7` | Mới | Hiển thị bình thường, không cảnh báo |
| `7 < staleness_days ≤ 30` | Đang chờ xử lý | Hiển thị bình thường, sort lên đầu nếu nhiều items |
| `staleness_days > 30` | Cũ | Cảnh báo "Inbox cũ ≥30 ngày". Không auto-discard. User quyết định. |

Phạm vi áp dụng: chỉ items có `status: pending`.

---

## Inbox type → Delegate mapping

> Canonical. Các skill khác (okr-track xử lý inbox) link về đây, không chép lại.

| Inbox type | Track tự xử lý? | Delegate sang |
|------------|-----------------|---------------|
| `action` | Không | `okr-plan` mode `update` (tạo action file) |
| `blocker` | Có (sửa action.status = blocked) | - |
| `resource` | Không | `okr-init` mode `update-resource` |
| `thought` → action | Không | `okr-plan` mode `update` |
| `thought` → log | Có (append log) | - |
| `thought` → giữ | Có (không làm gì) | - |

Status transitions:

```
pending → processed   (đã xử lý: tạo action, update resource, ghi log...)
pending → discarded   (user quyết định bỏ)
pending → pending     (giữ inbox, chờ rõ hơn)
```

> **Migrate dữ liệu cũ**: File inbox `type: idea` hoặc `type: note` → coi như `type: thought`. `okr-track` Phase 5 tự normalize khi đổi `status: processed`.

Gom delegate: nhiều items cùng skill → 1 lần delegate.

---

## Archive Rules

> Canonical. Các skill khác (okr-track archive) link về đây, không chép lại.

### Trigger

Khi `okr-track` (light hoặc deep) đánh `status: done` cho action, **trong cùng lần track, sau phase confirm**.

### Flow

0. `okr-track` set `completed_date: <today>` vào frontmatter action trước khi dời (dùng cho tốc độ hoàn thành + closure timeline). Nếu action đã có `completed_date` thì giữ nguyên.
1. Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md` (tạo thư mục `archive/` nếu chưa có).
2. Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`.
3. Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter `plan.md` (với `status: done`).
4. Cập nhật counters frontmatter `plan.md` (`completed` +N).

### Invisible by Default

| Nguồn dữ liệu | Mặc định đọc | Khi nào đọc thêm |
|----------------|-------------|-------------------|
| `actions/*.md` | Có (chỉ active) | Luôn đọc |
| `actions/archive/` | **Không** | User trace hoặc mode closure |
| `log/` | **Không** | Deep adaptive rule, closure tất cả, trace lazy |

### Quy tắc theo skill

| Skill | Actions archive | Log cũ |
|-------|----------------|--------|
| Orchestrator `/okr` | Không đọc | Không đọc log |
| `okr-track` light | Không đọc | Không đọc log |
| `okr-track` deep | Không đọc | Adaptive rule (xem data-format.md) |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả log** (tổng kết) |
| `okr-analyze` trace | **Đọc archive** (lazy) | **Đọc log** (lazy, filter type) |
| `okr-plan` update | Không đọc, không sửa | Không đọc |

Archive file schema giống hệt `actions/AXXX-slug.md`. Archive files là **read-only**.
