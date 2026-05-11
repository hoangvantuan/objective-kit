# OKR Skill — Đợt 1: Sync Tài liệu Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Đồng bộ 4 inconsistency tài liệu trong bộ skill OKR (M1, M2, M8, M9 trong review `deep-insight/okr-skill-review-260510.md`) để mọi nguồn nói cùng 1 thứ về `type: ongoing`, danh sách 5 mode track, rule KR status, và vị trí Practices schema.

**Architecture:** Doc-only refactor. Không đổi flow code, không đổi UX. Mỗi task = sửa 1 phạm vi nhỏ (1 file hoặc 1 vài sections cùng chủ đề), verify bằng `grep` trước và sau, commit ngay sau khi xanh. Không cần migration data vì chưa có user thật.

**Tech Stack:** Markdown + YAML frontmatter. Verify bằng `grep -n` / `rg`. Không cần test framework.

**Phạm vi files (6 files):**
- `docs/superpowers/specs/2026-05-09-objective-kit-design.md` (M1)
- `skills/okr-track/SKILL.md` (M2)
- `skills/okr/SKILL.md` (M2)
- `CLAUDE.md` (M2)
- `skills/okr-track/references/metrics.md` (M8)
- `skills/okr-init/references/data-format.md` (M9)
- `skills/okr-plan/references/data-format.md` (M9)

**Out of scope (sẽ làm ở đợt sau):**
- Bỏ field `verifier` khỏi action frontmatter (G10 → Đợt 2)
- Track quick mode confirm 1 dòng (K1 → Đợt 2)
- Practice schema implementation trong Phase 4a track (G2 → Đợt 3)

---

## File Structure

| File | Phạm vi sửa | Lý do |
|------|-------------|-------|
| `docs/superpowers/specs/2026-05-09-objective-kit-design.md` | Bảng "Hai loại mục tiêu" + block YAML/markdown Habit format | Spec gốc dùng `type: habit` + `frequency` + `streak`. Skill đã chuyển sang `type: ongoing` + `review_cycle` + KI ngưỡng. Spec lệch thực tế. |
| `skills/okr-track/SKILL.md` | Frontmatter `description` (line 3) | Mô tả ghi "3 mode" thiếu `inbox-only` và `trace`. |
| `skills/okr/SKILL.md` | Bảng skill con (line 23) + bảng `/okr <subcommand>` (line 117) | Liệt kê 3 mode track, thiếu `inbox-only` và `trace`. |
| `CLAUDE.md` | Bảng "Skill con (4)" cột Sub-mode dòng okr-track (line 17) | Liệt kê 4 mode, thiếu `trace`. |
| `skills/okr-track/references/metrics.md` | Section mới "Auto-compute KR Status" | Chưa document rule chuyển pending/in-progress/achieved/missed. |
| `skills/okr-init/references/data-format.md` | Body schema Ongoing (line 44): bỏ `## Practices` | Practices nằm DUY NHẤT ở plan.md, không double-source. |
| `skills/okr-plan/references/data-format.md` | Thêm section "## Practices schema" với format mẫu | Chưa có nơi nào document Practice schema chi tiết. |

---

## Task 1: M1 — Update spec `2026-05-09-objective-kit-design.md` từ `habit` sang `ongoing`

**Files:**
- Modify: `docs/superpowers/specs/2026-05-09-objective-kit-design.md:56-62` (bảng "Hai loại mục tiêu")
- Modify: `docs/superpowers/specs/2026-05-09-objective-kit-design.md:153-181` (block "Format objective.md (Habit)")

**Mục tiêu:** Spec phản ánh thực tế hiện tại của skill (`type: ongoing` + `review_cycle` + KI threshold-based, không streak/frequency).

- [ ] **Step 1: Verify state hiện tại của spec**

Run:
```bash
grep -n "type: habit\|streak\|frequency: weekly\|Habit" docs/superpowers/specs/2026-05-09-objective-kit-design.md
```

Expected: Tìm thấy ít nhất các dòng:
- 56-62: bảng cột "Habit"
- 153: heading `Format objective.md (Habit)`
- 157: `type: habit`
- 160: `frequency: weekly`
- 172: bảng KI có cột "Streak hiện tại" / "Streak dài nhất"
- 177: heading `## Checklist lặp lại`

Nếu khác đáng kể (vd: file đã được sửa) → DỪNG, đọc lại file và điều chỉnh các step bên dưới cho khớp với content hiện tại trước khi sửa.

- [ ] **Step 2: Sửa bảng "Hai loại mục tiêu" (line 56-62)**

Thay block (Edit, old_string là 7 dòng từ "| | Project (có kỳ hạn) | Habit (thói quen) |" đến "| Actions | Tasks có deadline, output, DoD | Recurring tasks, checklist lặp lại |"):

```markdown
| | Project (có kỳ hạn) | Ongoing (lĩnh vực duy trì) |
|---|---|---|
| Tính chất | Có điểm bắt đầu và kết thúc | Liên tục, không kết thúc |
| Đo lường | Key Results với target cụ thể (baseline → target) | Key Indicators với ngưỡng tối thiểu |
| Ví dụ | "Ra mắt MVP trước 30/7" | "Sức khoẻ thể chất duy trì healthy" |
| Khi nào xong | Đạt target hoặc hết deadline | Không bao giờ "xong", chỉ duy trì ở trạng thái khoẻ |
| Actions | Tasks có deadline, output, DoD | CÓ THỂ tạo action khi cần task cải thiện KI cụ thể (vd: "Mua đồ tập gym") |
```

- [ ] **Step 3: Sửa block "Format `objective.md` (Habit)" (line 153-181)**

Thay block từ heading `**Format \`objective.md\` (Habit)**:` đến hết phần "Checklist lặp lại" (gồm cả 2 code block YAML + markdown), bằng:

````markdown
**Format `objective.md` (Ongoing)**:

```yaml
---
type: ongoing
objective: "Mô tả lĩnh vực muốn duy trì"
start_date: 2026-05-09
review_cycle: weekly
status: active
---
```

```markdown
## Objective

[Mô tả WHY, tại sao lĩnh vực này quan trọng]

## Key Indicators

| # | Chỉ số | Ngưỡng tối thiểu | Current | Status |
|---|--------|------------------|---------|--------|
| KI1 | Tập thể dục | ≥3 lần/tuần | 0 | pending |
| KI2 | Ngủ đủ giấc | ≥7 giờ/đêm | 0 | pending |

KI Status: `healthy` (≥ ngưỡng) | `warning` (< ngưỡng, chênh < 20%) | `critical` (< 80% ngưỡng).

> **Lưu ý đổi tên**: Trước đây spec dùng `type: habit` + `frequency` + `streak`. Đã đổi sang `type: ongoing` + `review_cycle` + KI threshold-based. Không còn streak. Practices (hành động lặp lại) chuyển sang `plan.md` body, schema xem `skills/okr-plan/references/data-format.md`.
````

- [ ] **Step 4: Verify changes**

Run:
```bash
grep -nE "type: habit|streak|Habit \(thói quen\)|frequency: weekly|## Checklist lặp lại" docs/superpowers/specs/2026-05-09-objective-kit-design.md
```

Expected: KHÔNG có match nào (chuỗi rỗng).

Run:
```bash
grep -nE "type: ongoing|review_cycle|Ongoing \(lĩnh vực|Key Indicators|đổi sang.*ongoing" docs/superpowers/specs/2026-05-09-objective-kit-design.md
```

Expected: ≥4 match (1 trong bảng, 1 trong YAML block, 1 trong markdown table heading, 1 trong note "đổi tên").

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/specs/2026-05-09-objective-kit-design.md
git commit -m "$(cat <<'EOF'
docs: sync legacy spec habit→ongoing (M1)

Spec dùng `type: habit` + frequency + streak, lệch với skill thực
tế đã chuyển sang `type: ongoing` + review_cycle + KI threshold.
Cập nhật bảng "Hai loại mục tiêu" và block Format objective.md
(Ongoing). Thêm note giải thích đổi tên + chỉ sang plan.md cho
Practices schema.

Refs: deep-insight/okr-skill-review-260510.md M1
EOF
)"
```

---

## Task 2: M2.a — Sync 5 mode track trong `skills/okr-track/SKILL.md` description

**Files:**
- Modify: `skills/okr-track/SKILL.md:3` (frontmatter description)

**Mục tiêu:** Description liệt kê đủ 5 mode `light, deep, closure, inbox-only, trace`.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '1,5p' skills/okr-track/SKILL.md
```

Expected: dòng 3 chứa `"3 mode: light (progress nhanh), deep (review sâu + delegate), closure (chốt project)."`

- [ ] **Step 2: Sửa description**

Edit `skills/okr-track/SKILL.md`:

old_string (1 dòng, đúng nguyên văn frontmatter):
```
description: "Sub-skill của /okr. Đánh giá trạng thái + cập nhật progress + xử lý inbox + đề xuất next action. 3 mode: light (progress nhanh), deep (review sâu + delegate), closure (chốt project). Được kích hoạt từ orchestrator /okr. KHÔNG gọi trực tiếp trừ khi user gõ /okr-track."
```

new_string:
```
description: "Sub-skill của /okr. Đánh giá trạng thái + cập nhật progress + xử lý inbox + đề xuất next action. 5 mode: light (progress nhanh), deep (review sâu + delegate), closure (chốt project), inbox-only (chỉ xử lý inbox), trace (xem lại archive/log). Được kích hoạt từ orchestrator /okr. KHÔNG gọi trực tiếp trừ khi user gõ /okr-track."
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "5 mode: light, deep, closure, inbox-only, trace\|5 mode: light (progress nhanh), deep (review sâu + delegate), closure (chốt project), inbox-only (chỉ xử lý inbox), trace (xem lại archive/log)" skills/okr-track/SKILL.md
```

Expected: 1 match ở dòng 3.

Run:
```bash
grep -n "3 mode:" skills/okr-track/SKILL.md
```

Expected: KHÔNG có match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-track): list all 5 modes in skill description (M2.a)

Description ghi 3 mode, thiếu inbox-only và trace. Đồng bộ thành
5 mode đầy đủ để discovery của orchestrator khớp với reality.

Refs: deep-insight/okr-skill-review-260510.md M2
EOF
)"
```

---

## Task 3: M2.b — Sync 5 mode track trong `skills/okr/SKILL.md`

**Files:**
- Modify: `skills/okr/SKILL.md:23` (bảng "Skill con (4)" dòng okr-track)
- Modify: `skills/okr/SKILL.md:118` (bảng `/okr <subcommand>` dòng `/okr track light|deep|closure`)

**Mục tiêu:** Cả 2 bảng đề cập đủ 5 mode.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "okr-track\|track light\|track deep" skills/okr/SKILL.md
```

Expected: 
- line 23 chứa `\`light\`, \`deep\`, \`closure\``
- line 118 chứa `/okr track light\|deep\|closure`

- [ ] **Step 2: Sửa bảng skill con**

Edit `skills/okr/SKILL.md`:

old_string (1 dòng nguyên văn, backtick là backtick thật, không escape):
```
| `okr-track` | Đánh giá state + cập nhật progress + xử lý inbox + đề xuất + delegate điều chỉnh cấu trúc | `light`, `deep`, `closure` |
```

new_string:
```
| `okr-track` | Đánh giá state + cập nhật progress + xử lý inbox + đề xuất + delegate điều chỉnh cấu trúc | `light`, `deep`, `closure`, `inbox-only`, `trace` |
```

- [ ] **Step 3: Sửa bảng subcommand**

Edit `skills/okr/SKILL.md`:

old_string (1 dòng):
```
| `/okr track light\|deep\|closure` | Gọi `okr-track` mode tương ứng |
```

new_string:
```
| `/okr track light\|deep\|closure\|inbox-only\|trace` | Gọi `okr-track` mode tương ứng |
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -n "okr-track\` | .* | .*light.*deep.*closure.*inbox-only.*trace" skills/okr/SKILL.md
grep -n "/okr track light|deep|closure|inbox-only|trace" skills/okr/SKILL.md
```

Expected: cả 2 lệnh ra match (≥1 mỗi lệnh).

Run:
```bash
grep -nE "okr-track\` *\| *.* *\| *\`light\`, \`deep\`, \`closure\` *\|" skills/okr/SKILL.md
```

Expected: KHÔNG có match (đã thay).

- [ ] **Step 5: Commit**

```bash
git add skills/okr/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr): list all 5 track modes in orchestrator tables (M2.b)

Bảng skill con và bảng /okr <subcommand> đều liệt kê 3 mode track,
thiếu inbox-only và trace. Đồng bộ thành 5 mode đầy đủ.

Refs: deep-insight/okr-skill-review-260510.md M2
EOF
)"
```

---

## Task 4: M2.c — Sync 5 mode track trong `CLAUDE.md`

**Files:**
- Modify: `CLAUDE.md:17` (bảng Skill con cột Sub-mode dòng okr-track)

**Mục tiêu:** Cột Sub-mode liệt kê đủ 5 mode.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "okr-track" CLAUDE.md
```

Expected: line 17 chứa `\`light\`, \`deep\`, \`closure\`, \`inbox-only\`` (thiếu `trace`).

- [ ] **Step 2: Sửa bảng**

Edit `CLAUDE.md`:

old_string (1 dòng nguyên văn, giữ đúng số space sau `okr-track\`` và sau `inbox-only\``):
```
| `okr-track`   | Progress + review + inbox | `light`, `deep`, `closure`, `inbox-only`     |
```

new_string:
```
| `okr-track`   | Progress + review + inbox | `light`, `deep`, `closure`, `inbox-only`, `trace` |
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "okr-track" CLAUDE.md
```

Expected: line 17 chứa cả 5 mode `light`, `deep`, `closure`, `inbox-only`, `trace`.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "$(cat <<'EOF'
docs: list all 5 track modes in CLAUDE.md skill table (M2.c)

CLAUDE.md liệt kê 4 mode track, thiếu trace. Đồng bộ với
SKILL.md.

Refs: deep-insight/okr-skill-review-260510.md M2
EOF
)"
```

---

## Task 5: M8 — Document KR status auto-compute rule trong `metrics.md`

**Files:**
- Modify: `skills/okr-track/references/metrics.md` (thêm section mới sau "Tiến độ Key Result")

**Mục tiêu:** Document rõ rule chuyển status `pending → in-progress → achieved/missed` + ai compute (auto bởi track).

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "## " skills/okr-track/references/metrics.md
```

Expected: section list:
- `## Tiến độ Key Result (Project type)`
- `## Key Indicator Status (Ongoing type)`
- `## Trend (Project type)`
- `## Trend (Ongoing type)`
- `## Cập nhật SOT`
- `## Log format`

KHÔNG có section "KR Status" hay "auto-compute" hiện tại.

- [ ] **Step 2: Thêm section "KR Status auto-compute"**

Edit `skills/okr-track/references/metrics.md`:
- old_string: 
```
## Tiến độ Key Result (Project type)

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên): user cung cấp giá trị mới cho KR
2. **Tính từ actions**: đếm actions `done` thuộc KR đó / tổng actions thuộc KR

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng).

## Key Indicator Status (Ongoing type)
```
- new_string:
```
## Tiến độ Key Result (Project type)

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên): user cung cấp giá trị mới cho KR
2. **Tính từ actions**: đếm actions `done` thuộc KR đó / tổng actions thuộc KR

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng).

## KR Status auto-compute (Project type)

KR Status có 4 giá trị: `pending` | `in-progress` | `achieved` | `missed`. `okr-track` (mode `light` hoặc `deep`) tự compute lại status mỗi lần KR.current thay đổi, theo rule:

| Status | Điều kiện |
|--------|-----------|
| `pending` | `current == baseline` (chưa bắt đầu) |
| `in-progress` | `baseline < current < target` (đang chạy, chưa đạt) |
| `achieved` | `current >= target` (đạt hoặc vượt target) |
| `missed` | `current < target` AND `now > end_date` (hết hạn chưa đạt) |

**Lưu ý**:
- `achieved` ưu tiên hơn `missed`: nếu `current >= target` và `now > end_date` vẫn tính `achieved`.
- KR `missed` không tự reset: nếu user extend `end_date` qua `okr-init update-objective`, lần track kế tiếp sẽ re-compute lại (nếu `now <= end_date` mới và `current < target` → quay về `in-progress`).
- Không cần user tự set status khi tạo KR mới: mặc định `pending` (vì `current == baseline`). Khi user nhập `current` lần đầu, status tự nhảy.

## Key Indicator Status (Ongoing type)
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -nE "^## KR Status auto-compute" skills/okr-track/references/metrics.md
grep -nE "achieved.*current >= target|missed.*now > end_date" skills/okr-track/references/metrics.md
```

Expected: ≥1 match cho mỗi lệnh (section heading + ít nhất 1 row trong bảng rule).

Run:
```bash
awk '/^## /{print NR, $0}' skills/okr-track/references/metrics.md
```

Expected: section "## KR Status auto-compute (Project type)" nằm GIỮA "## Tiến độ Key Result (Project type)" và "## Key Indicator Status (Ongoing type)".

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/references/metrics.md
git commit -m "$(cat <<'EOF'
docs(okr-track): document KR status auto-compute rules (M8)

KR có 4 status (pending/in-progress/achieved/missed) nhưng rule
chuyển không document. Track tự compute mỗi lần KR.current thay
đổi, mặc định pending khi current==baseline, achieved khi
current>=target (override missed), missed khi current<target và
now>end_date.

Refs: deep-insight/okr-skill-review-260510.md M8
EOF
)"
```

---

## Task 6: M9.a — Bỏ `## Practices` khỏi schema body Ongoing trong `okr-init/references/data-format.md`

**Files:**
- Modify: `skills/okr-init/references/data-format.md:41-46` (block "Body" của Ongoing)

**Mục tiêu:** Body schema `objective.md` (Ongoing) không còn liệt kê `## Practices` (vì Practices ở plan.md, không double-source).

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -nA 6 "### Ongoing type" skills/okr-init/references/data-format.md
```

Expected: thấy block:
```
Body:
- `## Objective` (WHY: tại sao lĩnh vực này quan trọng)
- `## Key Indicators` (bảng: #, Chỉ số, Ngưỡng tối thiểu, Current, Status)
- `## Practices` (danh sách hành động thường xuyên để duy trì)
```

- [ ] **Step 2: Sửa Body section của Ongoing**

Edit `skills/okr-init/references/data-format.md`:
- old_string:
```
Body:
- `## Objective` (WHY: tại sao lĩnh vực này quan trọng)
- `## Key Indicators` (bảng: #, Chỉ số, Ngưỡng tối thiểu, Current, Status)
- `## Practices` (danh sách hành động thường xuyên để duy trì)
```
- new_string:
```
Body:
- `## Objective` (WHY: tại sao lĩnh vực này quan trọng)
- `## Key Indicators` (bảng: #, Chỉ số, Ngưỡng tối thiểu, Current, Status)

> **Practices không nằm ở `objective.md`**. Hành động lặp lại để duy trì KI nằm ở `plan.md` body section `## Practices`. Schema xem `skills/okr-plan/references/data-format.md`.
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -nE "^- \`## Practices\`" skills/okr-init/references/data-format.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "Practices không nằm ở .objective.md" skills/okr-init/references/data-format.md
```

Expected: 1 match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-init/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-init): remove Practices from objective.md schema (M9.a)

Practices có ở 2 file (objective.md + plan.md) tạo double source
of truth. Practices nằm DUY NHẤT ở plan.md body. Bỏ liệt kê khỏi
body schema của objective.md (Ongoing) + thêm note chỉ sang
okr-plan/references/data-format.md.

Refs: deep-insight/okr-skill-review-260510.md M9
EOF
)"
```

---

## Task 7: M9.b — Thêm Practices schema vào `okr-plan/references/data-format.md`

**Files:**
- Modify: `skills/okr-plan/references/data-format.md` (thêm section sau khối "## Tham chiếu objective.md")

**Mục tiêu:** Document schema mỗi practice (frequency, target_count, current_streak, description, ki_link) và format trong body `plan.md`. Thay thế chỗ hiện tại chỉ nói "plan.md body chứa `## Practices` (hành động lặp lại)" mà không có schema.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "Practices\|## " skills/okr-plan/references/data-format.md
```

Expected: thấy:
- `## plan.md`
- `## actions/AXXX-slug.md`
- `## Tham chiếu objective.md`
- `## actions/archive/`
- 1 mention "## Practices" trong text của section "Tham chiếu objective.md" (line ~53), không có section riêng và không có schema.

- [ ] **Step 2: Sửa câu nhắc Practices trong "Tham chiếu objective.md"**

Edit `skills/okr-plan/references/data-format.md`:

old_string:
```
Nếu type=ongoing, plan.md body chứa `## Practices` (hành động lặp lại). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI cụ thể (vd: "Mua đồ tập gym"). Actions tuân quy tắc bình thường. Milestones không bắt buộc với Ongoing.
```

new_string:
```
Nếu type=ongoing, plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Schema xem section "## Practices schema" bên dưới. Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI cụ thể (vd: "Mua đồ tập gym"). Actions tuân quy tắc bình thường. Milestones không bắt buộc với Ongoing.
```

- [ ] **Step 3: Thêm section "## Practices schema" trước "## actions/archive/"**

Edit `skills/okr-plan/references/data-format.md`. `old_string` là nguyên 1 dòng heading hiện có `## actions/archive/` (string này unique trong file, đã verify ở Step 1). `new_string` là toàn bộ block dưới đây (gồm section mới + heading cũ ở cuối):

old_string:
```
## actions/archive/
```

new_string:
````
## Practices schema (chỉ Ongoing)

Practices là hành động lặp lại theo chu kỳ để duy trì KI healthy. Nằm trong body `plan.md` dưới heading `## Practices`. KHÔNG nằm ở `objective.md`. KHÔNG có frontmatter riêng (đọc từ markdown body).

Mỗi practice là 1 sub-heading `### PN: <Tên>` với các field bullet:

```markdown
## Practices

### P1: Tập thể dục
- frequency: weekly
- target_count: 3
- current_streak: 2
- description: Cardio + strength, mỗi buổi 45 phút
- ki_link: KI1

### P2: Ngủ đủ giấc
- frequency: daily
- target_count: 7
- current_streak: 5
- description: Đi ngủ trước 23:00, ngủ ≥7 giờ
- ki_link: KI2
```

Field meanings:

| Field | Type | Mô tả |
|-------|------|-------|
| `frequency` | `daily` \| `weekly` \| `biweekly` \| `monthly` | Chu kỳ practice |
| `target_count` | int | Số lần mục tiêu trong 1 chu kỳ (vd: 3 lần/tuần) |
| `current_streak` | int | Số chu kỳ liên tiếp gần nhất đã đạt `target_count` |
| `description` | string | Mô tả ngắn cách thực hiện |
| `ki_link` | `KIN` | KI mà practice này phục vụ (link tới Key Indicator trong `objective.md`) |

Quy tắc:
- ID practice (`P1`, `P2`...) duy nhất trong file. Không reuse khi xoá.
- 1 practice link tới đúng 1 KI. 1 KI có thể có nhiều practices.
- `current_streak` được update bởi `okr-track` Ongoing flow (Phase 4a Ongoing): hỏi user "Tuần này đạt practice nào?" → tăng/reset streak.
- `okr-plan` mode `update` thêm/sửa/xoá Practices structure (frequency, target_count, description, ki_link). KHÔNG sửa `current_streak`.

## actions/archive/
````

- [ ] **Step 4: Verify**

Run:
```bash
grep -nE "^## Practices schema|frequency.*daily.*weekly|ki_link" skills/okr-plan/references/data-format.md
```

Expected: ≥3 match (heading section + bảng field type + ki_link mention).

Run:
```bash
awk '/^## /{print NR, $0}' skills/okr-plan/references/data-format.md
```

Expected: order các section là `plan.md` → `actions/AXXX-slug.md` → `Tham chiếu objective.md` → `Practices schema (chỉ Ongoing)` → `actions/archive/`.

Run:
```bash
grep -n "## Practices schema" skills/okr-plan/references/data-format.md
```

Expected: 1 match.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-plan/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): add Practices schema for Ongoing (M9.b)

Practices được nhắc trong plan.md nhưng không có schema field
nào được document. Thêm section riêng với format markdown mẫu
(### PN: <Tên> + bullet fields), bảng field types, và quy tắc
ownership: okr-plan sửa structure, okr-track sửa current_streak.

Refs: deep-insight/okr-skill-review-260510.md M9
EOF
)"
```

---

## Task 8: Final cross-check toàn đợt 1

**Mục tiêu:** Quét lại toàn bộ repo đảm bảo không còn rác `habit/streak/3 mode`, và 7 commit nằm trên branch hiện tại.

- [ ] **Step 1: Quét legacy `type: habit` toàn skills/ và docs/**

Run:
```bash
grep -rn "type: habit\|frequency: weekly\|streak hiện tại\|streak dài nhất" skills/ docs/ CLAUDE.md
```

Expected: KHÔNG có match nào. Nếu có → DỪNG, sửa file đó với cùng pattern Ongoing như Task 1 trước khi tiếp.

- [ ] **Step 2: Quét "3 mode" trong skills/ và CLAUDE.md**

Run:
```bash
grep -rnE "3 mode|3 modes" skills/ CLAUDE.md
```

Expected: KHÔNG có match. (Acceptable nếu chỉ xuất hiện trong `deep-insight/` review file - quét đã loại trừ.)

- [ ] **Step 3: Verify Practices schema không nằm 2 chỗ**

Run:
```bash
grep -rn "## Practices" skills/
```

Expected: 1 match duy nhất ở `skills/okr-plan/references/data-format.md` (heading section schema). KHÔNG có match ở `okr-init/references/data-format.md`.

- [ ] **Step 4: Verify danh sách 5 mode track xuất hiện ở 4 nơi**

Run:
```bash
grep -rnE "light.*deep.*closure.*inbox-only.*trace|light, deep, closure, inbox-only, trace|/okr track light\|deep\|closure\|inbox-only\|trace" skills/okr-track/SKILL.md skills/okr/SKILL.md CLAUDE.md
```

Expected: ≥4 match (1 ở okr-track/SKILL.md, 2 ở okr/SKILL.md, 1 ở CLAUDE.md).

- [ ] **Step 5: Verify KR status rule có trong metrics.md**

Run:
```bash
grep -nE "KR Status auto-compute|baseline < current < target|current >= target.*achieved|now > end_date.*missed" skills/okr-track/references/metrics.md
```

Expected: ≥4 match (heading + 3 row rule).

- [ ] **Step 6: Verify lịch sử git**

Run:
```bash
git log --oneline -8
```

Expected: 7 commit gần nhất chứa các tag `(M1)`, `(M2.a)`, `(M2.b)`, `(M2.c)`, `(M8)`, `(M9.a)`, `(M9.b)` theo thứ tự.

- [ ] **Step 7: Status sạch**

Run:
```bash
git status
```

Expected: `nothing to commit, working tree clean` (ngoại trừ untracked files đã có trước khi bắt đầu plan: `deep-insight/`, `docs/okr-system-review.md`).

---

## Tiêu chí hoàn thành Đợt 1

- [ ] 7 commit nhỏ trên branch hiện tại, mỗi commit reference review item (M1, M2.a-c, M8, M9.a-b).
- [ ] `grep "type: habit"` toàn `skills/` + `docs/` + `CLAUDE.md` ra rỗng.
- [ ] `grep "3 mode"` toàn `skills/` + `CLAUDE.md` ra rỗng.
- [ ] Danh sách 5 mode track xuất hiện ở 4 vị trí (okr-track/SKILL.md, okr/SKILL.md ×2, CLAUDE.md).
- [ ] Section "KR Status auto-compute" xuất hiện trong `skills/okr-track/references/metrics.md`.
- [ ] `## Practices` chỉ tồn tại ở `skills/okr-plan/references/data-format.md` (1 heading section).
- [ ] Schema mỗi Practice (frequency, target_count, current_streak, description, ki_link) document đầy đủ.

## Sau Đợt 1

Mở plan Đợt 2 (Solo defaults, 6 items breaking-schema): cần quyết định migration strategy nếu đã có user dùng `verifier` field, type `idea`/`note` trong inbox.
