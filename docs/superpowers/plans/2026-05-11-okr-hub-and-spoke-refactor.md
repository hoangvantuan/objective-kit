# OKR Hub-and-Spoke Refactor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Chuyển kiến trúc cross-dependency sang hub-and-spoke: orchestrator `okr` là gateway bắt buộc, shared content tập trung, skill con chỉ chứa logic riêng.

**Architecture:** Tạo 2 reference files shared trong `okr/references/` (sot-ownership, shared-schemas). Tách 5 flow chi tiết từ `okr-track/SKILL.md` thành reference files riêng. Sửa tất cả cross-reference paths thành tên skill (vì model đã biết context từ orchestrator). Thêm preload SOT, suggested next action, keyword `add` cho orchestrator.

**Tech Stack:** Markdown files (skill definitions + references). Không có code logic, chỉ restructure nội dung.

---

### Task 1: Tạo `okr/references/sot-ownership.md`

**Files:**
- Create: `skills/okr/references/sot-ownership.md`

- [ ] **Step 1: Tạo file**

```markdown
# Phân vai SOT (Source of Truth)

Mỗi field SOT chỉ được sửa bởi đúng 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để apply.

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| KR.current, KI.current, action.status, plan counters              | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |

> Bảng này là bản runtime (load cùng skill okr). Bản developer đọc repo ở `CLAUDE.md` section "Phân vai SOT".
```

- [ ] **Step 2: Verify file tồn tại**

Run: `cat skills/okr/references/sot-ownership.md | head -5`
Expected: thấy heading `# Phân vai SOT`

- [ ] **Step 3: Commit**

```bash
git add skills/okr/references/sot-ownership.md
git commit -m "feat(okr): add sot-ownership.md reference file

Extract SOT ownership table from CLAUDE.md into runtime reference."
```

---

### Task 2: Tạo `okr/references/shared-schemas.md`

**Files:**
- Create: `skills/okr/references/shared-schemas.md`
- Source (đọc để extract): `skills/okr-plan/references/data-format.md:22-63`, `skills/okr-capture/references/data-format.md:62-70`, `skills/okr-track/references/data-format.md:84-164`

- [ ] **Step 1: Tạo file**

Gom 4 schema dùng chung từ 3 nguồn. Mỗi schema giữ nguyên nội dung gốc, chỉ tổ chức lại heading.

```markdown
# Shared Schemas

Schema dùng chung giữa nhiều skill. Load 1 lần qua orchestrator `okr`.

## Roadmap format

> Nguồn gốc: `okr-plan/references/data-format.md`. Bản tại đây là canonical.

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

Cột Assignee (ẩn khi solo): reserved for future. Khi Solo Profile có >1 người, thêm cột Assignee giữa Task và Deadline. Nguồn: `pic` frontmatter.

Quy tắc sinh bảng:

- **Ai render**: `okr-plan` (khi tạo/update action) và `okr-track` (khi update status/archive).
- **Sắp xếp**: Trong mỗi milestone, sắp theo Priority (critical > high > medium > low), rồi Deadline (sớm trước).
- **Lọc**: Chỉ hiển thị action chưa done. Action done đã archive, không xuất hiện trong bảng.
- **SOT**: Action files là SOT. Bảng là view được sinh lại mỗi lần update. Không sửa bảng trực tiếp.
- **Ongoing type**: Ongoing có thể có action không thuộc milestone nào. Nhóm dưới heading `### Chưa phân milestone`. Nếu Ongoing không có action nào, section `## Roadmap` chỉ chứa `## Practices` (không có bảng).

---

## Inbox Aging

> Nguồn gốc: `okr-capture/references/data-format.md`. Bản tại đây là canonical.

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

> Nguồn gốc: `okr-track/references/data-format.md`. Bản tại đây là canonical.

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

> Nguồn gốc: `okr-track/references/data-format.md`. Bản tại đây là canonical.

### Trigger

Khi `okr-track` (light hoặc deep) đánh `status: done` cho action, **trong cùng lần track, sau phase confirm**.

### Flow

1. Dời file `actions/AXXX-slug.md` → `actions/archive/AXXX-slug.md` (tạo thư mục `archive/` nếu chưa có).
2. Xóa dòng action đó khỏi `## Roadmap` body trong `plan.md`.
3. Nếu milestone trống (tất cả actions thuộc milestone đều done) → xóa heading milestone khỏi Roadmap body. Giữ milestone trong frontmatter `plan.md` (với `status: done`).
4. Cập nhật counters frontmatter `plan.md` (`completed` +N).

### Invisible by Default

| Nguồn dữ liệu | Mặc định đọc | Khi nào đọc thêm |
|----------------|-------------|-------------------|
| `actions/*.md` | Có (chỉ active) | Luôn đọc |
| `actions/archive/` | **Không** | User trace hoặc mode closure |
| `log/` | **Không** | User trace theo ngày |
| `log/reviews/` latest | Có | Luôn đọc |
| `log/reviews/` cũ hơn | **Không** | User trace theo ngày |

### Quy tắc theo skill

| Skill | Actions archive | Log cũ |
|-------|----------------|--------|
| Orchestrator `/okr` | Không đọc | Không đọc log. Chỉ latest review |
| `okr-track` light | Không đọc | Chỉ latest log (để so trend) |
| `okr-track` deep | Không đọc | latest log + tối đa 3 reviews mới nhất |
| `okr-track` closure | **Đọc archive** (tổng kết) | **Đọc tất cả reviews** (tổng kết) |
| `okr-track` trace | **Đọc archive** (lazy) | **Đọc log cũ** (lazy) |
| `okr-plan` update | Không đọc, không sửa | Không đọc |

Archive file schema giống hệt `actions/AXXX-slug.md`. Archive files là **read-only**.
```

- [ ] **Step 2: Verify file tồn tại + có đủ 4 sections**

Run: `grep "^## " skills/okr/references/shared-schemas.md`
Expected:
```
## Roadmap format
## Inbox Aging
## Inbox type → Delegate mapping
## Archive Rules
```

- [ ] **Step 3: Commit**

```bash
git add skills/okr/references/shared-schemas.md
git commit -m "feat(okr): add shared-schemas.md consolidating cross-skill schemas

Gom Roadmap format, Inbox Aging, Delegate mapping, Archive Rules
vào 1 file canonical duy nhất."
```

---

### Task 3: Tạo 5 flow reference files cho okr-track

**Files:**
- Create: `skills/okr-track/references/flow-light.md`
- Create: `skills/okr-track/references/flow-deep.md`
- Create: `skills/okr-track/references/flow-closure.md`
- Create: `skills/okr-track/references/flow-inbox.md`
- Create: `skills/okr-track/references/flow-trace.md`
- Source: `skills/okr-track/SKILL.md:140-525`

- [ ] **Step 1: Tạo flow-light.md**

Extract `skills/okr-track/SKILL.md` lines 140-196 (Phase 4a). Nội dung giữ nguyên, chỉ đổi heading level từ `###` thành `#` và sửa cross-references:
- Dòng 175: thay `skills/okr-plan/references/data-format.md` → `Xem Roadmap format tại skill okr/references/shared-schemas.md (đã load từ orchestrator).`

Nội dung file = toàn bộ Phase 4a từ SKILL.md hiện tại, với 2 thay đổi:
1. Heading `### Phase 4a:` → `# Mode LIGHT (cập nhật progress nhanh)`
2. Line 175 cross-ref fix (Roadmap format pointer)

- [ ] **Step 2: Tạo flow-deep.md**

Extract `skills/okr-track/SKILL.md` lines 199-336 (Phase 4b). Giữ nguyên nội dung, đổi heading + sửa cross-references:
- Dòng 317: thay `skills/okr-init/SKILL.md` và `skills/okr-plan/SKILL.md` bằng tên skill đơn giản (vì model đã biết context)

Nội dung file = toàn bộ Phase 4b, heading `### Phase 4b:` → `# Mode DEEP (review sâu + delegate)`.

- [ ] **Step 3: Tạo flow-closure.md**

Extract `skills/okr-track/SKILL.md` lines 338-349 (Phase 4c). Heading `### Phase 4c:` → `# Mode CLOSURE (mọi action done)`.

- [ ] **Step 4: Tạo flow-inbox.md**

Extract `skills/okr-track/SKILL.md` lines 352-471 (Phase 5). Sửa cross-references:
- Dòng 360: thay `skills/okr-capture/references/data-format.md` → `Xem Inbox Aging tại skill okr/references/shared-schemas.md (đã load từ orchestrator).`

Heading `### Phase 5:` → `# Inbox Processing Flow`.

- [ ] **Step 5: Tạo flow-trace.md**

Extract `skills/okr-track/SKILL.md` lines 475-525 (Trace Flow). Heading `## Trace Flow` → `# Trace Flow (mode trace)`.

- [ ] **Step 6: Verify 5 files tồn tại**

Run: `ls -la skills/okr-track/references/flow-*.md | wc -l`
Expected: `5`

Run: `wc -l skills/okr-track/references/flow-*.md`
Expected: flow-light ~57 dòng, flow-deep ~138 dòng, flow-closure ~12 dòng, flow-inbox ~120 dòng, flow-trace ~51 dòng.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-track/references/flow-*.md
git commit -m "feat(okr-track): extract 5 flow details into reference files

flow-light, flow-deep, flow-closure, flow-inbox, flow-trace.
SKILL.md will be slimmed in next commit."
```

---

### Task 4: Sửa `okr/SKILL.md` (orchestrator enhancements)

**Files:**
- Modify: `skills/okr/SKILL.md:27-29` (inline SOT)
- Modify: `skills/okr/SKILL.md:33-43` (preload SOT)
- Modify: `skills/okr/SKILL.md:57-58` (suggested next action)
- Modify: `skills/okr/SKILL.md:75` (add keyword)

4 thay đổi độc lập trong cùng file.

- [ ] **Step 1: Inline bảng SOT (thay link CLAUDE.md)**

Thay dòng 27-29:
```
## Phân vai SOT

Bảng canonical ở `CLAUDE.md` section "Phân vai SOT". Tóm tắt: mỗi field SOT chỉ được sửa bởi 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa, delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.
```

Thành:
```
## Phân vai SOT

Mỗi field SOT chỉ được sửa bởi đúng 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa, delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| KR.current, KI.current, action.status, plan counters              | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |

> Chi tiết: `references/sot-ownership.md` (load cùng skill này).
```

- [ ] **Step 2: Thêm preload SOT vào Bước 1**

Sau dòng 35 (trước danh sách đọc file), thêm block:

```
**Preload SOT data**: Orchestrator đọc sẵn frontmatter + body thiết yếu của 3 file SOT dưới đây. Context này available cho mọi skill con khi được kích hoạt, skill con KHÔNG cần đọc lại.
```

- [ ] **Step 3: Thêm keyword `add` vào Bước 3 routing**

Sửa dòng 75 từ:
```
| Keyword: "thêm nhanh / ghi lại / note / capture / nhớ cái này / inbox"             | `okr-capture` | n/a                |
```
Thành:
```
| Keyword: "thêm nhanh / ghi lại / note / capture / nhớ cái này / inbox / add"       | `okr-capture` | n/a                |
```

- [ ] **Step 4: Thêm suggested next action sau Bước 2 status**

Sau block status template (sau dòng 57 `Log gần   : YYYY-MM-DD ([X ngày trước])`), thêm:

```

**Gợi ý tiếp theo** (1 dòng, render sau block status):

| Tín hiệu (ưu tiên từ trên xuống) | Gợi ý |
|-----------------------------------|-------|
| Action quá hạn | "Có N action quá hạn. Update status hoặc dời deadline qua track/plan update." |
| KR at-risk | "KR<N> at-risk. Chạy deep review để phân tích." |
| Inbox ≥5 items | "Inbox có N items. Xử lý qua track hoặc inbox-only." |
| Mọi action done | "Tất cả action done. Chạy closure để chốt." |
| Default | "Chạy track light để cập nhật tiến độ." |
```

- [ ] **Step 5: Verify 4 thay đổi**

Run: `grep -n "Phân vai SOT" skills/okr/SKILL.md` → phải thấy bảng inline
Run: `grep -n "Preload SOT" skills/okr/SKILL.md` → phải thấy dòng preload
Run: `grep -n "add" skills/okr/SKILL.md | grep -i keyword` → phải thấy "add" trong keyword row
Run: `grep -n "Gợi ý tiếp theo" skills/okr/SKILL.md` → phải thấy section mới

- [ ] **Step 6: Commit**

```bash
git add skills/okr/SKILL.md
git commit -m "feat(okr): inline SOT table, preload, add keyword, suggested action

4 enhancements cho orchestrator:
- Inline bảng SOT thay vì link CLAUDE.md
- Ghi rõ preload SOT data cho skill con
- Thêm keyword 'add' route sang capture
- Thêm suggested next action sau status block"
```

---

### Task 5: Slim `okr-track/SKILL.md` + fixes

**Files:**
- Modify: `skills/okr-track/SKILL.md` (thay thế Phase 4a/4b/4c/Phase 5/Trace Flow bằng tóm tắt + pointer)

Đây là task lớn nhất. Thay thế ~378 dòng chi tiết bằng ~75 dòng tóm tắt. Đồng thời sửa 3 vấn đề: deep mode đọc 3 reviews, inbox-only clarify, cross-refs.

- [ ] **Step 1: Sửa Phase 1 (deep mode đọc 3 reviews)**

Thay dòng 33:
```
- **Chỉ 1 file mới nhất** trong `.okr/log/reviews/` (nếu có, để so trend)
```
Thành:
```
- `.okr/log/reviews/`: mode light → chỉ 1 file mới nhất (sorted desc). Mode deep → tối đa 3 files mới nhất. Mode closure → tất cả. Trừ mode trace: đọc raw log khi user yêu cầu cụ thể.
- `.okr/log/`: KHÔNG đọc (reviews đã tổng hợp nội dung log).
```

- [ ] **Step 2: Thay Phase 4a chi tiết bằng tóm tắt**

Thay toàn bộ Phase 4a (dòng 140-196) bằng:

```markdown
### Phase 4a: Mode LIGHT (cập nhật progress nhanh)

Phạm vi: CHỈ progress fields. Không sửa cấu trúc.

**Project type:**
1. Sync pull external (nếu action có `external_ids`)
2. Hỏi user update: KR current, action status, blocker mới
3. CONFIRM: ≤2 field → 1 dòng compact, ≥3 field → bảng đầy đủ
4. Ghi SOT (objective, plan counters, action status) + append log
5. Sync push + archive actions done + re-render Roadmap
6. Xử lý inbox (Phase 5) nếu có items pending
7. Đề xuất next action (1-7 ngày tới)

**Ongoing type:**
1. Hỏi update KI current + practice streak (plan.md `## Practices`)
2. Hỏi update action status (nếu có action files)
3. CONFIRM + ghi SOT + log (kèm streak thay đổi)
4. Xử lý inbox + đề xuất cải thiện KI nếu warning/critical

Chi tiết đầy đủ: `references/flow-light.md`
```

- [ ] **Step 3: Thay Phase 4b chi tiết bằng tóm tắt**

Thay toàn bộ Phase 4b (dòng 199-336) bằng:

```markdown
### Phase 4b: Mode DEEP (review sâu + delegate)

Phạm vi: phân tích root cause + ĐỀ XUẤT điều chỉnh. KHÔNG tự sửa cấu trúc.

1. Update progress nếu cần (giống light, kèm sync + archive)
2. Phân tích root cause (hỏi "tại sao?" ≥3 lần mỗi vấn đề)
3. Đề xuất điều chỉnh cấu trúc (bảng kèm skill đích: okr-init/okr-plan)
4. All-changes confirm: gom theo skill đích, user chọn áp dụng
5. Delegate sang skill phù hợp (kèm payload: changes, reason, pre_confirmed)
6. Xử lý inbox (Phase 5)
7. Ghi log review vào `log/reviews/YYYY-MM-DD.md`
8. Đề xuất next action

Chi tiết đầy đủ: `references/flow-deep.md`
```

- [ ] **Step 4: Thay Phase 4c chi tiết bằng tóm tắt**

Thay toàn bộ Phase 4c (dòng 338-349) bằng:

```markdown
### Phase 4c: Mode CLOSURE (mọi action done)

Đọc mở rộng: `actions/archive/**` + tất cả `log/reviews/**` (tổng kết toàn period).

Như deep + thêm:
1. Tổng kết toàn period (KR đạt vs target, % thời gian, lessons learned)
2. Hỏi user: completed hay tạo follow-up project?
3. Delegate `okr-init update-objective` đổi status nếu user đồng ý
4. Xử lý inbox còn lại
5. Ghi log review closure (kèm section `## Lessons`)

Chi tiết đầy đủ: `references/flow-closure.md`
```

- [ ] **Step 5: Thay Phase 5 chi tiết bằng tóm tắt + inbox-only clarify**

Thay toàn bộ Phase 5 (dòng 352-471) bằng:

```markdown
### Phase 5: Inbox Processing Flow

Chạy sau update progress (light) hoặc sau delegate (deep).

**Mode inbox-only**: SOT đã có context từ orchestrator okr (preload Phase 1). Skip Phase 2 dashboard, Phase 3 detect, Phase 4 progress. Đi thẳng Phase 5.

Flow:
1. Đọc inbox pending + compute staleness (on-the-fly, xem Inbox Aging tại skill okr/references/shared-schemas.md)
2. Cảnh báo stale items (>30 ngày), hỏi user giữ/bỏ
3. Hiển thị bảng gợi ý xử lý (map vào KR/milestone/action)
4. Validate related IDs (KR, action, tool) trước khi xử lý
5. Xử lý từng item theo type: delegate hoặc tự apply
6. Gom delegate cùng skill → 1 lần gọi
7. Báo cáo: items đã xử lý + inbox còn lại

Inbox type → Delegate mapping: xem skill okr/references/shared-schemas.md (đã load từ orchestrator).

Chi tiết đầy đủ: `references/flow-inbox.md`
```

- [ ] **Step 6: Thay Trace Flow chi tiết bằng tóm tắt**

Thay toàn bộ Trace Flow (dòng 475-525) bằng:

```markdown
## Trace Flow (mode `trace`)

Lazy loading: frontmatter trước, body khi user drill-down. Không update progress.

4 kiểu trace:

| Kiểu | Trigger | Đọc gì |
|------|---------|--------|
| Action cụ thể | "trace A003" | `actions/archive/A003-*.md` |
| Milestone | "trace M1" | `plan.md` + `actions/archive/` theo milestone |
| Theo thời gian | "actions done tháng 4" | `actions/archive/` theo `due_date` |
| Log | "xem log tuần trước" | `log/` hoặc `log/reviews/` theo ngày |

Chi tiết đầy đủ: `references/flow-trace.md`
```

- [ ] **Step 7: Verify SKILL.md đã slim**

Run: `wc -l skills/okr-track/SKILL.md`
Expected: ~200-250 dòng (target spec ~180, thực tế hơn vì giữ dashboard templates)

Run: `grep -c "references/flow-" skills/okr-track/SKILL.md`
Expected: `5` (5 pointer tới flow files)

- [ ] **Step 8: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "refactor(okr-track): slim SKILL.md, extract flows to references

5 flow details -> reference files. SKILL.md giữ Phase 1-3 + summaries.
Fixes: deep reads 3 reviews, inbox-only clarify, cross-ref cleanup."
```

---

### Task 6: Giảm trùng lặp `okr-track/references/data-format.md`

**Files:**
- Modify: `skills/okr-track/references/data-format.md:84-164` (thay bằng pointers)

- [ ] **Step 1: Thay section "Inbox Processing" + "Archive Rules" bằng pointers**

Thay dòng 84-164 (từ `## Inbox Processing` đến hết `## Archive Rules` bao gồm cả archive file schema) bằng:

```markdown
## Inbox Processing

> Đã chuyển sang `skills/okr/references/shared-schemas.md` section "Inbox type → Delegate mapping". Đọc tại đó (load từ orchestrator okr).

## Archive Rules

> Đã chuyển sang `skills/okr/references/shared-schemas.md` section "Archive Rules". Đọc tại đó (load từ orchestrator okr).
```

- [ ] **Step 2: Sửa bảng "Quy tắc theo skill" trong Log Reading Rules (deep 3 reviews)**

Tìm dòng trong section "Log Reading Rules" (dòng ~169):
```
- `okr-track` Phase 1: chỉ đọc **1 file mới nhất** trong `log/` và **1 file mới nhất** trong `log/reviews/` (để so trend).
```

Thay thành:
```
- `okr-track` Phase 1: chỉ đọc **1 file mới nhất** trong `log/`. Mode light: **1 file mới nhất** trong `log/reviews/`. Mode deep: **tối đa 3 files mới nhất** trong `log/reviews/`.
```

- [ ] **Step 3: Verify sections đã thay**

Run: `grep -n "Đã chuyển sang" skills/okr-track/references/data-format.md`
Expected: 2 dòng (Inbox Processing + Archive Rules pointers)

Run: `wc -l skills/okr-track/references/data-format.md`
Expected: giảm ~80 dòng so với trước (233 → ~155)

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/references/data-format.md
git commit -m "refactor(okr-track): deduplicate data-format.md

Move Inbox Processing + Archive Rules to shared-schemas.md.
Update Log Reading Rules: deep reads 3 reviews."
```

---

### Task 7: Fix cross-refs trong okr-init, okr-plan

**Files:**
- Modify: `skills/okr-init/SKILL.md:19`
- Modify: `skills/okr-plan/SKILL.md:17`
- Modify: `skills/okr-plan/references/data-format.md:55`

- [ ] **Step 1: Sửa okr-init/SKILL.md dòng 19**

Thay:
```
Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.
```
Thành:
```
Áp dụng Quality Gate (đã load từ skill okr, file references/quality-gate.md). Đọc trước khi tiến hành các phase hỏi user.
```

- [ ] **Step 2: Sửa okr-plan/SKILL.md dòng 17**

Thay:
```
Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.
```
Thành:
```
Áp dụng Quality Gate (đã load từ skill okr, file references/quality-gate.md). Đọc trước khi tiến hành các phase hỏi user.
```

- [ ] **Step 3: Sửa okr-plan/references/data-format.md dòng 55**

Thay:
```
Cột Assignee (ẩn khi solo): Khi Solo Profile có >1 người (mở rộng tương lai), thêm cột Assignee giữa Task và Deadline. Nguồn: `pic` frontmatter.
```
Thành:
```
Cột Assignee (ẩn khi solo): reserved for future. Khi Solo Profile có >1 người (mở rộng tương lai), thêm cột Assignee giữa Task và Deadline. Nguồn: `pic` frontmatter. Hiện tại luôn ẩn (solo mode).
```

Thêm note cuối section Roadmap format (sau dòng 63):
```

> Schema Roadmap cũng có bản tổng hợp tại skill okr/references/shared-schemas.md.
```

- [ ] **Step 4: Verify cross-refs đã sửa**

Run: `grep -rn "skills/okr/references/quality-gate.md" skills/okr-init/ skills/okr-plan/`
Expected: 0 kết quả (đã thay hết path cũ)

Run: `grep -n "reserved for future" skills/okr-plan/references/data-format.md`
Expected: 1 kết quả

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr-plan/references/data-format.md
git commit -m "fix(okr-init, okr-plan): replace cross-ref paths with skill names

Quality Gate ref: path -> 'đã load từ skill okr'.
Assignee col: clarify 'reserved for future, hiện tại luôn ẩn'.
Add shared-schemas note to Roadmap format."
```

---

### Task 8: Sửa `okr-capture/SKILL.md` (single item skip confirm)

**Files:**
- Modify: `skills/okr-capture/SKILL.md:53-65`

- [ ] **Step 1: Thêm logic single item skip confirm**

Tìm Phase 3 (dòng 53 `### Phase 3: Confirm (nhanh)`). Thêm block mới **trước** phần "**Single item:**":

```markdown
### Phase 3: Confirm (nhanh)

**Single item skip confirm**: Khi input chỉ 1 item đơn giản + type rõ ràng (action hoặc blocker), skip Phase 3 confirm. Ghi luôn + thông báo:

```
✓ Đã lưu "Viết unit test API auth" vào inbox (type: action, gợi ý KR2).
```

Giữ confirm cho:
- Batch (≥2 items)
- Type mơ hồ (không chắc action hay thought)
- User muốn review (nói "xem lại trước khi lưu")

**Single item (cần confirm):**
```

Ngoài ra, đổi heading "**Single item:**" (dòng 55) thành "**Single item (cần confirm):**" để phân biệt với flow skip.

- [ ] **Step 2: Verify thay đổi**

Run: `grep -n "skip confirm" skills/okr-capture/SKILL.md`
Expected: thấy dòng "Single item skip confirm"

- [ ] **Step 3: Commit**

```bash
git add skills/okr-capture/SKILL.md
git commit -m "feat(okr-capture): single item skip confirm for simple inputs

1 item + type rõ -> ghi luôn, không hỏi confirm.
Batch hoặc type mơ hồ -> vẫn confirm."
```

---

### Task 9: Update `CLAUDE.md` (thêm note)

**Files:**
- Modify: `CLAUDE.md` (section "Phân vai SOT")

- [ ] **Step 1: Thêm note sau bảng SOT**

Tìm dòng cuối của bảng SOT trong CLAUDE.md (dòng sau `| External sync (pull/push status) |`). Thêm ngay sau:

```

> Bảng này cũng có bản runtime tại `skills/okr/references/sot-ownership.md` (load khi skill okr chạy). CLAUDE.md là bản cho developer đọc repo.
```

- [ ] **Step 2: Verify**

Run: `grep -n "sot-ownership" CLAUDE.md`
Expected: 1 kết quả

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add note linking SOT table to runtime reference"
```

---

### Task 10: Verification tổng thể

**Files:**
- Đọc: tất cả file đã sửa

- [ ] **Step 1: Grep cross-reference paths cũ**

Run:
```bash
grep -rn "skills/okr/references/quality-gate.md" skills/okr-init/ skills/okr-plan/
grep -rn "skills/okr-plan/references/data-format.md" skills/okr-track/
grep -rn "skills/okr-capture/references/data-format.md" skills/okr-track/
grep -rn "skills/okr-init/SKILL.md" skills/okr-track/
grep -rn "skills/okr-plan/SKILL.md" skills/okr-track/
```

Expected: tất cả trả về 0 kết quả. Nếu có match, sửa lại thành tên skill.

- [ ] **Step 2: Đếm dòng SKILL.md files**

Run:
```bash
wc -l skills/okr/SKILL.md skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr-track/SKILL.md skills/okr-capture/SKILL.md
```

Expected: `okr-track/SKILL.md` giảm đáng kể (từ 553 → ~200-250). Các file khác tăng nhẹ.

Target: mỗi SKILL.md ≤400 dòng.

- [ ] **Step 3: Kiểm tra references đủ**

Run:
```bash
ls skills/okr/references/
ls skills/okr-track/references/
```

Expected:
- `okr/references/`: quality-gate.md, sot-ownership.md, shared-schemas.md (3 files)
- `okr-track/references/`: data-format.md, metrics.md, flow-light.md, flow-deep.md, flow-closure.md, flow-inbox.md, flow-trace.md (7 files)

- [ ] **Step 4: Verify shared-schemas canonical sections tồn tại**

Run: `grep "^## " skills/okr/references/shared-schemas.md`
Expected: Roadmap format, Inbox Aging, Inbox type → Delegate mapping, Archive Rules

- [ ] **Step 5: Kiểm tra không còn cross-dependency path dạng `skills/<tên-skill>/`**

Run:
```bash
grep -rn "skills/okr-" skills/ --include="*.md" | grep -v "shared-schemas\|sot-ownership\|SKILL.md:1\|SKILL.md:2\|SKILL.md:3"
```

Expected: chỉ còn references hợp lệ (skill con trỏ về okr/references/), không còn skill con trỏ chéo sang skill con khác.

- [ ] **Step 6: Commit final (nếu có fix)**

Nếu Step 1-5 phát hiện lỗi, sửa rồi commit:
```bash
git add -A
git commit -m "fix: resolve remaining cross-references after hub-and-spoke refactor"
```
