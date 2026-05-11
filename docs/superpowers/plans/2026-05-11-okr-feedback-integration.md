# OKR Feedback Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Tích hợp 4 feedback cải thiện hệ thống OKR: bảng Roadmap per-milestone, external IDs sync 2 chiều, resources.md 6 cột thống nhất, và hướng dẫn sử dụng resource đầy đủ.

**Architecture:** Mở rộng tối thiểu 3 schema files (data-format.md), 3 SKILL.md, và CLAUDE.md. Không breaking change. Action frontmatter thêm 2 field optional (`notes`, `external_ids`). Resources.md chuyển từ 4 cột sang 6 cột thống nhất cho cả Công cụ và Tài liệu/KB. Plan.md Roadmap chuyển từ bullet list sang bảng per-milestone.

**Tech Stack:** Markdown schema files, YAML frontmatter, Skill prompt engineering

---

## File Structure

### Files sẽ sửa (modify)

| File | Thay đổi |
|------|---------|
| `skills/okr-plan/references/data-format.md` | Thêm field `notes`, `external_ids` vào action schema. Thêm Roadmap bảng per-milestone format. |
| `skills/okr-init/references/data-format.md` | Cập nhật resources.md schema: 6 cột cho Công cụ và KB. Thêm migration rules. |
| `skills/okr-track/references/data-format.md` | Thêm sync flow, reference external_ids trong archive rules. |
| `skills/okr-plan/SKILL.md` | Logic render bảng Roadmap khi tạo/update action. |
| `skills/okr-track/SKILL.md` | Thêm sync phase (pull/push). Logic re-render bảng Roadmap khi archive. |
| `skills/okr-init/SKILL.md` | Migration logic resources.md schema cũ → 6 cột. Cập nhật Phase 5 thu thập resource. |
| `CLAUDE.md` | Cập nhật bảng SOT: thêm `notes`, `external_ids`. Ghi chú sync behavior. |

### Không thay đổi

- `skills/okr-capture/` (không ảnh hưởng)
- `skills/okr/SKILL.md` (orchestrator routing không đổi)
- Cấu trúc thư mục `.okr/` (không thêm/xóa file)

---

## Task 1: Thêm field `notes` và `external_ids` vào action schema

**Files:**
- Modify: `skills/okr-plan/references/data-format.md:24-39` (action frontmatter schema)

- [ ] **Step 1: Đọc file hiện tại để xác nhận vị trí chính xác**

```bash
cat -n skills/okr-plan/references/data-format.md | head -50
```

Xác nhận action frontmatter block bắt đầu dòng 24, kết thúc dòng 39.

- [ ] **Step 2: Thêm 2 field mới vào action frontmatter schema**

Trong file `skills/okr-plan/references/data-format.md`, tìm block action frontmatter (```yaml ... ```). Thêm `notes` và `external_ids` SAU `depends_on`:

Thay thế block frontmatter cũ:

```yaml
---
id: AXXX
title: "string"
description: "string"
key_result: KR1
milestone: "string"
status: pending | doing | done | blocked
priority: critical | high | medium | low
effort: xs | s | m | l | xl
pic: "string"
due_date: YYYY-MM-DD
depends_on: [A001, A002]
---
```

Bằng:

```yaml
---
id: AXXX
title: "string"
description: "string"
key_result: KR1
milestone: "string"
status: pending | doing | done | blocked
priority: critical | high | medium | low
effort: xs | s | m | l | xl
pic: "string"
due_date: YYYY-MM-DD
depends_on: [A001, A002]
notes: "string (optional, ≤50 ký tự)"
external_ids:              # optional
  tool_key: "external_id_string"
---
```

- [ ] **Step 3: Thêm section giải thích 2 field mới**

Ngay sau block frontmatter (sau dòng `---`), trước dòng `Body:`, thêm:

```markdown
Field mới (optional):

| Field | Type | Mô tả |
|-------|------|-------|
| `notes` | string, optional | Ghi chú ngắn hiển thị trong bảng Roadmap. Khuyến nghị ≤50 ký tự. Vd: "Cần xong trước A003", "Blocked by API access". Khác với `## Ghi chú` body (chi tiết dài). Không có thì cell trống. |
| `external_ids` | YAML map, optional | Map tool_key → external ID string. Key lowercase, match tên tool trong resources.md (vd: `things3`, `notion`). Nhiều tool được. Dùng cho sync 2 chiều với tool bên thứ 3. |

Ví dụ `external_ids`:

```yaml
external_ids:
  things3: "ABC-123"
  notion: "page-xyz-456"
```
```

- [ ] **Step 4: Verify file sau chỉnh sửa**

```bash
cat -n skills/okr-plan/references/data-format.md | head -70
```

Confirm: frontmatter có `notes` + `external_ids`, section giải thích ngay sau.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-plan/references/data-format.md
git commit -m "feat(okr-plan): add notes and external_ids fields to action schema"
```

---

## Task 2: Thêm format bảng Roadmap per-milestone vào plan.md schema

**Files:**
- Modify: `skills/okr-plan/references/data-format.md:19-21` (Roadmap body description)

- [ ] **Step 1: Đọc phần Roadmap hiện tại**

```bash
grep -n "Roadmap\|Body:" skills/okr-plan/references/data-format.md
```

Xác nhận dòng 21: `Body: \`## Roadmap\` với các heading milestone, mỗi milestone liệt kê actions.`

- [ ] **Step 2: Thay thế mô tả Roadmap ngắn bằng format bảng chi tiết**

Thay dòng 21:

```
Body: `## Roadmap` với các heading milestone, mỗi milestone liệt kê actions.
```

Bằng:

```markdown
Body: `## Roadmap` với bảng action per-milestone. Bảng auto-generated từ action files, là view (không sửa trực tiếp). SOT là action files.

### Roadmap format

Mỗi milestone heading chứa bảng action bên dưới:

```markdown
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
```

Cột bảng (5 cột, solo mode):

| Cột | Nguồn dữ liệu | Ghi chú |
|-----|---------------|---------|
| ID | `id` frontmatter, link tới file action | Relative link: `actions/AXXX-slug.md` |
| Task | `title` frontmatter | Hiển thị nguyên văn |
| Deadline | `due_date` frontmatter | Format YYYY-MM-DD |
| Priority | `priority` frontmatter | critical, high, medium, low |
| Notes | `notes` frontmatter (optional) | Không có thì cell trống |

Cột Assignee (ẩn khi solo): Khi Solo Profile có >1 người (mở rộng tương lai), thêm cột Assignee giữa Task và Deadline. Nguồn: `pic` frontmatter.

Quy tắc sinh bảng:

- **Ai render**: `okr-plan` (khi tạo/update action) và `okr-track` (khi update status/archive).
- **Sắp xếp**: Trong mỗi milestone, sắp theo Priority (critical > high > medium > low), rồi Deadline (sớm trước).
- **Lọc**: Chỉ hiển thị action chưa done. Action done đã archive, không xuất hiện trong bảng.
- **SOT**: Action files là SOT. Bảng là view được sinh lại mỗi lần update. Không sửa bảng trực tiếp.
- **Ongoing type**: Ongoing có thể có action không thuộc milestone nào. Nhóm dưới heading `### Chưa phân milestone`. Nếu Ongoing không có action nào, section `## Roadmap` chỉ chứa `## Practices` (không có bảng).
```

- [ ] **Step 3: Verify format**

```bash
grep -A 5 "Roadmap format" skills/okr-plan/references/data-format.md
```

Confirm: section Roadmap format có bảng mẫu + 5 cột + quy tắc sinh bảng.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/references/data-format.md
git commit -m "feat(okr-plan): add per-milestone table format for Roadmap in plan.md"
```

---

## Task 3: Cập nhật resources.md schema 6 cột thống nhất

**Files:**
- Modify: `skills/okr-init/references/data-format.md:62-68` (resources.md body sections)

- [ ] **Step 1: Đọc schema resources hiện tại**

```bash
grep -n "Công cụ\|Tài liệu\|Knowledge Base" skills/okr-init/references/data-format.md
```

Xác nhận 2 dòng mô tả bảng hiện tại:
- `## Công cụ` (bảng: Tên công cụ, Khi nào dùng, Mục đích, Resource (URL/Account))
- `## Tài liệu & Knowledge Base` (bảng: Tên/Loại, Vị trí (Link/Folder/File), Mục đích, Status)

- [ ] **Step 2: Thay thế 2 dòng schema cũ bằng 6 cột thống nhất**

Tìm block body sections (khoảng dòng 62-68). Thay thế:

```
- `## Công cụ` (bảng: Tên công cụ, Khi nào dùng, Mục đích, Resource (URL/Account))
- `## Tài liệu & Knowledge Base` (bảng: Tên/Loại, Vị trí (Link/Folder/File), Mục đích, Status)
```

Bằng:

```
- `## Công cụ` (bảng 6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú)
- `## Tài liệu & Knowledge Base` (bảng 6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú)
```

- [ ] **Step 3: Thêm section chi tiết schema 6 cột + migration rules**

Sau block body sections, trước `Skill OKR phục vụ persona **solo only**`, thêm section mới:

```markdown
### Schema 6 cột thống nhất (Công cụ + Tài liệu/KB)

Cả 2 section dùng cùng 6 cột:

```markdown
| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
```

Ví dụ `## Công cụ`:

```markdown
| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| Things 3 | Quản lý task hàng ngày | Daily planning, tracking | skill: things-mac | macOS app | Sync 2 chiều với OKR |
| Cursor | Code editor AI | Khi code, debug | Mở project → chat | cursor.sh | License Pro |
```

Ví dụ `## Tài liệu & Knowledge Base`:

```markdown
| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| API Docs v2 | Reference backend endpoints | Khi tích hợp API | Mở link → search endpoint | https://docs.example.com | Cần VPN |
| Design System | UI components reference | Khi build frontend | Figma → Library → Components | figma.com/file/xxx | v2.1 |
```

Quy tắc cột "Cách dùng":

| Loại resource | Nội dung cột "Cách dùng" |
|--------------|-------------------------|
| Tool có tích hợp OKR | `skill: <tên-skill>` hoặc `mcp: <tên-mcp-server>` |
| Tool không tích hợp | Quy trình ngắn 1 dòng (vd: "Mở project → chat") |
| Tài liệu online | Cách truy cập (vd: "Mở link → search endpoint") |
| Tài liệu local | Path hoặc cách mở (vd: "GDrive → Folder meetings") |
| Knowledge base | Cách query (vd: "skill: deepwiki → search topic") |

### Migration schema cũ → 6 cột

`okr-init` mode `update-resource` lần đầu gặp schema cũ:

1. Detect bảng Công cụ có 4 cột (Tên công cụ, Khi nào dùng, Mục đích, Resource) → schema cũ.
2. Auto-migrate: map cột cũ → cột mới, thêm 2 cột trống (Cách dùng, Ghi chú). Báo user điền sau.
3. Detect bảng KB có 4 cột (Tên/Loại, Vị trí, Mục đích, Status) → map: Tên/Loại → Tên, Vị trí → Resource, Mục đích giữ, Status → gộp vào Ghi chú. Thêm Khi nào dùng + Cách dùng trống.
4. Không mất dữ liệu, chỉ restructure cột.
```

- [ ] **Step 4: Verify**

```bash
grep -A 3 "Schema 6 cột" skills/okr-init/references/data-format.md
```

Confirm: section mới có bảng mẫu + migration rules.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/references/data-format.md
git commit -m "feat(okr-init): update resources.md schema to unified 6-column format"
```

---

## Task 4: Thêm sync flow + external_ids reference vào okr-track data-format

**Files:**
- Modify: `skills/okr-track/references/data-format.md` (cuối file, trước hoặc sau Archive Rules)

- [ ] **Step 1: Đọc cuối file để tìm vị trí chèn**

```bash
wc -l skills/okr-track/references/data-format.md
tail -20 skills/okr-track/references/data-format.md
```

- [ ] **Step 2: Thêm section External Sync trước section "## Trace Flow"**

Tìm dòng `## Trace Flow` trong file. Chèn TRƯỚC nó:

```markdown
## External Sync (optional)

Sync 2 chiều giữa OKR action files và tool bên thứ 3 (Things 3, Notion, Jira...). Chỉ chạy nếu action có field `external_ids` (xem `skills/okr-plan/references/data-format.md`).

### Pull (trước tracking, Phase 4a/4b)

1. Đọc tất cả active action files có `external_ids` trong frontmatter.
2. Với mỗi tool trong `external_ids`:
   a. Tra `resources.md` `## Công cụ` → tìm cột "Cách dùng" để biết integration method (`skill: <name>`, `mcp: <name>`).
   b. Gọi skill/MCP tương ứng → lấy status task bên ngoài.
3. So sánh status:
   - Khác biệt → hiển thị: "Tool X: A001 = completed, OKR: A001 = doing. Đồng bộ? (y/n)"
   - User confirm → update action status trong OKR (progress field, track tự xử lý).
4. Tool không có skill/MCP hoặc cột "Cách dùng" rỗng → skip, log cảnh báo.

### Push (sau tracking, Phase 4a/4b)

1. Với mỗi action vừa thay đổi status + có `external_ids`:
   a. Tra integration method từ `resources.md`.
   b. Gọi skill/MCP → push status mới lên tool ngoài.
2. Báo kết quả: "Đã sync A001 → Things 3 (completed), A002 → Notion (doing)"
3. Push thất bại → log lỗi, không retry tự động.

### Mapping status

| OKR Status | Hướng map chung |
|------------|----------------|
| pending | Tool-specific "todo" / "not started" |
| doing | Tool-specific "in progress" |
| done | Tool-specific "completed" / "done" |
| blocked | Tool-specific "on hold" / "blocked" (nếu tool hỗ trợ) |

Mapping cụ thể theo từng tool nằm trong config của skill/MCP tương ứng. OKR system không hardcode.

### Nguyên tắc sync

- Pull: mọi thay đổi từ tool ngoài cần user confirm trước khi ghi vào OKR.
- Push: tự động sau confirm tracking (không hỏi thêm).
- Tool không có integration → skip, không lỗi.
- Sync là optional step, không block tracking flow nếu thất bại.
- Archive: action đã archive không sync nữa (read-only).
```

- [ ] **Step 3: Verify vị trí chèn**

```bash
grep -n "External Sync\|Trace Flow" skills/okr-track/references/data-format.md
```

Confirm: "External Sync" xuất hiện trước "Trace Flow".

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/references/data-format.md
git commit -m "feat(okr-track): add external sync flow to data-format reference"
```

---

## Task 5: Cập nhật okr-plan SKILL.md, thêm logic render bảng Roadmap

**Files:**
- Modify: `skills/okr-plan/SKILL.md:100-109` (Phase 4 + Phase 5 mode NEW)
- Modify: `skills/okr-plan/SKILL.md:193-198` (Phase 5 mode UPDATE)

- [ ] **Step 1: Đọc Phase 4 mode NEW hiện tại**

```bash
sed -n '100,110p' skills/okr-plan/SKILL.md
```

Xác nhận Phase 4 ghi file: tạo plan.md, tạo actions/, ghi action files, cập nhật counters.

- [ ] **Step 2: Thêm step render bảng Roadmap vào Phase 4 mode NEW**

Tìm block Phase 4 (khoảng dòng 100-109). Thay:

```markdown
### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`. Mọi action `pic: self`.
4. Cập nhật counters trong `plan.md`.
```

Bằng:

```markdown
### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`. Mọi action `pic: self`.
4. Cập nhật counters trong `plan.md`.
5. **Render bảng Roadmap** trong `plan.md` body `## Roadmap`: sinh bảng per-milestone từ action files vừa tạo. Format xem `references/data-format.md` section "Roadmap format". Sắp xếp: Priority (critical > high > medium > low), rồi Deadline (sớm trước). Ongoing không có milestone → heading `### Chưa phân milestone`.
```

- [ ] **Step 3: Thêm render bảng vào Phase 5 mode UPDATE**

Tìm block Phase 5 mode UPDATE (khoảng dòng 193-198). Thay:

```markdown
### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`. Mọi action mới `pic: self`.
3. Đề xuất chạy `okr-track` mode `light` để confirm state mới.
```

Bằng:

```markdown
### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`. Mọi action mới `pic: self`.
3. **Re-render bảng Roadmap** trong `plan.md` body `## Roadmap`: đọc lại tất cả `actions/*.md` (không archive), sinh bảng per-milestone. Format xem `references/data-format.md` section "Roadmap format".
4. Đề xuất chạy `okr-track` mode `light` để confirm state mới.
```

- [ ] **Step 4: Thêm field `notes` vào Phase 2 đề xuất action (mode NEW)**

Tìm Phase 2 mode NEW (khoảng dòng 42-60), phần "Mỗi Initiative → 2-5 **Actions** cụ thể". Sau dòng 4 (dependencies), thêm:

```
5. Nếu action có ghi chú ngắn (dependency, blocker, context quan trọng), ghi vào field `notes` (≤50 ký tự). Không có thì để trống.
```

- [ ] **Step 5: Thêm `external_ids` vào Phase 2 hỏi user (mode NEW)**

Sau step 5 vừa thêm, thêm:

```
6. Nếu action liên kết tool trong resources.md có `Cách dùng` là `skill:` hoặc `mcp:`, hỏi user có external ID để sync không. Nếu có → ghi `external_ids` frontmatter. Nếu không → bỏ qua.
```

- [ ] **Step 6: Thêm `notes` và `external_ids` vào Phase 3 CONFIRM bảng (mode NEW)**

Tìm bảng confirm Actions trong Phase 3 (khoảng dòng 74-80). Thêm 2 cột mới vào bảng:

Thay:

```
| ID    | Title              | Milestone | Effort | Deps      | Output            |
|-------|--------------------|-----------|--------|-----------|-------------------|
| A001  | Khảo sát user      | M1        | m      | -         | survey-report.md  |
```

Bằng:

```
| ID    | Title              | Milestone | Effort | Deps      | Output            | Notes         | Ext IDs      |
|-------|--------------------|-----------|--------|-----------|-------------------|---------------|--------------|
| A001  | Khảo sát user      | M1        | m      | -         | survey-report.md  |               |              |
| A002  | Phân tích đối thủ  | M1        | l      | -         | competitor.xlsx   | Cần data từ X | things3: T01 |
```

- [ ] **Step 7: Verify các thay đổi**

```bash
grep -n "Render bảng Roadmap\|Re-render bảng\|external_ids\|notes" skills/okr-plan/SKILL.md
```

Confirm: Phase 4 NEW có step 5 render, Phase 5 UPDATE có step 3 re-render, Phase 2 có notes + external_ids, Phase 3 bảng confirm có Notes + Ext IDs.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "feat(okr-plan): add Roadmap table rendering + notes/external_ids to plan flow"
```

---

## Task 6: Cập nhật okr-track SKILL.md, thêm sync phase + re-render bảng khi archive

**Files:**
- Modify: `skills/okr-track/SKILL.md:140-174` (Phase 4a mode LIGHT)
- Modify: `skills/okr-track/SKILL.md:196-203` (Phase 4b mode DEEP, Bước 1)

- [ ] **Step 1: Đọc Phase 4a LIGHT hiện tại**

```bash
sed -n '140,175p' skills/okr-track/SKILL.md
```

Xác nhận Phase 4a Project steps: 1 hỏi, 2 confirm, 3 áp dụng, 4 archive, 5 inbox, 6 next.

- [ ] **Step 2: Thêm sync pull TRƯỚC step 1 (hỏi update) trong Phase 4a LIGHT Project**

Tìm Phase 4a Project step 1 (khoảng dòng 146). Chèn step mới 0 TRƯỚC step 1:

```markdown
0. **Sync pull** (nếu có action với `external_ids`): chạy External Sync pull flow (xem `references/data-format.md` section "External Sync"). Diff status hiển thị trước bảng hỏi update. User confirm sync → merge vào danh sách thay đổi step 1. Không có `external_ids` → skip, không hiển thị gì.
```

Đánh lại số step: step 1 cũ → step 1, step 2 → step 2, ... (giữ nguyên số vì step 0 là prepend).

- [ ] **Step 3: Thêm sync push SAU step 3 (áp dụng) trong Phase 4a LIGHT Project**

Tìm step 3 "Áp dụng" (sau confirm). Chèn step mới sau step 3, trước step 4 (archive):

```markdown
3b. **Sync push** (nếu có action vừa thay đổi status + có `external_ids`): push status mới lên tool ngoài theo External Sync push flow. Báo kết quả.
```

- [ ] **Step 4: Thêm re-render bảng Roadmap vào step 4 archive**

Tìm step 4 "Archive actions done" (khoảng dòng 168-172). Sau dòng "Cập nhật counters frontmatter", thêm:

```markdown
  - **Re-render bảng Roadmap**: đọc lại `actions/*.md` (không archive), sinh lại bảng per-milestone trong `plan.md` body. Xóa dòng action vừa archive + xóa heading milestone trống (nếu milestone hết action).
```

- [ ] **Step 5: Thêm sync pull/push cho Phase 4b DEEP Bước 1**

Tìm Phase 4b Bước 1 (khoảng dòng 200-203). Sau "Hỏi user có update progress nào trước phân tích (giống light)", thêm:

```markdown
Bao gồm cả sync pull/push như light (step 0 + step 3b ở Phase 4a).
```

- [ ] **Step 6: Thêm re-render bảng vào Phase 4b Bước 1**

Trong cùng Bước 1, sau "archive actions done như light", thêm:

```markdown
Re-render bảng Roadmap sau archive (như Phase 4a step 4).
```

- [ ] **Step 7: Verify**

```bash
grep -n "Sync pull\|Sync push\|Re-render bảng\|External Sync" skills/okr-track/SKILL.md
```

Confirm: Phase 4a có sync pull (step 0), sync push (step 3b), re-render (step 4). Phase 4b bước 1 reference cả 3.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "feat(okr-track): add external sync phases + Roadmap table re-render on archive"
```

---

## Task 7: Cập nhật okr-init SKILL.md, migration + 6 cột resource

**Files:**
- Modify: `skills/okr-init/SKILL.md:189-199` (Phase 5 thu thập Resource, mode NEW)
- Modify: `skills/okr-init/SKILL.md:389-402` (Phase 1 mode UPDATE-RESOURCE, legacy migrate)

- [ ] **Step 1: Đọc Phase 5 mode NEW hiện tại**

```bash
sed -n '189,200p' skills/okr-init/SKILL.md
```

Xác nhận step 2 Công cụ và step 3 Tài liệu/KB.

- [ ] **Step 2: Cập nhật step 2 Công cụ (Phase 5 mode NEW) thêm 2 field mới**

Tìm step 2 Công cụ (khoảng dòng 195). Thay:

```
2. **Công cụ**: Danh sách công cụ sẽ sử dụng, khi nào dùng, dùng để làm gì, resource liên quan (URL, account).
```

Bằng:

```
2. **Công cụ**: Danh sách công cụ (6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú). Hỏi thêm: "Cách dùng" (có skill/MCP integration? hay quy trình thủ công?), "Ghi chú" (license, giới hạn, lưu ý). Xem schema `references/data-format.md` section "Schema 6 cột thống nhất".
```

- [ ] **Step 3: Cập nhật step 3 Tài liệu/KB (Phase 5 mode NEW) thêm 2 field mới**

Tìm step 3 Tài liệu/KB (khoảng dòng 196). Thay:

```
3. **Tài liệu / Knowledge Base**: Các tài liệu, hệ thống lưu trữ hiện có hoặc cần thiết (Link, folder, file), status (có sẵn/cần tạo/đang thiếu).
```

Bằng:

```
3. **Tài liệu / Knowledge Base**: Tài liệu và KB (6 cột: Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú). Status cũ gộp vào Ghi chú. Hỏi thêm: "Khi nào dùng" (trigger cụ thể), "Cách dùng" (mở link, dùng skill, search). Xem schema `references/data-format.md` section "Schema 6 cột thống nhất".
```

- [ ] **Step 4: Cập nhật Phase 1 mode UPDATE-RESOURCE, mở rộng legacy migrate**

Tìm block legacy migrate trong Phase 1 mode UPDATE-RESOURCE (khoảng dòng 399-402). Sau đoạn:

```
> **Legacy migrate**: Nếu file vẫn còn section `## Nhân sự (Vai trò & Trách nhiệm)` schema cũ, agent tự convert...
```

Thêm đoạn mới:

```markdown
> **Legacy migrate 6 cột**: Nếu bảng `## Công cụ` có 4 cột (Tên công cụ, Khi nào dùng, Mục đích, Resource) → auto-migrate sang 6 cột (thêm Cách dùng + Ghi chú trống). Nếu bảng `## Tài liệu & Knowledge Base` có 4 cột (Tên/Loại, Vị trí, Mục đích, Status) → map: Tên/Loại → Tên, Vị trí → Resource, Mục đích giữ, Status → Ghi chú. Thêm Khi nào dùng + Cách dùng trống. Thông báo user "Đã migrate bảng Công cụ/KB sang schema 6 cột. Vui lòng điền Cách dùng và Ghi chú."
```

- [ ] **Step 5: Verify**

```bash
grep -n "6 cột\|Legacy migrate 6\|Schema 6" skills/okr-init/SKILL.md
```

Confirm: Phase 5 step 2 + 3 reference 6 cột, Phase 1 update-resource có legacy migrate 6 cột.

- [ ] **Step 6: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "feat(okr-init): update resource collection to 6-column schema + migration logic"
```

---

## Task 8: Cập nhật CLAUDE.md bảng SOT

**Files:**
- Modify: `CLAUDE.md` (section "Phân vai SOT", bảng Field/Skill)

- [ ] **Step 1: Đọc bảng SOT hiện tại**

```bash
grep -A 20 "Phân vai SOT" CLAUDE.md | head -25
```

Xác nhận bảng Field/Skill được phép sửa.

- [ ] **Step 2: Thêm 2 field mới vào bảng SOT**

Tìm bảng `| Field | Skill được phép sửa |`. Thêm 2 dòng mới sau `| Inbox items (xử lý: status transition) | okr-track |`:

```markdown
| Action notes, external_ids (tạo/sửa) | `okr-plan` `new`/`update` |
| External sync (pull/push status) | `okr-track` `light`/`deep` |
```

- [ ] **Step 3: Verify**

```bash
grep -A 15 "Phân vai SOT" CLAUDE.md
```

Confirm: 2 dòng mới xuất hiện đúng vị trí.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update SOT table with notes, external_ids, and sync behavior"
```

---

## Task 9: Verify toàn bộ, cross-check consistency

**Files:**
- Read-only check tất cả files đã sửa

- [ ] **Step 1: Verify field name consistency**

```bash
grep -r "external_ids" skills/ CLAUDE.md
```

Confirm: tất cả đều dùng `external_ids` (không phải `external_id` hay `externalIds`).

```bash
grep -r "notes" skills/okr-plan/references/data-format.md skills/okr-plan/SKILL.md
```

Confirm: field `notes` consistent (không phải `note` hay `Notes` trong schema).

- [ ] **Step 2: Verify Roadmap table format consistency**

```bash
grep -r "Roadmap format\|bảng Roadmap\|Re-render bảng\|Render bảng" skills/
```

Confirm: okr-plan (render + re-render) và okr-track (re-render) đều reference cùng format từ `references/data-format.md`.

- [ ] **Step 3: Verify 6 cột consistency**

```bash
grep -r "6 cột\|Cách dùng\|Schema 6" skills/okr-init/
```

Confirm: data-format.md có schema definition, SKILL.md reference nó, migration rules nhất quán.

- [ ] **Step 4: Verify sync flow references**

```bash
grep -r "External Sync\|sync pull\|sync push" skills/okr-track/
```

Confirm: data-format.md có full spec, SKILL.md reference nó ở Phase 4a + 4b.

- [ ] **Step 5: Final commit nếu có fix**

Nếu step 1-4 phát hiện inconsistency → fix + commit:

```bash
git add -A
git commit -m "fix: resolve naming/reference inconsistencies across OKR skill files"
```

Nếu không có gì sửa → skip step này.

---

## Task 10: Cập nhật task-format.md template thêm `notes` + `external_ids`

**Files:**
- Modify: `skills/okr-plan/references/task-format.md:10-25` (frontmatter template)

Plan gốc (Tasks 1-9) cập nhật schema trong `data-format.md` nhưng bỏ sót file template. Template là cái agent đọc khi tạo action file mới. Thiếu field ở template → action mới không có `notes`/`external_ids`.

- [ ] **Step 1: Đọc template hiện tại**

```bash
cat -n skills/okr-plan/references/task-format.md | head -30
```

Xác nhận frontmatter block kết thúc ở `depends_on: []`, chưa có `notes` hay `external_ids`.

- [ ] **Step 2: Thêm `notes` và `external_ids` vào frontmatter template**

Trong file `skills/okr-plan/references/task-format.md`, tìm block:

```yaml
depends_on: []
---
```

Thay bằng:

```yaml
depends_on: []
notes: ""
external_ids: {}
---
```

- [ ] **Step 3: Verify**

```bash
grep -n "notes\|external_ids" skills/okr-plan/references/task-format.md
```

Confirm: cả 2 field xuất hiện trong frontmatter template.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/references/task-format.md
git commit -m "feat(okr-plan): add notes and external_ids to action file template"
```

---

## Task 11: Commit các thay đổi SKILL.md + CLAUDE.md còn pending

**Files:**
- Commit: `CLAUDE.md` (staged, chưa commit)
- Commit: `skills/okr-plan/SKILL.md` (unstaged, chưa commit)
- Commit: `skills/okr/SKILL.md` (unstaged, formatting)

Các thay đổi này thuộc Task 5 + Task 8 của plan gốc nhưng chưa được commit.

- [ ] **Step 1: Review diff**

```bash
git diff --stat
git diff --cached --stat
```

Confirm: 3 files thay đổi. CLAUDE.md staged, 2 SKILL.md unstaged.

- [ ] **Step 2: Stage và commit okr-plan SKILL.md (Task 5 changes)**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "feat(okr-plan): add Roadmap table rendering + notes/external_ids to plan flow"
```

- [ ] **Step 3: Commit CLAUDE.md (đã staged)**

```bash
git commit -m "docs: align SOT table formatting in CLAUDE.md"
```

- [ ] **Step 4: Stage và commit okr/SKILL.md (formatting)**

```bash
git add skills/okr/SKILL.md
git commit -m "style: align table formatting in okr orchestrator SKILL.md"
```

- [ ] **Step 5: Verify clean state**

```bash
git status
git log --oneline -5
```

Confirm: no uncommitted changes, all tasks committed.
