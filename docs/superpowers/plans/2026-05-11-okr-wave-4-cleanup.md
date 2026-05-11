# OKR Skill — Đợt 4: Dọn Thừa Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Dọn duplicate + refactor sau khi đã sync doc (Đợt 1), gỡ team bias (Đợt 2), lấp lifecycle gap (Đợt 3). Đợt này: (T1) merge bảng State + Keyword routing trong okr/SKILL.md, (T2) tách Quality Gate ra shared file, (T3) Phân vai SOT về single source ở CLAUDE.md, (T4) gom confirm track deep + handler `pre_confirmed` ở init/plan, (T5) merge capture Phase 5 reminder vào Phase 4, (M5) capture chỉ ghi không validate + track Phase 5 validate ID, (M7) okr-plan update bypass menu khi nhận inbox pre-processed. Đồng thời fix 2 minor issues từ final review Đợt 3: heading `### Phase 0` trùng ở okr-init, practice streak step 4 thiếu fallback khi không có `## Practices`.

**Architecture:** Toàn bộ là markdown refactor. Mỗi task = sửa 1 phạm vi rõ ràng trong 1-2 file, verify bằng `grep` trước/sau, commit ngay khi xanh. T2 tạo 1 file mới (`skills/okr/references/quality-gate.md`). T3 đụng nhiều file (CLAUDE.md, okr/SKILL.md, okr-track/references/data-format.md, docs/okr-system-review.md). T4 + M7 ghép thành 3 task (track wording + init handler + plan handler + plan bypass menu).

**Tech Stack:** Markdown + YAML frontmatter. Verify bằng `grep -nE` / `awk`. Không cần test framework.

**Phạm vi files (10 files):**

- `skills/okr/SKILL.md` (T1 — merge Bước 3 bảng routing; T3 — Phân vai SOT link sang CLAUDE.md)
- `skills/okr/references/quality-gate.md` (T2 — NEW shared file)
- `skills/okr-init/SKILL.md` (T2 — ref quality-gate.md; T4b — Phase 6 update-objective pre_confirmed handler; Minor1 — rename heading Phase 0 trùng)
- `skills/okr-plan/SKILL.md` (T2 — ref quality-gate.md; T4c — Phase 4 update pre_confirmed handler; M7 — Phase 1 update bypass menu khi từ track inbox)
- `skills/okr-track/SKILL.md` (T4a — Bước 4 gom all-changes confirm strengthened; M5b — Phase 5 Bước 3 validate ID; Minor2 — Phase 4a step 4 Ongoing fallback note)
- `skills/okr-track/references/data-format.md` (T3 — Phân vai SOT link sang CLAUDE.md)
- `skills/okr-capture/SKILL.md` (T5 — merge Phase 5 reminder vào Phase 4; M5a — clarify "không validate, track làm")
- `CLAUDE.md` (T3 — thêm section Phân vai SOT làm single source)
- `docs/okr-system-review.md` (T3 — link sang CLAUDE.md thay vì copy)

**Out of scope (sau Đợt 4):**

- Multi-objective workspace (G1 đã defer)
- Forecast / extrapolation (G7 đã defer)
- Pause/Resume snapshot (G12 đã defer)

---

## File Structure

| File | Phạm vi sửa | Lý do |
|------|-------------|-------|
| `skills/okr/SKILL.md` | Bước 3 (line ~66-91): merge 2 bảng routing; Phân vai SOT (line ~26-37): rút gọn + link sang CLAUDE.md | T1 + T3. |
| `skills/okr/references/quality-gate.md` | NEW file: chứa Quality Gate 3 câu + bảng hành vi + Deepening Techniques generic | T2 single source. |
| `skills/okr-init/SKILL.md` | Section `## Quality Gate` (line 17-33): rút gọn + link sang quality-gate.md; Phase 6 CONFIRM update-objective: pre_confirmed handler; line 51 `### Phase 0` đổi tên thành `### Phase 0a: Đọc inbox làm context` | T2 + T4b + Minor1. |
| `skills/okr-plan/SKILL.md` | Section `## Quality Gate` (line 15-30): rút gọn + link sang quality-gate.md; Mode UPDATE Phase 1 (line ~125-127): thêm bypass note khi từ track inbox; Phase 4 CONFIRM update: pre_confirmed handler | T2 + T4c + M7. |
| `skills/okr-track/SKILL.md` | Phase 4b Bước 4 + Bước 5: explicit all-changes diff confirm với pre_confirmed; Phase 5 Bước 3: thêm validate `related_kr/action` ID; Phase 4a Ongoing step 4: fallback note khi plan.md không có `## Practices` | T4a + M5b + Minor2. |
| `skills/okr-track/references/data-format.md` | Section "Structure fields" line 58-69: link sang CLAUDE.md#phân-vai-sot, giữ bảng nhưng note tham chiếu CLAUDE.md là canonical | T3. |
| `skills/okr-capture/SKILL.md` | Phase 4 (line 79-83) + Phase 5 (line 85-87): merge thành Phase 4 có reminder nếu inbox ≥5; Nguyên tắc / Quy tắc: clarify "Không validate related, track làm" | T5 + M5a. |
| `CLAUDE.md` | Thêm section mới "## Phân vai SOT" sau "## Skill con" với bảng field → skill | T3 single source. |
| `docs/okr-system-review.md` | Section 9 "Phân vai SOT (tham chiếu nhanh)" line 352-362: link sang CLAUDE.md#phân-vai-sot, xoá bảng inline (đã có ở CLAUDE.md). Cũng cập nhật row "người, PIC, khả dụng" cho phù hợp solo | T3 + cleanup. |

---

## Thứ tự thực thi

T1 sửa Bước 3 okr/SKILL.md (line cao). T3 sửa Phân vai SOT okr/SKILL.md (line thấp hơn) + thêm CLAUDE.md + sửa data-format + sửa system-review. Làm T1 trước, T3 sau để tránh shift line.

T2 tạo file mới + sửa init + plan section đầu. Làm sau T1+T3 (không đụng cùng region).

T4 + M7 + Minor1: cùng đụng init + plan. Làm theo thứ tự T4b (init) → Minor1 (init Phase 0 rename, không trùng region với T4b) → M7 + T4c (plan), rồi T4a (track Bước 4-5).

T5 + M5a: cùng đụng capture (Phase 4-5 + Nguyên tắc). Làm M5a + T5 trong 1 task.

M5b + Minor2: cùng đụng track. Làm sau T4a.

Thứ tự cuối: 1 (T1) → 2 (T3) → 3 (T2) → 4 (Minor1) → 5 (M7+T4c) → 6 (T4b) → 7 (T4a) → 8 (T5+M5a) → 9 (M5b) → 10 (Minor2) → 11 (Final cross-check).

---

## Task 1: T1 — Merge bảng State/Intent + Keyword routing trong `okr/SKILL.md`

**Files:**

- Modify: `skills/okr/SKILL.md:66-91` (Bước 3: 2 bảng routing)

**Mục tiêu:** Hiện tại Bước 3 có 1 bảng "State / Intent | Skill + Mode" (line 68-80) và 1 list "Keyword routing khi user cung cấp context" (line 82-91). Trùng thông tin (vd: "User nhắc tài liệu/capacity" xuất hiện cả 2 chỗ). Merge thành 1 bảng 3 cột: **Trigger** (state hoặc keyword) | **Skill** | **Mode**. Mỗi row hợp nhất nếu trigger giống nhau.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^### Bước 3|^\| State|^Keyword routing|^### Bước 4" skills/okr/SKILL.md
```

Expected: Match `### Bước 3` (line ~66), `| State / Intent` (line ~68 bảng), `Keyword routing khi user cung cấp context:` (line ~82), `### Bước 4` (line ~93).

- [ ] **Step 2: Thay 2 bảng/list bằng 1 bảng merged**

Edit `skills/okr/SKILL.md`:

old_string (toàn bộ Bước 3 hiện tại, từ heading đến trước Bước 4):

```
### Bước 3: Suy luận next action

| State / Intent | Skill + Mode |
|----------------|--------------|
| `.okr/` chưa có HOẶC objective.md thiếu | `okr-init` mode `new` |
| User nhắc objective/KR/KI/period/mục tiêu | `okr-init` mode `update-objective` |
| User nhắc capacity/skill/tài nguyên/ngân sách/tool | `okr-init` mode `update-resource` |
| Có objective + resource, thiếu plan.md | `okr-plan` mode `new` |
| Có plan.md + user nhắc thêm/sửa action, dời deadline, đổi milestone | `okr-plan` mode `update` |
| User nhắc kế hoạch/lộ trình/milestone (mới) | `okr-plan` mode `new` |
| Đủ SOT + actions còn mở + user nhắc tiến độ/cập nhật/xong/blocked | `okr-track` mode `light` |
| Đủ SOT + user nhắc review/tổng kết/lookback/đánh giá sâu | `okr-track` mode `deep` |
| Mọi action `done` | `okr-track` mode `closure` |
| User nhắc thêm nhanh/ghi lại/capture/inbox/note | `okr-capture` |
| User nhắc trace/xem lại/history/lịch sử action/milestone/log | `okr-track` mode `trace` |

Keyword routing khi user cung cấp context:
- "tài liệu / tài nguyên / capacity / skill / công cụ / ngân sách / Solo Profile" → `okr-init` `update-resource`
- "đổi target / sửa KR / sửa KI / đổi ngưỡng / đổi deadline objective" → `okr-init` `update-objective`
- "kế hoạch / plan / lộ trình / milestone / action mới" → `okr-plan` `new` (nếu chưa có) hoặc `update`
- "thêm action / sửa action / dời deadline / xoá action" → `okr-plan` `update`
- "khởi tạo / mục tiêu mới / dự án mới" → `okr-init` `new`
- "tiến độ / cập nhật / xong rồi / blocked" → `okr-track` `light`
- "review / tổng kết / lookback / đánh giá" → `okr-track` `deep`
- "thêm nhanh / ghi lại / note / capture / nhớ cái này / inbox" → `okr-capture`
- "trace / xem lại / history / lịch sử / xem action cũ / xem log cũ" → `okr-track` `trace`

```

new_string:

```
### Bước 3: Suy luận next action

Bảng routing dưới đây gộp 2 nguồn tín hiệu: (a) **State** = trạng thái `.okr/` đọc được ở Bước 1, (b) **Keyword** = từ khoá user nói khi gọi `/okr`. Đọc theo thứ tự State trước, Keyword sau. Nếu State + Keyword chỉ về cùng skill → đi thẳng. Nếu mâu thuẫn → ưu tiên Keyword (user đang nói rõ ý) và xác nhận lại ở Bước 4.

| Trigger (State hoặc Keyword) | Skill | Mode |
|------------------------------|-------|------|
| State: `.okr/` chưa có HOẶC `objective.md` thiếu | `okr-init` | `new` |
| Keyword: "khởi tạo / mục tiêu mới / dự án mới" | `okr-init` | `new` |
| Keyword: "đổi target / sửa KR / sửa KI / đổi ngưỡng / đổi deadline objective / objective / period" | `okr-init` | `update-objective` |
| Keyword: "tài liệu / tài nguyên / capacity / skill / công cụ / ngân sách / Solo Profile" | `okr-init` | `update-resource` |
| State: có objective + resource, thiếu `plan.md` | `okr-plan` | `new` |
| Keyword: "kế hoạch / plan / lộ trình / milestone mới" (chưa có `plan.md`) | `okr-plan` | `new` |
| Keyword: "thêm action / sửa action / dời deadline / xoá action / sửa milestone" | `okr-plan` | `update` |
| State: đủ SOT + actions còn mở + keyword tiến độ/cập nhật/xong/blocked | `okr-track` | `light` |
| Keyword: "review / tổng kết / lookback / đánh giá sâu" | `okr-track` | `deep` |
| State: mọi action `done` | `okr-track` | `closure` |
| Keyword: "thêm nhanh / ghi lại / note / capture / nhớ cái này / inbox" | `okr-capture` | n/a |
| Keyword: "trace / xem lại / history / lịch sử / xem action cũ / xem log cũ" | `okr-track` | `trace` |
| Keyword: "inbox / xử lý inbox" (skip update progress) | `okr-track` | `inbox-only` |

```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "^### Bước 3|^Keyword routing|^\| Trigger" skills/okr/SKILL.md
```

Expected: Match `### Bước 3` và `| Trigger`. KHÔNG còn `Keyword routing khi user cung cấp context:` (đã merge vào bảng).

Run:

```bash
awk '/^### Bước 3/,/^### Bước 4/' skills/okr/SKILL.md | grep -cE "^\|"
```

Expected: ≥14 (header + separator + ≥12 row).

- [ ] **Step 4: Commit**

```bash
git add skills/okr/SKILL.md
git commit -m "docs(okr): T1 merge state+keyword routing thành 1 bảng (Bước 3)"
```

---

## Task 2: T3 — Phân vai SOT về single source ở `CLAUDE.md`

**Files:**

- Modify: `CLAUDE.md` (thêm section `## Phân vai SOT`)
- Modify: `skills/okr/SKILL.md:26-37` (section "Phân vai SOT (quan trọng)": rút gọn + link sang CLAUDE.md)
- Modify: `skills/okr-track/references/data-format.md:58-69` (section "Structure fields": thêm note link CLAUDE.md là canonical)
- Modify: `docs/okr-system-review.md:352-362` (section 9 "Phân vai SOT (tham chiếu nhanh)": xoá bảng inline, link sang CLAUDE.md)

**Mục tiêu:** Bảng Phân vai SOT hiện đang đồng tồn tại ở 3 file skill (`okr/SKILL.md`, `okr-track/references/data-format.md`) + 1 file doc (`docs/okr-system-review.md`). Đặt canonical ở `CLAUDE.md` (mỗi project chỉ có 1 file CLAUDE.md), các nơi khác link sang. Đồng thời ở `docs/okr-system-review.md` cập nhật row "Người, tool, ngân sách, PIC, khả dụng" thành "Solo Profile (capacity, skills), tool, ngân sách" (đồng bộ Đợt 2 K6 + Đợt 3 PIC sweep).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "## Phân vai SOT|Field.*Skill được phép sửa|Field.*Skill áp dụng" CLAUDE.md skills/okr/SKILL.md skills/okr-track/references/data-format.md docs/okr-system-review.md
```

Expected: 0 match ở `CLAUDE.md` (chưa có). 1 match ở mỗi file còn lại.

- [ ] **Step 2: Thêm section Phân vai SOT vào CLAUDE.md**

Edit `CLAUDE.md`:

old_string:

```
Tất cả skill con được kích hoạt qua orchestrator `/okr`. Không trigger trực tiếp.

## Hai loại mục tiêu
```

new_string:

```
Tất cả skill con được kích hoạt qua orchestrator `/okr`. Không trigger trực tiếp.

## Phân vai SOT

Mỗi field SOT chỉ được sửa bởi đúng 1 skill (`okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để apply).

| Field | Skill được phép sửa |
|-------|---------------------|
| Objective text, KR/KI target/baseline/ngưỡng, period, status | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update` |
| KR.current, KI.current, action.status, plan counters | `okr-track` `light`/`deep` |
| Inbox items (tạo mới) | `okr-capture` |
| Inbox items (xử lý: status transition) | `okr-track` |

> Bảng này là **single source**. Các file `skills/okr/SKILL.md`, `skills/okr-track/references/data-format.md`, `docs/okr-system-review.md` link sang đây.

## Hai loại mục tiêu
```

- [ ] **Step 3: Rút gọn section Phân vai SOT trong `okr/SKILL.md`**

Edit `skills/okr/SKILL.md`:

old_string:

```
## Phân vai SOT (quan trọng)

| Field | Skill được phép sửa |
|-------|---------------------|
| Objective text, KR/KI target/baseline/ngưỡng, period, status | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update` |
| KR.current, KI.current, action.status, plan counters | `okr-track` `light`/`deep` |
| Inbox items (tạo mới) | `okr-capture` |
| Inbox items (xử lý: status transition) | `okr-track` |

`okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa. Nó delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.
```

new_string:

```
## Phân vai SOT

Bảng canonical ở `CLAUDE.md` section "Phân vai SOT". Tóm tắt: mỗi field SOT chỉ được sửa bởi 1 skill. `okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa, delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.
```

- [ ] **Step 4: Thêm note link trong `okr-track/references/data-format.md`**

Edit `skills/okr-track/references/data-format.md`:

old_string:

```
### Structure fields (track CHỈ đề xuất, delegate để apply)

| Field | Skill áp dụng |
|-------|---------------|
| `objective` text, `quarter`, `start_date`, `end_date` | `okr-init` `update-objective` |
| KR target, KR baseline, thêm/xoá KR | `okr-init` `update-objective` |
| `objective.md` frontmatter `status` (active/paused/completed/cancelled) | `okr-init` `update-objective` |
| Resource: Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Action: title, description, due_date, depends_on, deliverable | `okr-plan` `update` |
| Thêm/xoá action | `okr-plan` `update` |
| Milestone deadline, thêm/xoá milestone | `okr-plan` `update` |
```

new_string:

```
### Structure fields (track CHỈ đề xuất, delegate để apply)

> Bảng canonical ở `CLAUDE.md` section "Phân vai SOT". Bảng dưới là **chi tiết hoá theo field cụ thể** trong `objective.md` / `plan.md` / `resources.md`, dùng để track lookup khi delegate.

| Field | Skill áp dụng |
|-------|---------------|
| `objective` text, `quarter`, `start_date`, `end_date` | `okr-init` `update-objective` |
| KR target, KR baseline, thêm/xoá KR | `okr-init` `update-objective` |
| `objective.md` frontmatter `status` (active/paused/completed/cancelled) | `okr-init` `update-objective` |
| Resource: Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Action: title, description, due_date, depends_on, deliverable | `okr-plan` `update` |
| Thêm/xoá action | `okr-plan` `update` |
| Milestone deadline, thêm/xoá milestone | `okr-plan` `update` |
```

- [ ] **Step 5: Xoá bảng inline + link trong `docs/okr-system-review.md`**

Edit `docs/okr-system-review.md`:

old_string:

```
## 9. Phân vai SOT (tham chiếu nhanh)

| Field                                                             | Skill được phép sửa           |
| ----------------------------------------------------------------- | ----------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective` |
| Người, tool, ngân sách, PIC, khả dụng                             | `okr-init` `update-resource`  |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`           |
| KR.current, KI.current, action.status, plan counters              | `okr-track` `light`/`deep`    |
| Inbox items (tạo mới)                                             | `okr-capture`                 |
| Inbox items (xử lý: status transition)                            | `okr-track`                   |
```

new_string:

```
## 9. Phân vai SOT (tham chiếu nhanh)

Bảng canonical ở `CLAUDE.md` section "Phân vai SOT" (single source). Không lặp lại ở đây để tránh drift. Tham chiếu nhanh: `okr-init update-objective` sửa objective/KR/KI/period/status; `okr-init update-resource` sửa Solo Profile/tool/ngân sách; `okr-plan update` sửa milestones + action structure; `okr-track light/deep` sửa progress fields; `okr-capture` tạo inbox item; `okr-track` xử lý inbox status transition.
```

- [ ] **Step 6: Verify**

Run:

```bash
grep -nE "## Phân vai SOT|^\| Objective text" CLAUDE.md
```

Expected: Match `## Phân vai SOT` + `| Objective text...`.

Run:

```bash
grep -cE "^\| Objective text" skills/okr/SKILL.md docs/okr-system-review.md
```

Expected: 0 ở mỗi file (đã xoá bảng inline).

Run:

```bash
grep -nE "Người, tool, ngân sách, PIC, khả dụng" docs/okr-system-review.md
```

Expected: 0 match (đã xoá row PIC legacy).

- [ ] **Step 7: Commit**

```bash
git add CLAUDE.md skills/okr/SKILL.md skills/okr-track/references/data-format.md docs/okr-system-review.md
git commit -m "docs(okr): T3 Phân vai SOT single source ở CLAUDE.md, 3 nơi khác link sang"
```

---

## Task 3: T2 — Tách Quality Gate thành file shared

**Files:**

- Create: `skills/okr/references/quality-gate.md`
- Modify: `skills/okr-init/SKILL.md:17-33` (section `## Quality Gate`: rút gọn + link)
- Modify: `skills/okr-plan/SKILL.md:15-30` (section `## Quality Gate`: rút gọn + link)

**Mục tiêu:** Hiện tại Quality Gate copy-paste gần giống nhau ở okr-init (line 17-33) và okr-plan (line 15-30): cùng 3 câu kiểm tra + bảng hành vi 5 row. Khác biệt nhỏ ở example domain (init nói "tăng doanh thu", plan nói "Nghiên cứu thêm"). Tách core 3 câu + bảng hành vi sang file shared, hai skill chỉ giữ **example-specific** + link.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^## Quality Gate" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr/references/quality-gate.md
```

Expected: 1 match init, 1 match plan, 0 match quality-gate.md (chưa có).

Run:

```bash
ls skills/okr/references/ 2>&1
```

Expected: thư mục có thể chưa tồn tại hoặc rỗng (Đợt 1-3 chưa tạo).

- [ ] **Step 2: Tạo file shared `skills/okr/references/quality-gate.md`**

Write file `skills/okr/references/quality-gate.md`:

```markdown
# Quality Gate (shared cho okr-init + okr-plan)

Mỗi khi user trả lời 1 câu hỏi từ skill (init: tạo objective/KR/resource; plan: chọn initiative/action/effort), agent tự kiểm tra 3 câu **không hiển thị cho user**:

1. **Đủ cụ thể?** Câu trả lời có thể chuyển thành KR/KI đo được (init) hoặc action có deliverable rõ (plan) không?
2. **Giả định ẩn?** User có bỏ qua constraint quan trọng nào (capacity, deadline, dependency, skill)?
3. **Mâu thuẫn?** Câu trả lời có xung đột với context trước (capacity, timeline, objective, KR đã chốt)?

## Hành vi theo kết quả

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp câu hỏi kế |
| Bất kỳ fail | **TRƯỚC** follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Sau đó dùng Deepening Techniques phù hợp (mỗi skill có bảng kỹ thuật riêng). |
| User trả lời "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm BẮT BUỘC nhắc lại các field TBD trước khi ghi. |
| User tỏ ra sốt ruột | Giảm độ sâu, chỉ giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn. |
| User paste từ doc có sẵn (brief, PRD, spec) | Tóm tắt lại bằng lời mình, hỏi "tôi hiểu đúng chưa?" thay vì nhận nguyên xi. |

## Nguyên tắc

- Quality Gate là **internal check**, không hiển thị 3 câu cho user.
- Dòng `(Mình đào sâu thêm vì <lý do>)` chỉ in khi có fail, đặt **trước** câu hỏi follow-up trên cùng turn.
- Lý do trong ngoặc phải **cụ thể**, không chung chung. Vd:
  - ✅ `(Mình đào sâu thêm vì "tăng doanh thu" chưa nói kênh nào, sản phẩm nào.)`
  - ❌ `(Mình cần thêm thông tin)` (quá chung)
- Mỗi skill có thể bổ sung thêm rule riêng (vd plan check capacity vs effort), nhưng vẫn dựa trên 3 câu core ở đây.
```

- [ ] **Step 3: Rút gọn section Quality Gate trong `okr-init/SKILL.md`**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
## Quality Gate: đánh giá câu trả lời trước khi đi tiếp

Mỗi khi user trả lời, agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành KR/KI đo được hoặc action có deliverable không? Vd: "tăng doanh thu" → FAIL (chưa nói kênh nào, sản phẩm nào, bao nhiêu %).
2. **Giả định ẩn?** User bỏ qua điều kiện quan trọng nào không? Vd: "launch Q4" nhưng chưa nói team có ai.
3. **Mâu thuẫn?** Câu trả lời có xung đột với context trước không? Vd: muốn 12 bài/tháng nhưng chỉ có 1 writer part-time.

**Hành vi theo kết quả:**

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp câu hỏi kế |
| Bất kỳ fail | Trước follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Vd: `(Mình đào sâu thêm vì "tăng doanh thu" chưa nói kênh nào, sản phẩm nào.)`. Sau đó dùng kỹ thuật phù hợp (xem Deepening Techniques). |
| User trả lời "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm PHẢI nhắc lại các field TBD |
| User tỏ ra sốt ruột | Giảm độ sâu, chỉ giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn |
| User paste từ doc có sẵn (brief, PRD) | Tóm tắt lại bằng lời mình, hỏi "tôi hiểu đúng chưa?" thay vì nhận nguyên xi |
```

new_string:

```
## Quality Gate

Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.

Ví dụ áp dụng cho mode `new` / `update-objective`:
- "Đủ cụ thể?" → "tăng doanh thu" FAIL vì chưa nói kênh nào, sản phẩm nào, bao nhiêu %.
- "Giả định ẩn?" → "launch Q4" nhưng chưa nói capacity có đủ không.
- "Mâu thuẫn?" → muốn 12 bài/tháng nhưng capacity Solo Profile chỉ 8h/tuần.

Mode `update-resource` áp dụng câu 1 chính (tài nguyên có cụ thể đo được không: số giờ/tuần, skill rõ ràng, tool đã tồn tại).
```

- [ ] **Step 4: Rút gọn section Quality Gate trong `okr-plan/SKILL.md`**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
## Quality Gate: đánh giá câu trả lời trước khi đi tiếp

Mỗi khi user trả lời (chọn initiative, confirm action, ước effort...), agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành action có deliverable đo được không? Vd: "Nghiên cứu thêm" → FAIL (output là gì? đo bằng gì?).
2. **Giả định ẩn?** User bỏ qua constraint quan trọng không? Vd: thêm 5 actions mới nhưng không nói effort tổng bao nhiêu giờ.
3. **Mâu thuẫn?** Có xung đột với capacity/timeline đã có không? Vd: capacity 10h/tuần nhưng gán 6 actions effort `m` (1-2 ngày mỗi cái) cùng deadline tuần sau.

**Hành vi theo kết quả:**

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp |
| Bất kỳ fail | Trước follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Vd: `(Mình đào sâu thêm vì action "Nghiên cứu thêm" chưa nói output là gì.)`. Sau đó dùng kỹ thuật phù hợp (xem Deepening Techniques). |
| User nói "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm PHẢI nhắc lại |
| User sốt ruột | Giảm độ sâu, giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn |
```

new_string:

```
## Quality Gate

Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.

Ví dụ áp dụng cho mode `new` / `update`:
- "Đủ cụ thể?" → action "Nghiên cứu thêm" FAIL (output là gì? đo bằng gì?).
- "Giả định ẩn?" → thêm 5 actions mới nhưng không nói effort tổng bao nhiêu giờ.
- "Mâu thuẫn?" → capacity 10h/tuần nhưng gán 6 actions effort `m` (1-2 ngày mỗi cái) cùng deadline tuần sau.
```

- [ ] **Step 5: Verify**

Run:

```bash
grep -nE "^## Quality Gate|skills/okr/references/quality-gate.md" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr/references/quality-gate.md
```

Expected: Match `## Quality Gate` ở cả init/plan + reference path `skills/okr/references/quality-gate.md` ở cả 2.

Run:

```bash
wc -l skills/okr/references/quality-gate.md
```

Expected: ~20-30 dòng.

Run:

```bash
grep -cE "Đủ cụ thể\?" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr/references/quality-gate.md
```

Expected: init=1 (trong example), plan=1 (trong example), quality-gate.md=1 (canonical). Tổng 3 (giảm so với trước: init 1 + plan 1 trong full sections).

- [ ] **Step 6: Commit**

```bash
git add skills/okr/references/quality-gate.md skills/okr-init/SKILL.md skills/okr-plan/SKILL.md
git commit -m "docs(okr): T2 Quality Gate shared file, okr-init + okr-plan link sang"
```

---

## Task 4: Minor1 — Rename heading `### Phase 0` trùng trong `okr-init/SKILL.md`

**Files:**

- Modify: `skills/okr-init/SKILL.md:51` (thay `### Phase 0: Đọc inbox làm context (nếu có)` → `### Phase 0a: Đọc inbox làm context (nếu có)`)

**Mục tiêu:** Final review Đợt 3 phát hiện 2 heading `### Phase 0`: line 37 ("Detect mode" — global), line 51 ("Đọc inbox làm context" — bên trong Mode NEW). Tên trùng gây nhiễu khi LLM scan TOC. Rename heading thứ 2 thành `### Phase 0a`. Phase sau đó là Phase 1 (chọn loại mục tiêu) — không cần đổi tên (không tạo step 0b).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^### Phase 0" skills/okr-init/SKILL.md
```

Expected: 2 match. Line ~37 "Detect mode" + line ~51 "Đọc inbox làm context".

- [ ] **Step 2: Rename heading thứ 2**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
### Phase 0: Đọc inbox làm context (nếu có)
```

new_string:

```
### Phase 0a: Đọc inbox làm context (nếu có)
```

- [ ] **Step 3: Cập nhật cross-reference nếu có**

Run:

```bash
grep -nE "Phase 0.*inbox|inbox.*Phase 0" skills/okr-init/SKILL.md
```

Expected: Match có thể có ở Phase 8 ("post-init suggest map") hoặc bảng cuối. Nếu có "Phase 0" ám chỉ inbox context, sửa thành "Phase 0a".

Nếu match → edit từng cái, đổi "Phase 0" → "Phase 0a" khi context là "Đọc inbox làm context".

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "^### Phase 0" skills/okr-init/SKILL.md
```

Expected: 2 match. Line ~37 "### Phase 0: Detect mode" + line ~51 "### Phase 0a: Đọc inbox làm context".

Run:

```bash
grep -cE "^### Phase 0:" skills/okr-init/SKILL.md
```

Expected: 1 (chỉ heading "Detect mode" giữ `Phase 0:`, heading "đọc inbox" đã đổi thành `Phase 0a:`).

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "docs(okr-init): rename heading Phase 0 trùng -> Phase 0a (Minor 1 từ final review Đợt 3)"
```

---

## Task 5: M7 + T4c — Plan mode UPDATE bypass menu khi từ track inbox + pre_confirmed handler

**Files:**

- Modify: `skills/okr-plan/SKILL.md:121-150` (Mode UPDATE: Phase 1 thêm bypass note + Phase 2 menu)
- Modify: `skills/okr-plan/SKILL.md:158-189` (Phase 4 CONFIRM diff: thêm pre_confirmed handler)

**Mục tiêu:**

- **M7**: Khi track inbox processing (Phase 5 Bước 4) delegate sang plan với danh sách items đã pre-processed (title, description, related_kr/action đã validate), plan bypass menu hỏi "thêm/sửa/xoá/dời" mà đi thẳng vào Phase 4 confirm với data có sẵn.
- **T4c**: Khi payload có `context.pre_confirmed: true`, plan SKIP block "Tác động" + confirm "y/sửa/huỷ" → ghi luôn ở Phase 5. Vẫn HIỂN THỊ block "Lý do điều chỉnh" + bảng diff cho user xem trace (chỉ skip ask confirm).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^### Phase 1: Hiển thị state|^### Phase 2: Hỏi user|^### Phase 4: CONFIRM diff" skills/okr-plan/SKILL.md
```

Expected: Match 3 heading trong Mode UPDATE (line ~125, 143, 158).

- [ ] **Step 2: Thêm bypass note vào Phase 1 + Phase 2 Mode UPDATE**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 1: Hiển thị state hiện tại

Đọc `plan.md` + frontmatter `actions/*.md` (**không đệ quy**, bỏ qua `actions/archive/`) + `objective.md` + `resources.md`. Hiển thị:
```

new_string:

```
### Phase 1: Hiển thị state hiện tại

**Bypass detect (M7):** Nếu mode `update` được kích hoạt qua delegate từ `okr-track` Phase 5 (inbox processing) **kèm `context.items[]` đã pre-processed** (mỗi item có `title`, `description`, `related_kr`/`related_action` đã validate, `effort` ước, `milestone` gợi ý), SKIP Phase 1 state display + Phase 2 menu, đi thẳng Phase 3 (gom item thành changes) → Phase 4 confirm với data sẵn. State display chỉ chạy khi user vào trực tiếp `/okr plan update`.

Mode bình thường (state display): đọc `plan.md` + frontmatter `actions/*.md` (**không đệ quy**, bỏ qua `actions/archive/`) + `objective.md` + `resources.md`. Hiển thị:
```

- [ ] **Step 3: Thêm bypass note vào Phase 2 menu**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 2: Hỏi user/nhận đề xuất từ track

Nếu vào từ `okr-track`: hiển thị đề xuất track đã đưa, user chọn áp dụng cái nào.

Menu nếu user vào trực tiếp:
1. Thêm action mới
2. Sửa action (title, due_date, status, deps, deliverable, effort, priority)
3. Xoá action
4. Dời deadline milestone
5. Thêm/xoá milestone
```

new_string:

```
### Phase 2: Hỏi user/nhận đề xuất từ track

3 nguồn trigger:

1. **Từ track deep delegate** (`context.changes` + `context.reason` ở payload): hiển thị đề xuất track đã đưa, user chọn áp dụng cái nào (nếu `pre_confirmed: false` hoặc không có), hoặc đi thẳng Phase 4 confirm (nếu `pre_confirmed: true`).
2. **Từ track inbox delegate** (`context.items[]` pre-processed): gom items thành Phase 3 changes, đi thẳng Phase 4 confirm. Mỗi `action` item → 1 "Thêm action mới" với data có sẵn (title, description, related_kr, effort gợi ý, milestone gợi ý). Mỗi `blocker` đụng action → 1 "Sửa action" status=blocked + lý do. Mỗi `resource` → delegate tiếp sang `okr-init update-resource` (không xử lý ở đây).
3. **User vào trực tiếp** (`/okr plan update`): hiển thị menu:
   1. Thêm action mới
   2. Sửa action (title, due_date, status, deps, deliverable, effort, priority)
   3. Xoá action
   4. Dời deadline milestone
   5. Thêm/xoá milestone
```

- [ ] **Step 4: Thêm pre_confirmed handler vào Phase 4 CONFIRM**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 4: CONFIRM diff (BẮT BUỘC)

Nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:
```

new_string:

```
### Phase 4: CONFIRM diff

**Pre-confirmed flow (T4c):** Nếu payload có `context.pre_confirmed: true` (user đã confirm tại track Bước 4 với full all-changes preview), SKIP ask "Xác nhận? (y/sửa/huỷ)" và đi thẳng Phase 5 ghi file. Vẫn HIỂN THỊ block "Lý do điều chỉnh" + bảng diff để user trace, kèm 1 dòng cuối: `(Đã được confirm tại track Bước 4. Ghi file ngay.)`.

Default flow (không pre_confirmed): nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:
```

- [ ] **Step 5: Cập nhật quy tắc reason display**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
Quy tắc reason display:
- Nếu `context.reason` rỗng hoặc không có (user vào trực tiếp `/okr plan update`) → KHÔNG render block "Lý do điều chỉnh".
- `source_review` luôn đi kèm reason (cùng block).
- Reason là plain text 1-3 câu, KHÔNG markdown đặc biệt (giữ readable trong terminal).
```

new_string:

```
Quy tắc reason display:
- Nếu `context.reason` rỗng hoặc không có (user vào trực tiếp `/okr plan update`) → KHÔNG render block "Lý do điều chỉnh".
- `source_review` luôn đi kèm reason (cùng block).
- Reason là plain text 1-3 câu, KHÔNG markdown đặc biệt (giữ readable trong terminal).

Quy tắc pre_confirmed:
- `pre_confirmed: true` chỉ áp dụng khi track Bước 4 đã hiển thị **all-changes diff** + user reply "y" (xem `okr-track/SKILL.md` Phase 4b Bước 4-5).
- Nếu thiếu 1 trong 2 (track Bước 4 không show full diff, hoặc payload không set `pre_confirmed`), plan vẫn chạy default flow (hỏi confirm).
- Pre-confirmed bypass CHỈ skip ask "y/sửa/huỷ". Vẫn ghi log + báo cáo bình thường.
```

- [ ] **Step 6: Verify**

Run:

```bash
grep -nE "Bypass detect \(M7\)|pre_confirmed|Pre-confirmed flow" skills/okr-plan/SKILL.md
```

Expected: ≥3 match (Phase 1 bypass + Phase 4 handler + quy tắc).

Run:

```bash
grep -nE "context\.items\[\]" skills/okr-plan/SKILL.md
```

Expected: ≥2 match (Phase 1 mention + Phase 2 detail).

- [ ] **Step 7: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "docs(okr-plan): M7 bypass menu khi từ track inbox + T4c pre_confirmed handler Phase 4"
```

---

## Task 6: T4b — Init mode UPDATE-OBJECTIVE pre_confirmed handler Phase 6

**Files:**

- Modify: `skills/okr-init/SKILL.md` (Phase 6 CONFIRM update-objective: thêm pre_confirmed handler)

**Mục tiêu:** Tương tự T4c nhưng cho init. Khi track delegate update-objective với `pre_confirmed: true`, init Phase 6 hiển thị reason + diff nhưng skip ask "y/sửa/huỷ" → đi thẳng Phase 7 ghi file.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^### Phase 6:|^### Phase 7:|Lý do điều chỉnh" skills/okr-init/SKILL.md
```

Expected: Match Phase 6 CONFIRM (line ~?) + Phase 7 Ghi file + "Lý do điều chỉnh" trong Phase 6.

- [ ] **Step 2: Đọc xung quanh Phase 6 để xác định old_string chính xác**

Run:

```bash
awk '/^### Phase 6:/,/^### Phase 7:/' skills/okr-init/SKILL.md | head -60
```

Expected: thấy block "Lý do điều chỉnh" + bảng diff + dòng "Xác nhận?".

- [ ] **Step 3: Thêm pre_confirmed handler vào Phase 6**

Edit `skills/okr-init/SKILL.md`: Sửa heading Phase 6 + chèn block "Pre-confirmed flow" ngay sau heading, TRƯỚC block "Lý do điều chỉnh" hiện tại.

Pattern edit (dùng old_string = heading + dòng đầu Phase 6 hiện tại; new_string = heading + block pre_confirmed + dòng đầu Phase 6 gốc):

Tuỳ vào nội dung thực tế ở Phase 6, có thể có 2 variant:

**Variant A** (nếu Phase 6 mở đầu bằng "Nếu vào từ track delegate"):

old_string:

```
### Phase 6: CONFIRM (BẮT BUỘC)

Nếu vào từ track delegate (có `context.reason`)
```

new_string:

```
### Phase 6: CONFIRM

**Pre-confirmed flow (T4b):** Nếu payload có `context.pre_confirmed: true` (user đã confirm tại track Bước 4 với full all-changes preview), SKIP ask "Xác nhận? (y/sửa/huỷ)" và đi thẳng Phase 7 ghi file. Vẫn HIỂN THỊ block "Lý do điều chỉnh" + bảng diff + Impact Check để user trace, kèm 1 dòng cuối: `(Đã được confirm tại track Bước 4. Ghi file ngay.)`.

Default flow (không pre_confirmed): nếu vào từ track delegate (có `context.reason`)
```

**Variant B** (nếu Phase 6 mở đầu khác): điều chỉnh old_string theo nội dung thực tế tìm thấy ở Step 2.

- [ ] **Step 4: Thêm quy tắc pre_confirmed ở cuối Phase 6**

Tìm block "Quy tắc reason display" trong Phase 6 (nếu có), thêm block "Quy tắc pre_confirmed" sau (giống Task 5 Step 5).

Nếu không có block quy tắc → thêm 1 block mới ngay trước Phase 7:

old_string (xác định bằng Step 2): dòng cuối Phase 6 trước khi `### Phase 7:`.

new_string: dòng cũ + block:

```

Quy tắc pre_confirmed:
- `pre_confirmed: true` chỉ áp dụng khi track Bước 4 đã hiển thị **all-changes diff** + user reply "y" (xem `okr-track/SKILL.md` Phase 4b Bước 4-5).
- Pre-confirmed bypass CHỈ skip ask "y/sửa/huỷ". Vẫn hiển thị Impact Check (Phase 5) + ghi log + báo cáo bình thường.
```

- [ ] **Step 5: Verify**

Run:

```bash
grep -nE "Pre-confirmed flow|pre_confirmed: true" skills/okr-init/SKILL.md
```

Expected: ≥2 match (handler + quy tắc).

Run:

```bash
awk '/^### Phase 6:/,/^### Phase 7:/' skills/okr-init/SKILL.md | grep -cE "pre_confirmed"
```

Expected: ≥2.

- [ ] **Step 6: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "docs(okr-init): T4b pre_confirmed handler Phase 6 update-objective"
```

---

## Task 7: T4a — Track Phase 4b Bước 4-5 gom all-changes confirm

**Files:**

- Modify: `skills/okr-track/SKILL.md` (Phase 4b Bước 4: explicit all-changes diff confirm trước delegate)

**Mục tiêu:** Hiện tại Bước 4 chỉ hỏi "Đồng ý đề xuất nào? (vd: 1,3,4 / không / sửa N: <new value>)". User chọn rồi Bước 5 delegate với `pre_confirmed: true`. Cần explicit: TRƯỚC khi delegate, render bảng **all-changes diff** gom theo skill đích, hỏi 1 lần confirm cho tất cả. Sau confirm → set `pre_confirmed: true` cho mỗi payload. Refactor để wording rõ ràng "đây là all-changes confirm, init/plan sẽ skip phase confirm riêng".

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^#### Bước 4:|^#### Bước 5:" skills/okr-track/SKILL.md
```

Expected: Match `#### Bước 4: User chọn áp dụng cái nào` + `#### Bước 5: Delegate sang skill phù hợp`.

- [ ] **Step 2: Refactor Bước 4 thành "All-changes diff confirm"**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
#### Bước 4: User chọn áp dụng cái nào

```
Đồng ý đề xuất nào? (vd: 1,3,4 / không / sửa N: <new value>)
```
```

new_string:

```
#### Bước 4: All-changes confirm + chọn áp dụng

Bước 1: User chọn cái nào áp dụng:

```
Đồng ý đề xuất nào? (vd: 1,3,4 / không / sửa N: <new value>)
```

Bước 2: Sau khi user chọn (vd `1,3,4`), gom thay đổi theo skill đích + render **all-changes diff** trước khi delegate. Bảng phải gom theo skill (nhóm các change cùng skill vào 1 block), để user thấy tổng quan + biết init/plan sẽ áp dụng gì:

```
All-changes preview (gom theo skill đích)

→ okr-init update-objective
  - KR2.target: 50 > 35  (lý do: market shift Q4)

→ okr-plan update
  - Thêm A013 "Tăng marketing"  (gắn KR2, effort m)
  - M2.deadline: 2026-11-15 > 2026-11-30  (A005 chậm)

→ okr-init update-resource
  - tool: thêm Mailchimp (cho A013 email campaign)

Confirm tất cả? (y / sửa N / huỷ N / huỷ tất)
```

User trả lời:
- `y` → đi tiếp Bước 5 với `pre_confirmed: true` cho mỗi delegate payload. Init/plan nhận signal sẽ SKIP phase confirm riêng (xem `okr-init/SKILL.md` Phase 6 + `okr-plan/SKILL.md` Phase 4).
- `sửa N` → quay lại Bước 3 sửa đề xuất số N, xong render lại all-changes preview.
- `huỷ N` → loại N khỏi danh sách áp dụng, render lại preview.
- `huỷ tất` → quay lại Bước 4 lựa chọn ban đầu.
```

- [ ] **Step 3: Cập nhật Bước 5 wording để chỉ rõ pre_confirmed luôn true sau Bước 4 confirm**

Edit `skills/okr-track/SKILL.md`:

Tìm trong Bước 5 dòng giải thích `pre_confirmed` (đã có sau Đợt 3 G9):

old_string:

```
| `context.pre_confirmed` | optional | `true` nếu user đã confirm tại Bước 4 ở track. Skill nhận có thể skip phase confirm riêng (xem T4 ở Đợt 4). Mặc định `false`. |
```

new_string:

```
| `context.pre_confirmed` | có | `true` sau khi user reply `y` cho all-changes preview ở Bước 4 (track đã gom + show full diff trước). Skill nhận BẮT BUỘC honor: skip ask "y/sửa/huỷ" ở phase confirm riêng, đi thẳng ghi file. Vẫn hiển thị diff + reason để trace. |
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "All-changes preview|pre_confirmed: true" skills/okr-track/SKILL.md
```

Expected: ≥2 match.

Run:

```bash
awk '/^#### Bước 4:/,/^#### Bước 5:/' skills/okr-track/SKILL.md | grep -cE "All-changes preview|Confirm tất cả"
```

Expected: ≥2.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "docs(okr-track): T4a gom all-changes confirm Bước 4 + pre_confirmed BẮT BUỘC"
```

---

## Task 8: T5 + M5a — Capture: merge Phase 5 vào Phase 4 + clarify không validate

**Files:**

- Modify: `skills/okr-capture/SKILL.md:79-87` (Phase 4 + Phase 5: merge)
- Modify: `skills/okr-capture/SKILL.md:14-19` (Nguyên tắc + Quy tắc: clarify "không validate related, track làm")

**Mục tiêu:**

- **T5**: Phase 5 hiện chỉ là 1 dòng reminder "Nếu inbox ≥5 → nhắc xử lý". Quá ngắn để là phase riêng. Merge vào Phase 4 (sau ghi file).
- **M5a**: Hiện tại Phase 2 nói "gợi ý related_kr/related_action" nhưng không nói rõ là gợi ý best-effort, không validate. Clarify Nguyên tắc + Quy tắc: capture chỉ ghi nội dung user nhập + gợi ý related (best-effort), validate là việc của track (Phase 5 Bước 3).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^### Phase 4:|^### Phase 5:|## Nguyên tắc|## Quy tắc" skills/okr-capture/SKILL.md
```

Expected: Match Phase 4 (line ~79), Phase 5 (line ~85), Nguyên tắc (line 14), Quy tắc (line ~93).

- [ ] **Step 2: Merge Phase 5 vào Phase 4**

Edit `skills/okr-capture/SKILL.md`:

old_string:

```
### Phase 4: Ghi file

1. Tạo `.okr/inbox/` nếu chưa có.
2. Ghi mỗi item thành 1 file: `YYYY-MM-DD-HHmm-slug.md`
3. Hiển thị: "Đã lưu N items vào inbox. Chạy `/okr track` để xử lý inbox."

### Phase 5: Gợi ý (tuỳ chọn)

Nếu inbox đã có ≥5 items chưa xử lý → nhắc: "Inbox có N items. Nên xử lý sớm bằng `/okr track`."
```

new_string:

```
### Phase 4: Ghi file + nhắc xử lý

1. Tạo `.okr/inbox/` nếu chưa có.
2. Ghi mỗi item thành 1 file: `YYYY-MM-DD-HHmm-slug.md`
3. Hiển thị: "Đã lưu N items vào inbox. Chạy `/okr track` để xử lý inbox."
4. **Reminder (nếu áp dụng):** đếm tổng items pending trong `.okr/inbox/` (kể cả vừa thêm). Nếu ≥5 → in thêm 1 dòng: `⚠️ Inbox có N items pending. Xử lý sớm bằng /okr track để tránh tích luỹ.`
```

- [ ] **Step 3: Clarify Nguyên tắc**

Edit `skills/okr-capture/SKILL.md`:

old_string:

```
## Nguyên tắc

- Nhanh: tối thiểu bước xác nhận. Mục tiêu là ghi xong trong 1-2 turn.
- Không sửa SOT: chỉ tạo file trong `inbox/`. Mọi thay đổi SOT do skill khác xử lý khi inbox được process.
- Giữ nguyên văn gốc: dù agent tinh chỉnh title/description, phần context gốc (nguyên văn user nhập) luôn được giữ lại.
- Cho phép capture khi chưa có `.okr/objective.md`: tạo `.okr/inbox/` ngay, cảnh báo user nên chạy `/okr init` khi sẵn sàng.
```

new_string:

```
## Nguyên tắc

- Nhanh: tối thiểu bước xác nhận. Mục tiêu là ghi xong trong 1-2 turn.
- Không sửa SOT: chỉ tạo file trong `inbox/`. Mọi thay đổi SOT do skill khác xử lý khi inbox được process.
- Giữ nguyên văn gốc: dù agent tinh chỉnh title/description, phần context gốc (nguyên văn user nhập) luôn được giữ lại.
- **Không validate `related_kr`/`related_action` ID (M5a):** capture chỉ **gợi ý** best-effort dựa trên context user nhập + đọc nhanh `.okr/objective.md`/`actions/`. Nếu gợi ý sai (ID không tồn tại, KR đã missed, action đã archive), không phải lỗi capture. Track Phase 5 Bước 3 sẽ validate lại từng ID lúc xử lý inbox + đề xuất sửa hoặc set `null`.
- Cho phép capture khi chưa có `.okr/objective.md`: tạo `.okr/inbox/` ngay, cảnh báo user nên chạy `/okr init` khi sẵn sàng.
```

- [ ] **Step 4: Clarify Quy tắc**

Edit `skills/okr-capture/SKILL.md`:

old_string:

```
## Quy tắc

- KHÔNG sửa SOT (objective.md, plan.md, resources.md, actions/).
- KHÔNG tự phân loại nếu mơ hồ. Hỏi user.
- Giữ nguyên văn gốc trong body (section "## Context gốc").
- Batch: tách rõ từng item, confirm tất cả 1 lần.
- File name: `YYYY-MM-DD-HHmm-slug.md`. Slug dùng tiếng Việt không dấu hoặc tiếng Anh, dấu gạch ngang.
- Nếu `.okr/` chưa có → vẫn tạo `.okr/inbox/` + cảnh báo chạy init.
- KHÔNG ghi `staleness_days` vào frontmatter. Capture chỉ lưu `captured_at`. Staleness được compute on-the-fly bởi `okr-track` Phase 5 (xem `references/data-format.md` section "Inbox Aging").
```

new_string:

```
## Quy tắc

- KHÔNG sửa SOT (objective.md, plan.md, resources.md, actions/).
- KHÔNG tự phân loại nếu mơ hồ. Hỏi user.
- KHÔNG validate `related_kr`/`related_action` ID. Gợi ý best-effort là đủ. Track xử lý inbox sẽ validate.
- Giữ nguyên văn gốc trong body (section "## Context gốc").
- Batch: tách rõ từng item, confirm tất cả 1 lần.
- File name: `YYYY-MM-DD-HHmm-slug.md`. Slug dùng tiếng Việt không dấu hoặc tiếng Anh, dấu gạch ngang.
- Nếu `.okr/` chưa có → vẫn tạo `.okr/inbox/` + cảnh báo chạy init.
- KHÔNG ghi `staleness_days` vào frontmatter. Capture chỉ lưu `captured_at`. Staleness được compute on-the-fly bởi `okr-track` Phase 5 (xem `references/data-format.md` section "Inbox Aging").
```

- [ ] **Step 5: Verify**

Run:

```bash
grep -nE "^### Phase 5:" skills/okr-capture/SKILL.md
```

Expected: 0 match (đã merge).

Run:

```bash
grep -nE "Không validate `related_kr`" skills/okr-capture/SKILL.md
```

Expected: 2 match (Nguyên tắc + Quy tắc).

Run:

```bash
grep -nE "Reminder.*inbox|Inbox có N items pending" skills/okr-capture/SKILL.md
```

Expected: ≥1 match (Phase 4 step 4).

- [ ] **Step 6: Commit**

```bash
git add skills/okr-capture/SKILL.md
git commit -m "docs(okr-capture): T5 merge Phase 5 vào Phase 4 + M5a clarify không validate related"
```

---

## Task 9: M5b — Track Phase 5 Bước 3 thêm validate related_kr/action ID

**Files:**

- Modify: `skills/okr-track/SKILL.md` (Phase 5 Bước 3: thêm step validate trước xử lý)

**Mục tiêu:** Khi track xử lý inbox `action`/`blocker`/`resource` ở Phase 5 Bước 3, validate `related_kr`/`related_action`/`related_tool` ID có thực tế trong SOT không. Nếu sai → đề xuất ID đúng (nếu detect được match gần) hoặc set `null` rồi hỏi user.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "^#### Bước 3: Xử lý từng item" skills/okr-track/SKILL.md
```

Expected: Match (Phase 5 Bước 3).

- [ ] **Step 2: Đọc xung quanh Bước 3**

Run:

```bash
awk '/^#### Bước 3: Xử lý từng item/,/^#### Bước 4:/' skills/okr-track/SKILL.md
```

Expected: thấy bảng "Inbox type | Xử lý | Ai thực hiện".

- [ ] **Step 3: Thêm step validate trước bảng**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
#### Bước 3: Xử lý từng item user chọn

| Inbox type | Xử lý                                                                  | Ai thực hiện                                 |
```

new_string:

```
#### Bước 3: Xử lý từng item user chọn

**Step 3.0: Validate related ID (M5b).** Trước khi xử lý từng item user chọn, agent validate field `related_kr`/`related_action`/`related_tool` (do capture ghi vào, không guarantee đúng):

| Field | Quy tắc validate | Hành vi nếu sai |
|-------|------------------|-----------------|
| `related_kr` | Phải khớp KR ID trong `objective.md` frontmatter (vd `KR1`, `KR2`). | Tìm KR có title match fuzzy với context item → đề xuất ID đúng. Nếu không tìm được → set `null`, hỏi user. |
| `related_action` | Phải khớp action ID trong `actions/*.md` (vd `A003`). KHÔNG khớp `actions/archive/*.md` (action đã done). | Action đã archive → cảnh báo "Action X đã done/archive. Bỏ link?". Không tồn tại → set `null`, hỏi user. |
| `related_tool` | Phải khớp tool ID hoặc tên trong `resources.md` Công cụ section. | Không tồn tại → đề xuất thêm tool qua `okr-init update-resource` hoặc set `null`. |

Hiển thị cho user khi có sai sót:

```
Validate inbox related fields
  Item #1 (action "Viết unit test cho API auth"): related_kr=KR9 → KR9 không tồn tại.
    Đề xuất: KR2 (code coverage) hoặc null.
  Item #3 (blocker "Server staging down"): related_action=A100 → A100 không tồn tại.
    Đề xuất: A007 (deploy staging) match fuzzy 78%, hoặc null.

Áp dụng đề xuất? (y / sửa N / null tất / huỷ)
```

User trả lời:
- `y` → áp dụng tất cả đề xuất, ghi đè frontmatter inbox file (chỉ field `related_*`, không đổi `status`).
- `sửa N` → user nói ID đúng cho item N.
- `null tất` → set tất cả related sai thành `null`.
- `huỷ` → giữ nguyên related sai, đi tiếp Bước 3 xử lý (sẽ skip mapping liên quan).

Sau validate, đi tiếp xử lý từng item theo bảng:

| Inbox type | Xử lý                                                                  | Ai thực hiện                                 |
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "Step 3.0: Validate related ID|Validate inbox related fields" skills/okr-track/SKILL.md
```

Expected: ≥2 match.

Run:

```bash
awk '/^#### Bước 3: Xử lý từng item/,/^#### Bước 4:/' skills/okr-track/SKILL.md | grep -cE "validate|Validate"
```

Expected: ≥3.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "docs(okr-track): M5b validate related_kr/action/tool ID ở Phase 5 Bước 3"
```

---

## Task 10: Minor2 — Track Phase 4a step 4 fallback note khi không có `## Practices`

**Files:**

- Modify: `skills/okr-track/SKILL.md` (Phase 4a Ongoing step 4: thêm fallback note)

**Mục tiêu:** Final review Đợt 3 phát hiện step 4 (Update practice streak) giả định `plan.md` luôn có section `## Practices`. Nhưng Ongoing objective có thể chưa setup practices (mới tạo, chưa qua `/okr plan`). Thêm fallback: đọc `plan.md` body, nếu không tìm thấy `## Practices` → skip step 4 với note ngắn cho user (không lỗi).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "Update practice streak" skills/okr-track/SKILL.md
```

Expected: 1 match ở step 4 Phase 4a Ongoing (line ~181).

- [ ] **Step 2: Đọc xung quanh step 4**

Run:

```bash
awk '/4\. \*\*Update practice streak\*\*/,/^5\./' skills/okr-track/SKILL.md | head -20
```

Expected: thấy nội dung step 4 hiện tại.

- [ ] **Step 3: Thêm fallback note**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
4. **Update practice streak**: đọc `plan.md` body section `## Practices` (nếu có). Với mỗi practice (`### PN: <Tên>` kèm field `frequency`, `target_count`, `current_streak`, `ki_link`):
   - Tính chu kỳ vừa qua dựa trên `frequency` (vd `weekly` = tuần vừa rồi, `daily` = hôm qua). Hỏi: "Practice **P1: Tập thể dục** (target ≥3 lần/tuần, streak hiện tại: 2). Tuần vừa rồi đạt target chưa? (y = đạt → streak +1 / n = không đạt → reset streak về 0 / skip = bỏ qua không update)".
   - User trả lời `y` → `current_streak += 1`. `n` → `current_streak = 0`. `skip` → giữ nguyên.
   - Nếu practice `ki_link` đã có KI tương ứng vừa update ở step 2-3, hiển thị inline để user thấy liên kết: `(P1 → KI1 healthy)`.
```

new_string:

```
4. **Update practice streak**: đọc `plan.md` body section `## Practices`.
   - **Fallback (Minor2):** nếu `plan.md` không tồn tại HOẶC body không có heading `## Practices` HOẶC section rỗng (không có `### PN:`), in 1 dòng `(Ongoing chưa có practices. Skip step 4. Tạo practices qua /okr plan để track streak.)` rồi đi tiếp step 5. KHÔNG báo lỗi.
   - Với mỗi practice (`### PN: <Tên>` kèm field `frequency`, `target_count`, `current_streak`, `ki_link`):
     - Tính chu kỳ vừa qua dựa trên `frequency` (vd `weekly` = tuần vừa rồi, `daily` = hôm qua). Hỏi: "Practice **P1: Tập thể dục** (target ≥3 lần/tuần, streak hiện tại: 2). Tuần vừa rồi đạt target chưa? (y = đạt → streak +1 / n = không đạt → reset streak về 0 / skip = bỏ qua không update)".
     - User trả lời `y` → `current_streak += 1`. `n` → `current_streak = 0`. `skip` → giữ nguyên.
     - Nếu practice `ki_link` đã có KI tương ứng vừa update ở step 2-3, hiển thị inline để user thấy liên kết: `(P1 → KI1 healthy)`.
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "Fallback \(Minor2\)|Ongoing chưa có practices" skills/okr-track/SKILL.md
```

Expected: ≥2 match.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "docs(okr-track): Minor2 fallback note khi Ongoing chưa có ## Practices"
```

---

## Task 11: Final cross-check Đợt 4

**Mục tiêu:** Verify toàn bộ thay đổi Đợt 4 đã đúng, không drift, không legacy còn sót.

- [ ] **Step 1: Verify routing bảng merged trong okr/SKILL.md**

Run:

```bash
grep -cE "^Keyword routing khi user cung cấp context:" skills/okr/SKILL.md
```

Expected: 0 (đã merge).

Run:

```bash
awk '/^### Bước 3:/,/^### Bước 4:/' skills/okr/SKILL.md | grep -cE "^\| (Trigger|State|Keyword)"
```

Expected: ≥1 (chỉ còn 1 header `| Trigger`).

- [ ] **Step 2: Verify Phân vai SOT single source ở CLAUDE.md**

Run:

```bash
grep -cE "^## Phân vai SOT" CLAUDE.md skills/okr/SKILL.md docs/okr-system-review.md
```

Expected: 1 ở mỗi file (CLAUDE.md có bảng đầy đủ; okr/SKILL.md heading + tóm tắt + link; docs/okr-system-review.md heading + link).

Run:

```bash
grep -cE "^\| Objective text, KR/KI" CLAUDE.md skills/okr/SKILL.md docs/okr-system-review.md
```

Expected: 1 ở CLAUDE.md, 0 ở 2 file còn lại.

Run:

```bash
grep -nE "Người, tool, ngân sách, PIC, khả dụng" docs/okr-system-review.md
```

Expected: 0 (đã xoá row PIC legacy).

- [ ] **Step 3: Verify Quality Gate shared file**

Run:

```bash
test -f skills/okr/references/quality-gate.md && echo "exists" || echo "missing"
```

Expected: `exists`.

Run:

```bash
grep -cE "skills/okr/references/quality-gate.md" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md
```

Expected: ≥1 ở mỗi file.

Run:

```bash
grep -cE "Đủ cụ thể\?|Giả định ẩn\?|Mâu thuẫn\?" skills/okr/references/quality-gate.md
```

Expected: 3 (canonical đủ 3 câu).

- [ ] **Step 4: Verify Phase 0a heading sửa xong**

Run:

```bash
grep -nE "^### Phase 0" skills/okr-init/SKILL.md
```

Expected: 2 match. `### Phase 0:` (Detect mode) + `### Phase 0a:` (Đọc inbox).

- [ ] **Step 5: Verify M7 + T4c trong okr-plan**

Run:

```bash
grep -cE "Bypass detect \(M7\)|context\.items\[\]|Pre-confirmed flow \(T4c\)" skills/okr-plan/SKILL.md
```

Expected: ≥3.

- [ ] **Step 6: Verify T4b trong okr-init**

Run:

```bash
grep -cE "Pre-confirmed flow \(T4b\)|pre_confirmed: true" skills/okr-init/SKILL.md
```

Expected: ≥2.

- [ ] **Step 7: Verify T4a trong okr-track**

Run:

```bash
grep -cE "All-changes preview \(gom theo skill đích\)|All-changes confirm" skills/okr-track/SKILL.md
```

Expected: ≥2.

Run:

```bash
awk '/^#### Bước 4:/,/^#### Bước 5:/' skills/okr-track/SKILL.md | grep -cE "Confirm tất cả"
```

Expected: ≥1.

- [ ] **Step 8: Verify T5 + M5a trong okr-capture**

Run:

```bash
grep -cE "^### Phase 5:" skills/okr-capture/SKILL.md
```

Expected: 0 (đã merge).

Run:

```bash
grep -cE "Reminder.*inbox|Inbox có N items pending" skills/okr-capture/SKILL.md
```

Expected: ≥1.

Run:

```bash
grep -cE "Không validate \`related_kr\`" skills/okr-capture/SKILL.md
```

Expected: 2 (Nguyên tắc + Quy tắc).

- [ ] **Step 9: Verify M5b trong okr-track**

Run:

```bash
grep -cE "Step 3.0: Validate related ID|Validate inbox related fields" skills/okr-track/SKILL.md
```

Expected: ≥2.

- [ ] **Step 10: Verify Minor2 trong okr-track**

Run:

```bash
grep -cE "Fallback \(Minor2\)|Ongoing chưa có practices" skills/okr-track/SKILL.md
```

Expected: ≥2.

- [ ] **Step 11: Verify lịch sử git**

Run:

```bash
git log --oneline -10
```

Expected: 10 commit gần nhất bao gồm (theo thứ tự task):
- T1 (Task 1)
- T3 (Task 2)
- T2 (Task 3)
- Minor1 (Task 4)
- M7 + T4c (Task 5)
- T4b (Task 6)
- T4a (Task 7)
- T5 + M5a (Task 8)
- M5b (Task 9)
- Minor2 (Task 10)

Tổng 10 commit Đợt 4.

- [ ] **Step 12: Status sạch**

Run:

```bash
git status
```

Expected: `nothing to commit, working tree clean` (ngoại trừ untracked files đã có trước plan: `deep-insight/`, file plan Wave 4 này, các plan Đợt 1-3).

---

## Tiêu chí hoàn thành Đợt 4

- [ ] 10 commit nhỏ trên branch hiện tại, mỗi commit reference review item.
- [ ] `skills/okr/SKILL.md` Bước 3 chỉ còn 1 bảng routing 3 cột (Trigger | Skill | Mode), không còn list "Keyword routing" rời.
- [ ] `CLAUDE.md` có section `## Phân vai SOT` với bảng 6 row canonical.
- [ ] `skills/okr/SKILL.md`, `docs/okr-system-review.md` đều **không còn bảng inline Phân vai SOT**, chỉ giữ heading + 1 đoạn link sang `CLAUDE.md`.
- [ ] `skills/okr-track/references/data-format.md` giữ bảng chi tiết Structure fields nhưng có note link sang CLAUDE.md là canonical.
- [ ] `skills/okr/references/quality-gate.md` tồn tại với 3 câu kiểm tra + bảng hành vi.
- [ ] `skills/okr-init/SKILL.md` + `skills/okr-plan/SKILL.md` section `## Quality Gate` ngắn (≤10 dòng mỗi cái), link sang `skills/okr/references/quality-gate.md`, kèm 3 ví dụ domain-specific.
- [ ] `skills/okr-init/SKILL.md` có 2 heading `### Phase 0` riêng biệt: `### Phase 0:` (Detect mode) + `### Phase 0a:` (Đọc inbox).
- [ ] `skills/okr-plan/SKILL.md` Mode UPDATE Phase 1+2 có block "Bypass detect (M7)" cho trường hợp track inbox delegate với `context.items[]`.
- [ ] `skills/okr-plan/SKILL.md` Phase 4 + `skills/okr-init/SKILL.md` Phase 6 có block "Pre-confirmed flow (T4c/T4b)" với hành vi skip ask khi `pre_confirmed: true`.
- [ ] `skills/okr-track/SKILL.md` Phase 4b Bước 4 đổi tên thành "All-changes confirm + chọn áp dụng", có bảng all-changes preview gom theo skill + dòng "Confirm tất cả?".
- [ ] `skills/okr-track/SKILL.md` Bước 5 ghi rõ `pre_confirmed: true` là **bắt buộc** sau Bước 4 confirm (không còn optional).
- [ ] `skills/okr-capture/SKILL.md` không còn `### Phase 5:` heading. Phase 4 có step 4 reminder khi inbox ≥5.
- [ ] `skills/okr-capture/SKILL.md` Nguyên tắc + Quy tắc đều có bullet "Không validate `related_kr`/`related_action`".
- [ ] `skills/okr-track/SKILL.md` Phase 5 Bước 3 có Step 3.0 validate related ID + bảng 3 row quy tắc.
- [ ] `skills/okr-track/SKILL.md` Phase 4a Ongoing step 4 có dòng "Fallback (Minor2)" cho trường hợp không có `## Practices`.

## Sau Đợt 4

Bộ skill OKR đã xong 27 items theo deep-insight (Đợt 1: 4 items sync doc, Đợt 2: 6 items solo defaults, Đợt 3: 8 items lifecycle gaps, Đợt 4: 9 items dọn thừa). Hệ thống đạt:

- 1 single entry point `/okr`, 4 skill con phân vai rõ.
- Solo-only default (verifier `self`, không PIC, resource form Solo Profile).
- 5 mode track nhất quán: `light`, `deep`, `closure`, `inbox-only`, `trace`.
- Practice + KI cho Ongoing, KR + actions cho Project.
- Lifecycle gap đã lấp (effort xl Checkpoints, period overdue, inbox aging, KR↔Action map dashboard, delegate reason).
- Duplicate đã dọn (Quality Gate shared, Phân vai SOT single source CLAUDE.md, routing 1 bảng).
- Pre-confirmed flow hoàn chỉnh (track gom confirm → init/plan honor signal).

Có thể revisit các item defer (G1 multi-objective, G7 forecast, G12 pause/resume) nếu phạm vi mở rộng.
