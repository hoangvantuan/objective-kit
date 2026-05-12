# Output Tracking Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Mở rộng okr-track để ghi output thực tế vào action file và log khi action hoàn thành.

**Architecture:** okr-track ghi đè section `## Output/Deliverable` trong action body (không thêm field frontmatter mới). Đồng thời append output vào log entry. okr-plan vẫn tạo section dự kiến lúc planning, okr-track ghi đè bằng output thực khi user cung cấp.

**Tech Stack:** Markdown prompts (không có code runtime)

---

## File Map

| File | Hành động | Trách nhiệm |
|------|-----------|-------------|
| `skills/okr/references/sot-ownership.md` | Modify:9-10 | Thêm dòng SOT cho `## Output/Deliverable` (actual) thuộc okr-track |
| `skills/okr-track/references/data-format.md` | Modify:20-29 | Thêm `A003.output:` vào schema section `## Thay đổi` |
| `skills/okr-track/references/flow-light.md` | Modify:36-41 | Thêm bước nhắc output sau archive, trước re-render |
| `skills/okr-track/references/flow-deep.md` | Modify:5-7 | Thêm output prompt vào Bước 1 (reuse logic light) |
| `skills/okr-track/SKILL.md` | Modify:12-14, 148-153, 168-169, 249-254 | Thêm capability ghi output, cập nhật progress fields list, phase summaries, rules |
| `CLAUDE.md` | Modify:48-49 | Sync bảng SOT: thêm dòng output |

---

### Task 1: Cập nhật SOT ownership (canonical)

**Files:**
- Modify: `skills/okr/references/sot-ownership.md:9-15`

- [ ] **Step 1: Thêm dòng output vào bảng SOT**

Mở file `skills/okr/references/sot-ownership.md`. Sau dòng 9 (action structure), thêm dòng mới cho output:

```markdown
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| Action `## Output/Deliverable` (ghi đè output thực tế)           | `okr-track` `light`/`deep`    |
```

Dòng 9 giữ nguyên (okr-plan tạo deliverable dự kiến). Dòng mới thêm ngay sau dòng 9.

Kết quả bảng SOT sau khi sửa (toàn bộ):

```markdown
| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| Action `## Output/Deliverable` (ghi đè output thực tế)           | `okr-track` `light`/`deep`    |
| KR.current, KI.current, plan counters                             | `okr-track` `light`/`deep`    |
| action.status                                                     | `okr-track` `light`/`deep`, `okr-plan` `update` |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`     |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`    |
```

- [ ] **Step 2: Commit**

```bash
git add skills/okr/references/sot-ownership.md
git commit -m "docs(okr): add output/deliverable SOT ownership for okr-track"
```

---

### Task 2: Cập nhật log schema (data-format.md)

**Files:**
- Modify: `skills/okr-track/references/data-format.md:20-29`

- [ ] **Step 1: Thêm output vào ví dụ tracking entry**

Mở file `skills/okr-track/references/data-format.md`. Sửa block tracking (dòng 20-29) để thêm `output` vào schema `## Thay đổi`:

```markdown
**tracking** (từ light hoặc deep Bước 1 update progress):

~~~markdown
## Thay đổi
- KR1.current: 40 > 50
- A003.status: doing > done
- A003.output:
  - [report.md](docs/report.md)
  - https://docs.google.com/spreadsheets/d/xxx

## Ghi chú
- A005 vẫn blocked
~~~
```

- [ ] **Step 2: Thêm mô tả output vào progress fields**

Cùng file, sau dòng 83 (`status` field của action), thêm mô tả output:

```markdown
- Body section `## Output/Deliverable`: ghi đè nội dung dự kiến bằng output thực tế (free-form markdown). Chỉ ghi khi user cung cấp output.
```

Kết quả section progress fields cho action (dòng 82+):

```markdown
`.okr/actions/AXXX-*.md`:
- Frontmatter `status`: `pending` | `doing` | `done` | `blocked` (cũng được sửa bởi `okr-plan` `update`)
- Body section `## Output/Deliverable`: ghi đè nội dung dự kiến bằng output thực tế (free-form markdown). Chỉ ghi khi user cung cấp output.
```

- [ ] **Step 3: Commit**

```bash
git add skills/okr-track/references/data-format.md
git commit -m "docs(okr-track): add output field to log schema and progress fields"
```

---

### Task 3: Cập nhật flow-light.md

**Files:**
- Modify: `skills/okr-track/references/flow-light.md:36-41`

Bước output phải nằm SAU archive actions done (step 4), TRƯỚC re-render Roadmap. Lý do: cần ghi output vào action file trước khi archive di chuyển file vào `archive/`. Tuy nhiên archive chạy trước, nên output cần ghi TRƯỚC archive. Xét lại flow:

Spec nói: "nhắc khi mark done". Thời điểm tốt nhất: TRƯỚC archive (vì archive di chuyển file). Nên chèn bước output giữa step 3 (apply) và step 4 (archive).

- [ ] **Step 1: Chèn bước nhắc output vào Project type flow**

Mở file `skills/okr-track/references/flow-light.md`. Sửa phần Project type. Sau step 3 (áp dụng + log) và TRƯỚC step 4 (archive), chèn bước output mới. Đồng thời cập nhật numbering:

Thay thế từ dòng 31 đến dòng 41 bằng:

```markdown
3. Áp dụng:
  - Ghi đè progress: `objective.md` (KR.current, KR status), `plan.md` (counters), `actions/*.md` (frontmatter status).
  - Append log: `.okr/log/YYYY-MM-DD.md`. File ngày đã có → append section mới.

3b. **Sync push** (nếu có action vừa thay đổi status + có `external_ids`): push status mới lên tool ngoài theo External Sync push flow. Báo kết quả.
4. **Nhắc output** (nếu có action chuyển sang `done` trong lần này): với mỗi action vừa done, kiểm tra body section `## Output/Deliverable`. Nếu section vẫn là mô tả dự kiến (do okr-plan tạo), hỏi:

  ```
  A003 done. Output là gì? (file path, link, text... hoặc "skip")
  ```

  - User cung cấp output → ghi đè section `## Output/Deliverable` trong action file bằng output thực tế (free-form markdown). Append output vào log entry (format: `A003.output:` + danh sách items).
  - User nói "skip" → giữ nguyên section dự kiến, không ghi output vào log.
  - Nếu section đã được ghi đè trước đó (user đã ghi output giữa chừng) → skip, không hỏi lại.

5. **Archive actions done**: chạy Archive Rules (xem `okr/references/shared-schemas.md` section "Archive Rules"). Sau đó:

  - **Re-render bảng Roadmap**: đọc lại tất cả `actions/*.md` (không archive), sinh lại toàn bộ bảng per-milestone trong `plan.md` body. Format xem `okr/references/shared-schemas.md` section "Roadmap format".

6. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
7. Đề xuất next action: highlight việc cần làm trong 1-7 ngày tới.
```

- [ ] **Step 2: Thêm ghi output chủ động vào step 2**

Cùng file, sửa step 2 (dòng 8) để thêm khả năng ghi output bất kỳ lúc nào (không chỉ khi mark done). Thay dòng 8:

Từ:
```markdown
2. Hỏi user có thay đổi: KR current, action status (pending/doing/done/blocked), blocker mới.
```

Thành:
```markdown
2. Hỏi user có thay đổi: KR current, action status (pending/doing/done/blocked), blocker mới, output (nếu user chủ động cung cấp).
  - User cung cấp output cho action đang `doing` hoặc `done` → ghi đè section `## Output/Deliverable` trong action file. Không cần chờ mark done.
```

- [ ] **Step 3: Thêm output vào confirm UI**

Cùng file, sửa ví dụ confirm bảng (dòng 21-28) để bao gồm output:

Thêm dòng output vào ví dụ bảng ≥3 field:

```markdown
- **Nếu ≥3 field thay đổi** → confirm bảng đầy đủ:
  ```
  Thay đổi sắp ghi
  - KR1.current: 40 > 50
  - A003.status: doing > done
  - A003.output: [report.md](docs/report.md), https://docs.google.com/...
  - A005.status: doing > blocked, lý do: chờ approve
  Xác nhận? (y/sửa/huỷ)
  ```
```

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/references/flow-light.md
git commit -m "docs(okr-track): add output prompt to light flow"
```

---

### Task 4: Cập nhật flow-deep.md

**Files:**
- Modify: `skills/okr-track/references/flow-deep.md:5-7`

- [ ] **Step 1: Thêm output vào Bước 1**

Mở file `skills/okr-track/references/flow-deep.md`. Sửa Bước 1 (dòng 5-7):

Từ:
```markdown
## Bước 1: Update progress nếu cần

Hỏi user có update progress nào trước phân tích (giống light). Nếu có → áp dụng + log + **archive actions done** như light (Phase 4a bước 4). Bao gồm sync pull/push (step 0 + 3b) và re-render Roadmap (step 4) như light.
```

Thành:
```markdown
## Bước 1: Update progress nếu cần

Hỏi user có update progress nào trước phân tích (giống light). Nếu có → áp dụng + log + **nhắc output cho actions done** (step 4 light) + **archive actions done** như light (Phase 4a bước 5). Bao gồm sync pull/push (step 0 + 3b) và re-render Roadmap (step 5) như light.
```

- [ ] **Step 2: Commit**

```bash
git add skills/okr-track/references/flow-deep.md
git commit -m "docs(okr-track): add output prompt reference to deep flow"
```

---

### Task 5: Cập nhật SKILL.md

**Files:**
- Modify: `skills/okr-track/SKILL.md:12-14, 148-153, 249-254`

- [ ] **Step 1: Thêm output vào phân vai**

Mở file `skills/okr-track/SKILL.md`. Sửa section phân vai (dòng 10-14):

Từ:
```markdown
**Phân vai rõ ràng**:

- `okr-track` ghi đè **progress fields** (KR.current, KI.current, action.status) và ghi log.
- `okr-track` **xử lý inbox**: phân loại items → delegate hoặc tự apply tuỳ loại.
- Thay đổi **cấu trúc** (KR target, action mới, dời deadline, đổi PIC) → delegate sang `okr-init` hoặc `okr-plan` mode `update-*`. Track đề xuất, init/plan áp dụng.
```

Thành:
```markdown
**Phân vai rõ ràng**:

- `okr-track` ghi đè **progress fields** (KR.current, KI.current, action.status, action body `## Output/Deliverable`) và ghi log.
- `okr-track` **xử lý inbox**: phân loại items → delegate hoặc tự apply tuỳ loại.
- Thay đổi **cấu trúc** (KR target, action mới, dời deadline, đổi PIC) → delegate sang `okr-init` hoặc `okr-plan` mode `update-*`. Track đề xuất, init/plan áp dụng.
```

- [ ] **Step 2: Thêm output vào Phase 4a summary**

Sửa Phase 4a summary (dòng 147-153):

Từ:
```markdown
1. Sync pull external (nếu action có `external_ids`)
2. Hỏi user update: KR current, action status, blocker mới
3. CONFIRM: ≤2 field → 1 dòng compact, ≥3 field → bảng đầy đủ
4. Ghi SOT (objective, plan counters, action status) + append log
5. Sync push + archive actions done + re-render Roadmap
6. Xử lý inbox (Phase 5) nếu có items pending
7. Đề xuất next action (1-7 ngày tới)
```

Thành:
```markdown
1. Sync pull external (nếu action có `external_ids`)
2. Hỏi user update: KR current, action status, blocker mới, output (nếu user chủ động cung cấp)
3. CONFIRM: ≤2 field → 1 dòng compact, ≥3 field → bảng đầy đủ
4. Ghi SOT (objective, plan counters, action status) + append log
5. Sync push
6. Nhắc output cho actions vừa done (ghi đè `## Output/Deliverable`, append log)
7. Archive actions done + re-render Roadmap
8. Xử lý inbox (Phase 5) nếu có items pending
9. Đề xuất next action (1-7 ngày tới)
```

- [ ] **Step 3: Thêm output vào Phase 4b summary**

Sửa Phase 4b summary (dòng 169):

Từ:
```markdown
1. Update progress nếu cần (giống light, kèm sync + archive)
```

Thành:
```markdown
1. Update progress nếu cần (giống light, kèm sync + nhắc output + archive)
```

- [ ] **Step 4: Thêm output vào quy tắc**

Sửa section quy tắc (dòng 249-254). Thêm 2 rule mới sau dòng 254:

```markdown
- Ghi đè SOT progress fields, append log. Không ngược lại.
- Output: ghi đè section `## Output/Deliverable` trong action body. Nhắc khi mark done, cho phép skip. Cũng chấp nhận output chủ động bất kỳ lúc nào (action đang doing hoặc done). Append output vào log entry.
```

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "docs(okr-track): add output tracking capability to skill definition"
```

---

### Task 6: Sync bảng SOT trong CLAUDE.md

**Files:**
- Modify: `CLAUDE.md:44-54`

- [ ] **Step 1: Thêm dòng output vào bảng SOT**

Mở file `CLAUDE.md`. Sửa bảng SOT (dòng 44-54). Thêm dòng mới sau dòng 48 (action structure):

Từ:
```markdown
| Milestones, action structure (title, deadline, deps)         | `okr-plan` `update`                              |
| KR.current, KI.current, plan counters                        | `okr-track` `light`/`deep`                       |
```

Thành:
```markdown
| Milestones, action structure (title, deadline, deps)         | `okr-plan` `update`                              |
| Action `## Output/Deliverable` (ghi đè output thực tế)      | `okr-track` `light`/`deep`                       |
| KR.current, KI.current, plan counters                        | `okr-track` `light`/`deep`                       |
```

Chú ý: CLAUDE.md dùng `deps` (không có `deliverable` như canonical). Giữ nguyên dòng action structure, chỉ thêm dòng output mới.

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: sync SOT table with output tracking ownership"
```

---

### Task 7: Kiểm tra consistency

**Files:** Tất cả file đã sửa

- [ ] **Step 1: Cross-check SOT ownership**

Verify 2 bảng SOT khớp nhau:
- `skills/okr/references/sot-ownership.md` (canonical)
- `CLAUDE.md` (readable copy)

Cả 2 phải có dòng output thuộc `okr-track`.

- [ ] **Step 2: Cross-check flow references**

Verify `flow-deep.md` Bước 1 tham chiếu đúng step number trong `flow-light.md`:
- Output prompt = step 4 (light)
- Archive = step 5 (light)
- Re-render Roadmap = step 5 (light)

- [ ] **Step 3: Cross-check SKILL.md summary vs flow detail**

Verify Phase 4a summary trong SKILL.md khớp step numbers trong flow-light.md:
- Step 6 = nhắc output
- Step 7 = archive + re-render

- [ ] **Step 4: Verify data-format.md ví dụ khớp flow**

Verify log format `A003.output:` trong data-format.md khớp format mô tả trong flow-light.md step 4.

- [ ] **Step 5: Commit fix (nếu có)**

```bash
git add -A
git commit -m "docs(okr-track): fix consistency issues in output tracking"
```

Nếu không có issue → skip commit này.
