# Gộp Log + Review Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Gộp `log/` và `log/reviews/` thành 1 folder `log/` duy nhất, phân biệt bằng frontmatter `type` (array).

**Architecture:** Xoá mọi reference tới `log/reviews/` subfolder trong 7 files skill docs. Thay bằng unified schema dùng `type: [tracking | review | closure]` array trong frontmatter. Log reading rules chuyển sang adaptive rule dựa trên `type` array thay vì folder path.

**Tech Stack:** Markdown skill docs (không có code runtime)

---

## File Structure

| File | Thay đổi |
|------|----------|
| Modify: `CLAUDE.md:55-63` | Xoá `reviews/` subfolder khỏi cây thư mục |
| Modify: `skills/okr/SKILL.md:46-47` | Bước 1: bỏ đọc `log/reviews/`, sửa thành KHÔNG đọc log |
| Modify: `skills/okr-track/SKILL.md:33-35,176,185,231,242,257` | Phase 1 + Phase 4b + Phase 4c + Trace + Schema + Quy tắc: gộp references |
| Modify: `skills/okr-track/references/data-format.md` (toàn bộ) | Gộp schema 2 loại log thành 1 unified schema |
| Modify: `skills/okr-track/references/flow-deep.md:90,101,114,126,133` | Bước 7: ghi `log/` thay vì `log/reviews/`, bỏ append log entry riêng |
| Modify: `skills/okr-track/references/flow-closure.md:3` | Đọc tất cả `log/` thay vì `log/reviews/` |
| Modify: `skills/okr-track/references/flow-trace.md:48` | Bỏ `log/reviews/` reference |
| Modify: `skills/okr/references/shared-schemas.md:138-139,145-149` | Cập nhật bảng Invisible by Default + Quy tắc theo skill |

---

### Task 1: Unified log schema trong `data-format.md`

File trung tâm định nghĩa schema. Làm trước vì các file khác reference tới.

**Files:**
- Modify: `skills/okr-track/references/data-format.md`

- [ ] **Step 1: Đọc file hiện tại**

Đọc toàn bộ `skills/okr-track/references/data-format.md` để confirm nội dung hiện tại match context đã có.

- [ ] **Step 2: Rewrite title + mở đầu**

Thay:

```markdown
# Data Format: log/ và log/reviews/

Skill `okr-track` ghi 2 loại log tuỳ mode (light hay deep).
```

Bằng:

```markdown
# Data Format: log/

Skill `okr-track` ghi log vào `log/YYYY-MM-DD.md`. Phân biệt loại entry bằng frontmatter `type` (array).
```

- [ ] **Step 3: Gộp 2 schema (Mode LIGHT + Mode DEEP) thành 1 unified schema**

Xoá toàn bộ 2 section `## Mode LIGHT` (dòng 6-18) và `## Mode DEEP` (dòng 20-39).

Thay bằng section mới:

```markdown
## Schema file log: log/YYYY-MM-DD.md

### Frontmatter

~~~yaml
---
date: YYYY-MM-DD
type: [tracking]          # array, giá trị hợp lệ: tracking | review | closure
---
~~~

`type` là array vì cùng ngày có thể có light sáng (tracking) + deep chiều (review) → `type: [tracking, review]`.

### Body theo type

**tracking** (từ light hoặc deep Bước 1 update progress):

~~~markdown
## Thay đổi
- KR1.current: 40 > 50
- A003.status: doing > done

## Ghi chú
- A005 vẫn blocked
~~~

**review** (từ deep): gồm cả `## Thay đổi` (nếu deep có update progress) + phân tích.

~~~markdown
## Thay đổi
- KR1.current: 40 > 50

## Tổng kết
(bảng KR: Target, Current, %, Trend)

## Phân tích
### Đạt tốt
### Cần cải thiện (root cause ≥3 lần "tại sao")

## Đề xuất điều chỉnh
(bảng: #, đề xuất, lý do, skill áp dụng, đã apply?)
~~~

**closure** (từ closure): giống review + thêm section `## Lessons`.

~~~markdown
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
~~~

### Append rule

- Cùng ngày, cùng type: append section mới `## Thay đổi (lần N, HH:MM)`.
- Cùng ngày, khác type (light sáng + deep chiều): append deep sections vào file đã có, update frontmatter `type` thành union (vd `[tracking]` → `[tracking, review]`).
```

- [ ] **Step 4: Cập nhật bảng Hai tầng dữ liệu**

Tìm bảng `Hai tầng dữ liệu`. Xoá dòng:

```
| **Log review** | `.okr/log/reviews/YYYY-MM-DD.md` | Append-only |
```

Sửa dòng log thường:

```
| **Log** | `.okr/log/YYYY-MM-DD.md` | Append-only |
```

(giữ nguyên, chỉ xoá dòng Log review)

- [ ] **Step 5: Rewrite Log Reading Rules**

Xoá toàn bộ section `## Log Reading Rules` hiện tại (dòng 92-96). Thay bằng:

```markdown
## Log Reading Rules

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
```

- [ ] **Step 6: Xoá references `log/reviews/` trong Trace section**

Tìm dòng 151:
```
| Log | "xem log tuần trước" | Lọc `log/*.md` hoặc `log/reviews/*.md` theo filename (ngày) |
```

Sửa thành:
```
| Log | "xem log tuần trước" | Lọc `log/*.md` theo filename (ngày), filter theo `type` nếu user chỉ cần review |
```

- [ ] **Step 7: Commit**

```bash
git add skills/okr-track/references/data-format.md
git commit -m "refactor(okr-track): unify log schema, merge log/ and log/reviews/"
```

---

### Task 2: Cập nhật `okr-track/SKILL.md`

**Files:**
- Modify: `skills/okr-track/SKILL.md`

- [ ] **Step 1: Sửa Phase 1 log reading rules (dòng 33-35)**

Tìm:

```markdown
- `.okr/log/`: mode light → 1 file mới nhất. Mode deep → tối đa 3 files mới nhất. Mode closure → tất cả.
- `.okr/log/reviews/`: mode light → 1 file mới nhất. Mode deep → tối đa 3 files mới nhất. Mode closure → tất cả.
- Mode trace: đọc log/reviews theo yêu cầu user.
```

Thay bằng:

```markdown
- `.okr/log/`: mode light → KHÔNG đọc. Mode deep → adaptive rule (xem `references/data-format.md` Log Reading Rules). Mode closure → tất cả files.
- Mode trace: đọc `log/` theo yêu cầu user, filter `type` khi cần.
```

- [ ] **Step 2: Sửa Phase 4b deep Bước 7 (dòng 176)**

Tìm:

```markdown
7. Ghi log review vào `log/reviews/YYYY-MM-DD.md`
```

Thay bằng:

```markdown
7. Ghi log review vào `log/YYYY-MM-DD.md` (type: [review] hoặc union với type hiện có)
```

- [ ] **Step 3: Sửa Phase 4c closure (dòng 185)**

Tìm:

```markdown
Đọc mở rộng: `actions/archive/**` + tất cả `log/reviews/**` (tổng kết toàn period).
```

Thay bằng:

```markdown
Đọc mở rộng: `actions/archive/**` + tất cả `log/**` (tổng kết toàn period).
```

- [ ] **Step 4: Sửa Trace bảng (dòng 231)**

Tìm:

```
| Log | "xem log tuần trước" | `log/` hoặc `log/reviews/` theo ngày |
```

Thay bằng:

```
| Log | "xem log tuần trước" | `log/` theo ngày, filter `type` nếu cần |
```

- [ ] **Step 5: Sửa Schema section (dòng 242)**

Tìm:

```markdown
- `log/reviews/YYYY-MM-DD.md` (review entries)
```

Xoá dòng này. Dòng trên (`log/YYYY-MM-DD.md`) đã cover.

Sửa dòng trên nếu cần:

```markdown
- `log/YYYY-MM-DD.md` (unified: tracking, review, closure entries)
```

- [ ] **Step 6: Sửa Quy tắc section (dòng 257)**

Tìm:

```markdown
- Log review (deep) ghi vào `log/reviews/`. Log thường ghi vào `log/`.
```

Thay bằng:

```markdown
- Log ghi vào `log/YYYY-MM-DD.md`. Deep/closure append sections review, update frontmatter `type`.
```

- [ ] **Step 7: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "refactor(okr-track): update SKILL.md to unified log folder"
```

---

### Task 3: Cập nhật `flow-deep.md`

**Files:**
- Modify: `skills/okr-track/references/flow-deep.md`

- [ ] **Step 1: Sửa delegate payload `source_review` path (dòng 90)**

Tìm:

```yaml
  source_review: log/reviews/YYYY-MM-DD.md
```

Thay bằng:

```yaml
  source_review: log/YYYY-MM-DD.md
```

- [ ] **Step 2: Sửa bảng field meanings (dòng 101)**

Tìm:

```
| `context.source_review` | có | Path file review log để init/plan trace lại. Mặc định `log/reviews/<today>.md`. |
```

Thay bằng:

```
| `context.source_review` | có | Path file review log để init/plan trace lại. Mặc định `log/<today>.md`. |
```

- [ ] **Step 3: Sửa ví dụ payload (dòng 114)**

Tìm:

```yaml
  source_review: log/reviews/2026-12-01.md
```

Thay bằng:

```yaml
  source_review: log/2026-12-01.md
```

- [ ] **Step 4: Sửa Bước 7 ghi log review (dòng 126-133)**

Tìm:

```markdown
Sau khi tất cả delegate + inbox processing hoàn tất, append `.okr/log/reviews/YYYY-MM-DD.md`:

- Tổng kết KR/KI
- Phân tích root cause
- Đề xuất + cái nào đã apply (kèm skill nào áp dụng)
- Inbox items đã xử lý

Đồng thời append `log/YYYY-MM-DD.md` link sang file review.
```

Thay bằng:

```markdown
Sau khi tất cả delegate + inbox processing hoàn tất, ghi vào `.okr/log/YYYY-MM-DD.md`:

- Nếu file ngày đã có (từ Bước 1 update progress): append sections review, update frontmatter `type` thành union (vd `[tracking]` → `[tracking, review]`).
- Nếu file ngày chưa có: tạo mới với `type: [review]`.
- Nội dung review: Tổng kết KR/KI, Phân tích root cause, Đề xuất + cái nào đã apply, Inbox items đã xử lý.
```

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/references/flow-deep.md
git commit -m "refactor(okr-track): update flow-deep.md to unified log path"
```

---

### Task 4: Cập nhật `flow-closure.md`

**Files:**
- Modify: `skills/okr-track/references/flow-closure.md`

- [ ] **Step 1: Sửa dòng đọc mở rộng (dòng 3)**

Tìm:

```markdown
Đọc mở rộng: **đọc cả `actions/archive/**` (tổng kết toàn bộ actions đã hoàn thành) + **đọc tất cả `log/reviews/**` (tổng kết toàn period).
```

Thay bằng:

```markdown
Đọc mở rộng: **đọc cả `actions/archive/**` (tổng kết toàn bộ actions đã hoàn thành) + **đọc tất cả `log/**` (tổng kết toàn period).
```

- [ ] **Step 2: Sửa mục 5 ghi closure log (dòng 11)**

Tìm:

```markdown
5. Ghi log review closure (kèm section `## Lessons`).
```

Thay bằng:

```markdown
5. Ghi log vào `log/YYYY-MM-DD.md` với `type: [closure]` (hoặc union nếu file ngày đã có). Kèm section `## Lessons`.
```

- [ ] **Step 3: Commit**

```bash
git add skills/okr-track/references/flow-closure.md
git commit -m "refactor(okr-track): update flow-closure.md to unified log path"
```

---

### Task 5: Cập nhật `flow-trace.md`

**Files:**
- Modify: `skills/okr-track/references/flow-trace.md`

- [ ] **Step 1: Sửa Trace log section (dòng 48)**

Tìm:

```markdown
1. Lọc `log/*.md` hoặc `log/reviews/*.md` theo filename (chứa ngày)
```

Thay bằng:

```markdown
1. Lọc `log/*.md` theo filename (chứa ngày). Filter theo frontmatter `type` nếu user chỉ muốn xem review.
```

- [ ] **Step 2: Commit**

```bash
git add skills/okr-track/references/flow-trace.md
git commit -m "refactor(okr-track): update flow-trace.md to unified log path"
```

---

### Task 6: Cập nhật orchestrator `okr/SKILL.md`

**Files:**
- Modify: `skills/okr/SKILL.md`

- [ ] **Step 1: Sửa Bước 1 đọc trạng thái (dòng 46-47)**

Tìm:

```markdown
- `.okr/log/`: **KHÔNG đọc**. Log thô không cần cho state assessment.
- **Chỉ 1 file mới nhất** trong `.okr/log/reviews/` (sorted by filename desc, lấy đầu tiên). KHÔNG đọc review cũ hơn.
```

Thay bằng:

```markdown
- `.okr/log/`: **KHÔNG đọc**. Log không cần cho state assessment. Orchestrator không cần historical context.
```

- [ ] **Step 2: Commit**

```bash
git add skills/okr/SKILL.md
git commit -m "refactor(okr): remove log/reviews/ reading from orchestrator"
```

---

### Task 7: Cập nhật `shared-schemas.md`

**Files:**
- Modify: `skills/okr/references/shared-schemas.md`

- [ ] **Step 1: Sửa bảng Invisible by Default (dòng 137-139)**

Tìm:

```
| `log/` | **Không** | User trace theo ngày |
| `log/reviews/` latest | Có | Luôn đọc |
| `log/reviews/` cũ hơn | **Không** | User trace theo ngày |
```

Thay bằng:

```
| `log/` | **Không** | Deep adaptive rule, closure tất cả, trace lazy |
```

- [ ] **Step 2: Sửa bảng Quy tắc theo skill (dòng 145-149)**

Tìm:

```
| Orchestrator `/okr` | Không đọc | Không đọc log. Chỉ latest review |
| `okr-track` light | Không đọc | Chỉ latest log (để so trend) |
| `okr-track` deep | Không đọc | latest log + tối đa 3 reviews mới nhất |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả reviews** (tổng kết) |
| `okr-track` trace | **Đọc archive** (lazy) | **Đọc log cũ** (lazy) |
```

Thay bằng:

```
| Orchestrator `/okr` | Không đọc | Không đọc log |
| `okr-track` light | Không đọc | Không đọc log |
| `okr-track` deep | Không đọc | Adaptive rule (xem data-format.md) |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả log** (tổng kết) |
| `okr-track` trace | **Đọc archive** (lazy) | **Đọc log** (lazy, filter type) |
```

- [ ] **Step 3: Commit**

```bash
git add skills/okr/references/shared-schemas.md
git commit -m "refactor(okr): update shared-schemas.md log reading rules"
```

---

### Task 8: Cập nhật `CLAUDE.md` cây thư mục

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Xoá `reviews/` subfolder khỏi cây thư mục (dòng 62)**

Tìm:

```
└── log/               # Append-only                  (okr-track)
    └── reviews/       # Deep/closure reviews          (okr-track deep/closure)
```

Thay bằng:

```
└── log/               # Append-only, type: [tracking|review|closure]  (okr-track)
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: remove reviews/ subfolder from directory tree"
```

---

### Task 9: Verification

- [ ] **Step 1: Grep toàn bộ `skills/` tìm `log/reviews`**

```bash
grep -rn "log/reviews" skills/ --include="*.md"
```

Expected: 0 kết quả.

- [ ] **Step 2: Grep `CLAUDE.md` tìm `reviews/`**

```bash
grep -n "reviews/" CLAUDE.md
```

Expected: 0 kết quả.

- [ ] **Step 3: Verify `data-format.md` có unified schema**

```bash
grep -c "type: \[tracking\]" skills/okr-track/references/data-format.md
```

Expected: ≥1 (schema example).

```bash
grep -c "log/reviews" skills/okr-track/references/data-format.md
```

Expected: 0.

- [ ] **Step 4: Verify deep adaptive rule tồn tại**

```bash
grep -c "adaptive" skills/okr-track/references/data-format.md
```

Expected: ≥1.

- [ ] **Step 5: Verify orchestrator không đọc log**

```bash
grep -A2 "KHÔNG đọc" skills/okr/SKILL.md | head -5
```

Expected: thấy dòng `KHÔNG đọc` cho `log/` mà KHÔNG có reference tới `log/reviews/`.

- [ ] **Step 6: Final commit (nếu cần fixup)**

Nếu Step 1-5 phát hiện lỗi sót, sửa rồi commit:

```bash
git add -A
git commit -m "fix: address remaining log/reviews references"
```
