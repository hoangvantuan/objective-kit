# OKR Skill — Đợt 3: Lifecycle Gaps Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Lấp các khoảng trống lifecycle của bộ skill OKR sau khi đã sync tài liệu (Đợt 1) và gỡ team bias (Đợt 2). Đợt này thêm: (1) checkpoints cho effort `xl`, (2) practice streak update cho Ongoing, (3) cảnh báo period overdue, (4) KR ↔ Action map trong dashboard, (5) inbox aging warning, (6) delegate payload có `reason` để skill nhận biết lý do, (7) init quét inbox khi tạo objective, (8) impact check khi update-objective. Đồng thời sweep nốt 3 chỗ còn mention "PIC" còn sót sau Đợt 2 (theo ghi chú defer của user).

**Architecture:** Toàn bộ là markdown + YAML frontmatter refactor. Mỗi task = sửa 1 phạm vi rõ ràng trong 1-2 file, verify bằng `grep` trước/sau, commit ngay khi xanh. Một số task cần renumber phase (vd M4 thêm Impact Check vào mode update-objective: Phase 4 SMART → 5 Impact Check → 6 CONFIRM → 7 Ghi). Khi renumber, sửa cả các cross-reference nội bộ.

**Tech Stack:** Markdown + YAML frontmatter. Verify bằng `grep -nE` / `awk`. Không cần test framework.

**Phạm vi files (10 files):**

- `skills/okr-plan/references/task-format.md` (M6/K4 — Checkpoints schema + effort xl rule)
- `skills/okr-plan/references/action-guide.md` (M6/K4 — rewrite Effort xl behavior)
- `skills/okr-plan/SKILL.md` (M6/K4 — Quy tắc effort xl; G9 — delegate reason hiển thị; PIC sweep — 17 mention)
- `skills/okr-init/SKILL.md` (G11 — Phase 0 đọc inbox + Phase 7 suggest map; M4 — Impact Check phase mới; G9 — CONFIRM diff hiển thị reason)
- `skills/okr-track/SKILL.md` (G2/M9 — Phase 4a Ongoing update practice streak; G3 — Phase 2 cảnh báo period overdue; G4 — Phase 2 KR↔Action sub-line; G5 — Phase 5 cảnh báo stale; G9 — Phase 4b Bước 5 payload reason)
- `skills/okr-track/references/metrics.md` (G3 — period overdue rule)
- `skills/okr-track/references/data-format.md` (PIC sweep — 1 line)
- `skills/okr-capture/SKILL.md` (G5 — mention staleness compute ở track)
- `skills/okr-capture/references/data-format.md` (G5 — compute staleness_days note)
- `skills/okr-plan/references/data-format.md` (G2/M9 — clear TODO note ở Practices schema)
- `skills/okr/SKILL.md` (PIC sweep — 5 mention)

**Out of scope (Đợt 4):**

- T1-T5 dọn thừa (state+keyword merge, Quality Gate shared file, status table single source, gom confirm track deep, capture Phase 5 merge)
- M5 Capture đơn giản, validate ở track (đã set foundation Đợt 2 K5, sẽ hoàn tất Đợt 4)
- M7 Plan update bypass menu khi từ track (cần Đợt 3 G9 reason payload làm tiền đề)

---

## File Structure

| File | Phạm vi sửa | Lý do |
|------|-------------|-------|
| `skills/okr-plan/references/task-format.md` | Block YAML template + Effort table + Checkpoints schema | Effort xl bắt buộc Checkpoints body. |
| `skills/okr-plan/references/action-guide.md` | Section "Effort" (line 17-27) | Cảnh báo xl + bắt buộc checkpoints khi user giữ xl. |
| `skills/okr-plan/SKILL.md` | Quy tắc action (line 196-202); Phase 4 CONFIRM (line 162-176); Phase 2 menu, Phase 4 row, sửa action menu (line 150-176); description + rải PIC mention | xl rule + reason context hiển thị + sweep PIC. |
| `skills/okr-init/SKILL.md` | Phase 0 NEW (mới); Phase 7 NEW (mới); Mode UPDATE-OBJECTIVE Phase 5 mới (Impact Check); Phase 5 CONFIRM thêm reason | G11 + M4 + G9. |
| `skills/okr-track/SKILL.md` | Phase 2 dashboard (cả Project + Ongoing); Phase 4a Ongoing step practice; Phase 4b Bước 5 payload; Phase 5 inbox aging | G3 + G4 + G2/M9 + G9.a + G5.b. |
| `skills/okr-track/references/metrics.md` | Section mới "Period Overdue" | Quy tắc detect period quá hạn. |
| `skills/okr-track/references/data-format.md` | Line 65 row Resource | Đổi "người, tool, ngân sách, PIC, khả dụng" → "Solo Profile (capacity, skills), tool, ngân sách". |
| `skills/okr-capture/SKILL.md` | Mention staleness compute là việc của track | Đồng bộ M5 + G5. |
| `skills/okr-capture/references/data-format.md` | Section mới "Inbox Aging" | Doc compute staleness_days từ captured_at. |
| `skills/okr-plan/references/data-format.md` | Practices schema bullet TODO (line 91) | Xoá TODO note vì đã implement ở Task 3. |
| `skills/okr/SKILL.md` | description + bảng phân vai SOT + Resource line + routing tables | Sweep PIC khỏi orchestrator. |

---

## Thứ tự thực thi

Tasks 1+2 chạm `okr-plan/references/*.md`. Task 15 sweep PIC trong `okr-plan/SKILL.md` (lớn nhất). Tasks 8+9+10 (G9) chạm 3 file khác nhau (track + init + plan), nhưng đều là CONFIRM diff display, có thể tuần tự. Tasks 4+5+7+8 đều chạm `okr-track/SKILL.md` ở phase khác nhau → tuần tự để tránh conflict line number. Tasks 11+12 chạm `okr-init/SKILL.md` ở phase khác nhau (Phase 0/7 NEW vs Phase 5 UPDATE-OBJECTIVE) → tuần tự an toàn.

**Đặc biệt**: Task 12 (M4) renumber phase trong mode UPDATE-OBJECTIVE. Sau Task 12, mode UPDATE-OBJECTIVE có 7 phase: 1 → 2 → 3 → 4 SMART → 5 Impact Check → 6 CONFIRM → 7 Ghi. Task 9 (G9.b) thêm reason vào Phase 6 CONFIRM (cũ là Phase 5). Phải làm Task 9 SAU Task 12 nếu không sẽ chỉ vào sai line. Plan sắp xếp: Task 9 → 12 vì Task 9 sửa CONFIRM hiện tại (Phase 5), Task 12 sau đó renumber và edit lại CONFIRM thành Phase 6. → Đảo lại: Task 12 trước, Task 9 sau.

Thứ tự cuối cùng: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 10 → 12 → 9 → 11 → 13 → 14 → 15 → 16. (Task 9 G9.b làm sau Task 12 M4 vì Task 12 đổi số phase trong mode UPDATE-OBJECTIVE.)

---

## Task 1: M6/K4.a — `task-format.md` thêm Checkpoints schema + rule effort xl

**Files:**

- Modify: `skills/okr-plan/references/task-format.md:27-44` (Body template) + `skills/okr-plan/references/task-format.md:62-70` (Effort values table)

**Mục tiêu:** Template body action thêm `## Checkpoints` (optional). Bảng Effort values dòng `xl` ghi rõ "BẮT BUỘC body có `## Checkpoints` ≥2 mục". Solo user vẫn có thể chọn `xl` nhưng phải tự chia nhỏ thành các mốc kiểm tra giữa kỳ.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "## Checkpoints|effort|xl" skills/okr-plan/references/task-format.md
```

Expected: 0 match cho `## Checkpoints`. Match cho `effort` (line 20 frontmatter, line 62 heading, line 70 row xl).

- [ ] **Step 2: Thêm `## Checkpoints` (optional, only effort xl) vào template body**

Edit `skills/okr-plan/references/task-format.md`:

old_string:

```
## Tiêu chí chất lượng

[Output đạt khi nào? Đo bằng gì? Vd: "Report cover đủ 5 đối thủ, mỗi đối thủ có ≥3 điểm so sánh"]

## Ghi chú

[Context bổ sung, ràng buộc, link tham khảo]
```

new_string:

```
## Tiêu chí chất lượng

[Output đạt khi nào? Đo bằng gì? Vd: "Report cover đủ 5 đối thủ, mỗi đối thủ có ≥3 điểm so sánh"]

## Checkpoints (BẮT BUỘC khi effort=xl, optional khác)

- [ ] [Mốc 1: phase/phần đầu xong vào ngày YYYY-MM-DD]
- [ ] [Mốc 2: phase/phần giữa xong vào ngày YYYY-MM-DD]
- [ ] [Mốc 3 (optional): phase/phần cuối xong vào ngày YYYY-MM-DD]

## Ghi chú

[Context bổ sung, ràng buộc, link tham khảo]
```

- [ ] **Step 3: Sửa bảng Effort values dòng xl**

Edit `skills/okr-plan/references/task-format.md`:

old_string:

```
| Value | Thời gian ước lượng |
|-------|-------------------|
| xs | < 1 giờ |
| s | 1-4 giờ |
| m | 1-2 ngày |
| l | 3-5 ngày |
| xl | > 1 tuần |
```

new_string:

```
| Value | Thời gian ước lượng | Quy tắc đặc biệt |
|-------|---------------------|------------------|
| xs | < 1 giờ | - |
| s | 1-4 giờ | - |
| m | 1-2 ngày | - |
| l | 3-5 ngày | - |
| xl | > 1 tuần | **BẮT BUỘC body có `## Checkpoints` ≥2 mục, mỗi mục có ngày deadline cụ thể**. Track đọc Checkpoints để cảnh báo trượt giữa kỳ. Nếu không có Checkpoints, action không hợp lệ. |
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "## Checkpoints \(BẮT BUỘC|BẮT BUỘC body có \`## Checkpoints\` ≥2" skills/okr-plan/references/task-format.md
```

Expected: 2 match (template body + Effort table).

Run:

```bash
awk '/^##/{print NR, $0}' skills/okr-plan/references/task-format.md
```

Expected: section order body template gồm `Definition of Done` → `Output/Deliverable` → `Tiêu chí chất lượng` → `Checkpoints (BẮT BUỘC khi effort=xl, optional khác)` → `Ghi chú`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-plan/references/task-format.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): add Checkpoints section + effort xl rule (M6/K4)

Effort xl trước đây chỉ cảnh báo "tách thành m/l", user vẫn ghi xl
được nhưng không có structure track tiến độ giữa kỳ. Thêm:
- Template body có section "## Checkpoints (BẮT BUỘC khi effort=xl)"
- Bảng Effort values cột mới "Quy tắc đặc biệt": xl phải có
  Checkpoints ≥2 mục với ngày deadline cụ thể.
Track đọc Checkpoints để cảnh báo trượt giữa kỳ.

Refs: deep-insight/okr-skill-review-260510.md M6, K4
EOF
)"
```

---

## Task 2: M6/K4.b — `action-guide.md` rewrite Effort xl behavior

**Files:**

- Modify: `skills/okr-plan/references/action-guide.md:17-27` (section Effort + line cảnh báo)

**Mục tiêu:** Hướng dẫn rõ flow khi user định ghi `effort: xl`: (1) gợi ý tách trước, (2) nếu user vẫn giữ xl → BẮT BUỘC tạo `## Checkpoints` ≥2 mục với ngày deadline cụ thể.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '17,30p' skills/okr-plan/references/action-guide.md
```

Expected: thấy bảng Effort + line "Nếu effort = `xl`, hỏi user: 'Task này khá lớn. Có thể tách thành 2-3 tasks nhỏ hơn không?'".

- [ ] **Step 2: Thay block Effort + line cảnh báo xl**

Edit `skills/okr-plan/references/action-guide.md`:

old_string:

```
## Effort (ước lượng độ lớn)

| Value | Thời gian | Khi nào dùng |
|-------|-----------|-------------|
| `xs` | < 1 giờ | Config, fix nhỏ, gửi email |
| `s` | 1-4 giờ | Viết 1 function, 1 trang doc |
| `m` | 1-2 ngày | 1 feature nhỏ, 1 report |
| `l` | 3-5 ngày | 1 module, design hoàn chỉnh |
| `xl` | > 1 tuần | Nên tách thành nhiều actions nhỏ hơn |

Nếu effort = `xl`, hỏi user: "Task này khá lớn. Có thể tách thành 2-3 tasks nhỏ hơn không?"
```

new_string:

```
## Effort (ước lượng độ lớn)

| Value | Thời gian | Khi nào dùng |
|-------|-----------|-------------|
| `xs` | < 1 giờ | Config, fix nhỏ, gửi email |
| `s` | 1-4 giờ | Viết 1 function, 1 trang doc |
| `m` | 1-2 ngày | 1 feature nhỏ, 1 report |
| `l` | 3-5 ngày | 1 module, design hoàn chỉnh |
| `xl` | > 1 tuần | Cảnh báo overrun cao, ưu tiên tách. Nếu giữ → BẮT BUỘC Checkpoints. |

### Quy tắc cho effort = xl

Solo user thường underestimate task xl: thực tế overrun gấp 1.5-2× ước lượng ban đầu (không có review giữa kỳ → không phát hiện trượt sớm). Quy trình bắt buộc khi user định ghi `effort: xl`:

1. **Hỏi tách trước**: "Task này khá lớn (>1 tuần). Có thể tách thành 2-3 tasks `m` hoặc `l` không?". Đề xuất ngay split candidate dựa trên DoD nếu thấy được.
2. **Nếu user giữ xl** → BẮT BUỘC tạo body section `## Checkpoints` với ≥2 mục, mỗi mục:
   - 1 dòng mô tả mốc đạt được (vd "Phase 1: Spec hoàn chỉnh")
   - Ngày deadline cụ thể (vd "by 2026-11-15")
   - Format: `- [ ] [Mô tả mốc] (by YYYY-MM-DD)`
3. **Nếu user từ chối tạo Checkpoints** → action không hợp lệ. Quay lại bước 1 (yêu cầu tách).

Track đọc Checkpoints khi chạy `light` / `deep`: mốc nào quá hạn mà chưa tick → cảnh báo "Action AXXX trượt checkpoint N". User có thể update tiến độ checkpoint riêng (tick checkbox) trước khi đánh action `done`.

Ví dụ Checkpoints hợp lệ:

```markdown
## Checkpoints
- [ ] Phase 1: Khảo sát user xong (by 2026-11-08)
- [ ] Phase 2: Spec MVP draft (by 2026-11-15)
- [ ] Phase 3: Review + finalize (by 2026-11-22)
```
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "BẮT BUỘC Checkpoints|Quy tắc cho effort = xl|by YYYY-MM-DD|cảnh báo \"Action AXXX trượt checkpoint" skills/okr-plan/references/action-guide.md
```

Expected: ≥4 match.

Run:

```bash
grep -n "Nên tách thành nhiều actions nhỏ hơn" skills/okr-plan/references/action-guide.md
```

Expected: KHÔNG có match (đã thay).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/references/action-guide.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): mandate Checkpoints when effort=xl (M6/K4)

Effort xl chỉ cảnh báo soft, user vẫn ghi xl được mà không có
structure track giữa kỳ. Rewrite section Effort:
- Bảng Effort dòng xl: "BẮT BUỘC Checkpoints"
- Section mới "Quy tắc cho effort = xl": flow 3 bước (hỏi tách →
  nếu giữ xl → bắt buộc Checkpoints ≥2 mục với ngày → từ chối
  thì quay lại tách).
- Format Checkpoints item: "- [ ] [Mô tả] (by YYYY-MM-DD)".
Track đọc Checkpoints để cảnh báo trượt giữa kỳ.

Refs: deep-insight/okr-skill-review-260510.md M6, K4
EOF
)"
```

---

## Task 3: G2/M9 — `okr-track/SKILL.md` Phase 4a Ongoing update practice streak + clear TODO

**Files:**

- Modify: `skills/okr-track/SKILL.md:142-152` (Phase 4a Ongoing flow)
- Modify: `skills/okr-plan/references/data-format.md:88-93` (Practices schema bullet "current_streak update by track")

**Mục tiêu:** Track Ongoing có bước hỏi practice streak. Sau khi user xác nhận tuần này đạt practice nào, agent update `current_streak` trong `plan.md` body section `## Practices`. Đồng thời xoá TODO note đã ghi ở Đợt 1 (M9.b) trong `okr-plan/references/data-format.md`.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '142,154p' skills/okr-track/SKILL.md
```

Expected: thấy "**Ongoing type:**" với 9 step, KHÔNG có step nào liên quan practices.

Run:

```bash
grep -n "TODO (wave 3, item G2/M9)" skills/okr-plan/references/data-format.md
```

Expected: 1 match line ~91.

- [ ] **Step 2: Thay block Ongoing flow trong Phase 4a (thêm step practices)**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
**Ongoing type:**

1. Hiển thị KI hiện tại (tên, ngưỡng, current, status).
2. Hỏi user update từng KI: "KI1 (Tập thể dục, hiện tại: 2, ngưỡng: ≥3). Tuần này bao nhiêu?"
3. Tính status mới theo logic: healthy (≥ ngưỡng), warning (< ngưỡng, chênh < 20%), critical (< 80% ngưỡng). Xem `references/metrics.md`.
4. Nếu có action files (task cải thiện KI) → hỏi update status (pending/doing/done/blocked) như Project.
5. CONFIRM trước khi ghi (tương tự Project: ≤2 field → 1 dòng, ≥3 field → bảng).
6. Áp dụng: ghi đè `objective.md` (KI current, status). Nếu có actions → cập nhật `plan.md` counters + `actions/*.md`.
7. Append log.
8. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
9. Đề xuất: nếu KI warning/critical → gợi ý tạo action cải thiện qua `/okr plan`.
```

new_string:

```
**Ongoing type:**

1. Hiển thị KI hiện tại (tên, ngưỡng, current, status).
2. Hỏi user update từng KI: "KI1 (Tập thể dục, hiện tại: 2, ngưỡng: ≥3). Tuần này bao nhiêu?"
3. Tính status mới theo logic: healthy (≥ ngưỡng), warning (< ngưỡng, chênh < 20%), critical (< 80% ngưỡng). Xem `references/metrics.md`.
4. **Update practice streak**: đọc `plan.md` body section `## Practices` (nếu có). Với mỗi practice (`### PN: <Tên>` kèm field `frequency`, `target_count`, `current_streak`, `ki_link`):
   - Tính chu kỳ vừa qua dựa trên `frequency` (vd `weekly` = tuần vừa rồi, `daily` = hôm qua). Hỏi: "Practice **P1: Tập thể dục** (target ≥3 lần/tuần, streak hiện tại: 2). Tuần vừa rồi đạt target chưa? (y = đạt → streak +1 / n = không đạt → reset streak về 0 / skip = bỏ qua không update)".
   - User trả lời `y` → `current_streak += 1`. `n` → `current_streak = 0`. `skip` → giữ nguyên.
   - Nếu practice `ki_link` đã có KI tương ứng vừa update ở step 2-3, hiển thị inline để user thấy liên kết: `(P1 → KI1 healthy)`.
5. Nếu có action files (task cải thiện KI) → hỏi update status (pending/doing/done/blocked) như Project.
6. CONFIRM trước khi ghi (tương tự Project: ≤2 field → 1 dòng, ≥3 field → bảng). Bao gồm cả `current_streak` thay đổi vào diff list.
7. Áp dụng: ghi đè `objective.md` (KI current, status), ghi đè `plan.md` body section `## Practices` (chỉ field `current_streak` cho practice nào thay đổi). Nếu có actions → cập nhật `plan.md` counters + `actions/*.md`.
8. Append log: ngoài KI/action, ghi cả practice streak thay đổi (vd `## Thay đổi → P1.current_streak: 2 > 3`).
9. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
10. Đề xuất: nếu KI warning/critical → gợi ý tạo action cải thiện qua `/okr plan`. Nếu practice streak vừa reset về 0 → cảnh báo "Practice P1 đã reset streak, KI1 có thể giảm tuần tới. Cân nhắc điều chỉnh practice (giảm target_count, đổi thời gian) qua `/okr plan update`".
```

- [ ] **Step 3: Xoá TODO note trong `okr-plan/references/data-format.md`**

Edit `skills/okr-plan/references/data-format.md`:

old_string:

```
- `current_streak` được update bởi `okr-track` Ongoing flow (Phase 4a Ongoing): hỏi user "Tuần này đạt practice nào?" → tăng/reset streak. **TODO (wave 3, item G2/M9)**: bước này chưa implement trong `skills/okr-track/SKILL.md` Phase 4a, sẽ thêm ở wave 3.
```

new_string:

```
- `current_streak` được update bởi `okr-track` Ongoing flow (Phase 4a Ongoing step 4 "Update practice streak"): với mỗi practice, agent hỏi "Tuần vừa rồi đạt target chưa? (y/n/skip)". `y` → +1, `n` → reset 0, `skip` → giữ nguyên. Track ghi đè `current_streak` trong `plan.md` body khi user confirm.
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "Update practice streak|streak \+= 1|streak = 0|P1 → KI1 healthy" skills/okr-track/SKILL.md
```

Expected: ≥4 match.

Run:

```bash
grep -nE "TODO \(wave 3" skills/okr-plan/references/data-format.md
```

Expected: KHÔNG có match.

Run:

```bash
grep -nE "Phase 4a Ongoing step 4 \"Update practice streak\"" skills/okr-plan/references/data-format.md
```

Expected: 1 match.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md skills/okr-plan/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-track): implement practice streak update in Ongoing flow (G2/M9)

Practices schema đã có sẵn ở plan.md (Đợt 1 M9), nhưng track
chưa biết cách update current_streak. Thêm Phase 4a Ongoing step
4 "Update practice streak": với mỗi practice, hỏi y/n/skip,
update streak tương ứng. Step 6 CONFIRM bao gồm streak diff,
step 7 ghi đè plan.md body, step 8 log practice change, step 10
cảnh báo khi streak reset 0. Đồng thời xoá TODO note ở
okr-plan/references/data-format.md (đã implement).

Refs: deep-insight/okr-skill-review-260510.md G2, M9
EOF
)"
```

---

## Task 4: G3 — `okr-track/SKILL.md` Phase 2 cảnh báo period overdue + `metrics.md` rule

**Files:**

- Modify: `skills/okr-track/SKILL.md:42-62` (Phase 2 dashboard Project block)
- Modify: `skills/okr-track/references/metrics.md` (thêm section "## Period Overdue (Project type)" sau "## Trend (Project type)")

**Mục tiêu:** Phase 2 dashboard detect `now > end_date AND objective.status = active` (Project type). Khi detect → hiển thị cảnh báo block "⚠️ Period đã qua [N] ngày" + đề xuất extend hoặc đổi status. Document rule chi tiết trong `metrics.md`.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '42,65p' skills/okr-track/SKILL.md
```

Expected: thấy block "**Project type:**" + dashboard mẫu có dòng `Period: 2026-10-01 > 2026-12-31 (60% thời gian đã dùng)` + section "Cần chú ý".

Run:

```bash
grep -n "Period Overdue\|period overdue\|now > end_date" skills/okr-track/references/metrics.md
```

Expected: KHÔNG có match (chưa document).

- [ ] **Step 2: Thay block dashboard Project (thêm cảnh báo period overdue)**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
**Project type:**

```
Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (60% thời gian đã dùng)

Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
  KR3: ██████░░░░ 3/5    (60%) > ahead

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending
Tốc độ: 1.2 done/tuần (kế hoạch: 1.5)
Inbox: 3 items chưa xử lý

Cần chú ý
  - A005 blocked 5 ngày: chờ approve từ stakeholder
  - A007 quá hạn 3 ngày
  - KR2 cần +8 đơn vị/tháng để kịp target
```
```

new_string:

```
**Project type:**

Trước khi render dashboard, tính `period_overdue_days = max(0, today - end_date)`. Nếu `period_overdue_days > 0` AND `objective.status = active` → render dashboard kèm block cảnh báo period overdue ở vị trí ĐẦU TIÊN, trên cả Period progress (xem mẫu thứ 2 dưới).

Dashboard chuẩn (period chưa quá hạn):

```
Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (60% thời gian đã dùng)

Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
  KR3: ██████░░░░ 3/5    (60%) > ahead

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending
Tốc độ: 1.2 done/tuần (kế hoạch: 1.5)
Inbox: 3 items chưa xử lý

Cần chú ý
  - A005 blocked 5 ngày: chờ approve từ stakeholder
  - A007 quá hạn 3 ngày
  - KR2 cần +8 đơn vị/tháng để kịp target
```

Dashboard khi period đã qua (`period_overdue_days > 0` AND `status = active`):

```
⚠️ Period đã qua 12 ngày
End date 2026-12-31, hôm nay 2027-01-12. Status objective vẫn `active`.

KR chưa achieved:
  - KR1: 80/100 (80%) — còn thiếu 20 đơn vị
  - KR2: 35/50  (70%) — còn thiếu 15 đơn vị

Đề xuất:
  - Extend end_date qua `/okr init update-objective` nếu vẫn muốn theo đuổi target
  - Đổi status sang `completed` (chấp nhận kết quả hiện tại) hoặc `cancelled`
    (dừng theo đuổi) qua `/okr init update-objective`

Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (period đã đóng, +12 ngày)
... (phần còn lại giống dashboard chuẩn)
```

Logic chi tiết: xem `references/metrics.md` section "Period Overdue (Project type)".
```

- [ ] **Step 3: Thêm section "Period Overdue" vào `metrics.md`**

Edit `skills/okr-track/references/metrics.md`:

old_string:

```
## Trend (Ongoing type)

Ongoing không dùng timeline trend. Thay vào đó, so sánh status hiện tại vs lần review trước:
- `improving`: có KI chuyển từ critical/warning → healthy
- `stable`: không đổi
- `declining`: có KI chuyển từ healthy → warning/critical
```

new_string:

```
## Trend (Ongoing type)

Ongoing không dùng timeline trend. Thay vào đó, so sánh status hiện tại vs lần review trước:
- `improving`: có KI chuyển từ critical/warning → healthy
- `stable`: không đổi
- `declining`: có KI chuyển từ healthy → warning/critical

## Period Overdue (Project type)

Áp dụng chỉ cho `type: project`. Ongoing không có deadline → không cần.

### Detect

```
period_overdue_days = max(0, today - end_date)
overdue = (period_overdue_days > 0) AND (objective.status == "active")
```

Ví dụ: `end_date = 2026-12-31`, `today = 2027-01-12`, `status = active` → `overdue = true`, `period_overdue_days = 12`.

### Hành vi của okr-track

- Phase 2 dashboard: nếu `overdue` → render block cảnh báo ở vị trí đầu (trước Key Results), liệt kê KR chưa `achieved` + đề xuất hành động (extend end_date hoặc đổi status).
- Không tự đổi status. Chỉ ĐỀ XUẤT, user quyết.
- Nếu `objective.status` đã là `completed` / `cancelled` / `paused` / `archived` → KHÔNG render cảnh báo (period overdue chỉ relevant khi user vẫn theo đuổi).

### Đề xuất hành động (in trong cảnh báo)

| Tình huống | Đề xuất |
|------------|---------|
| Còn KR gần đạt (≥80%) | Extend end_date thêm vài tuần qua `okr-init update-objective`, deadline mới thực tế dựa trên tốc độ hiện tại |
| Mọi KR < 50% | Cân nhắc `cancelled` (scope không khả thi với capacity) hoặc reset KR target qua `update-objective` |
| Mix (1 KR đạt, 1 KR cách xa) | Chia objective: đóng phần đạt thành `completed`, tách phần còn lại thành objective mới |

Đề xuất chỉ là gợi ý. Track không enforce, user tự chọn.
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "Period đã qua|period_overdue_days|Period Overdue \(Project type\)" skills/okr-track/SKILL.md skills/okr-track/references/metrics.md
```

Expected: ≥3 match (1 trong SKILL.md dashboard mẫu, 1 link sang metrics.md, 1+ trong metrics.md).

Run:

```bash
awk '/^## /{print NR, $0}' skills/okr-track/references/metrics.md
```

Expected: section order kết thúc bằng `Trend (Ongoing type)` → `Period Overdue (Project type)` → `Cập nhật SOT` → `Log format`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md skills/okr-track/references/metrics.md
git commit -m "$(cat <<'EOF'
docs(okr-track): warn when period overdue (G3)

Project end_date qua mà status vẫn active → hiện tại track render
dashboard bình thường, user dễ quên đóng objective. Phase 2 detect
period_overdue_days > 0 AND status = active → render block cảnh báo
ở đầu dashboard, liệt kê KR chưa achieved + đề xuất extend/đổi
status. Không tự đổi, chỉ đề xuất qua /okr init update-objective.
Document rule chi tiết + bảng đề xuất ở metrics.md section
"Period Overdue (Project type)".

Refs: deep-insight/okr-skill-review-260510.md G3
EOF
)"
```

---

## Task 5: G4 — `okr-track/SKILL.md` Phase 2 dashboard KR↔Action sub-line

**Files:**

- Modify: `skills/okr-track/SKILL.md:48-52` (block "Key Results" trong dashboard mẫu Project)

**Mục tiêu:** Mỗi row KR trong dashboard Project kèm sub-line liệt kê actions thuộc KR đó (count theo status + danh sách active IDs). Khi KR at-risk, user thấy ngay actions nào liên quan để follow-up.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '48,53p' skills/okr-track/SKILL.md
```

Expected: 3 row KR ngắn không có sub-line.

- [ ] **Step 2: Thay block Key Results (thêm sub-line actions)**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
  KR3: ██████░░░░ 3/5    (60%) > ahead
```

new_string:

```
Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
    Actions: 3 done | 2 doing | 1 blocked | 0 pending
    Active: A001 (doing), A003 (doing), A005 (blocked: chờ approve)
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
    Actions: 1 done | 1 doing | 0 blocked | 2 pending
    Active: A007 (doing), A009 (pending), A010 (pending)
  KR3: ██████░░░░ 3/5    (60%) > ahead
    Actions: 0 done | 0 doing | 0 blocked | 1 pending
    Active: A012 (pending)
```

- [ ] **Step 3: Thêm logic build sub-line vào Phase 1 (đọc actions group by KR)**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
Tính metrics (xem `references/metrics.md`):

- Project: KR % đạt, trend so với log trước, timeline, actions tổng/done/doing/blocked/pending, tốc độ done/tuần
- Ongoing: KI status (healthy/warning/critical), trend so với review trước
- Inbox: số items pending
```

new_string:

```
Tính metrics (xem `references/metrics.md`):

- Project:
  - KR % đạt, trend so với log trước, timeline, actions tổng/done/doing/blocked/pending, tốc độ done/tuần
  - **Actions group by KR**: với mỗi KR, lọc `actions/*.md` có frontmatter `key_result: KR<N>`. Đếm theo status (done/doing/blocked/pending). Liệt kê IDs active (status ∈ {doing, blocked, pending}, ưu tiên blocked + doing trước, pending sau). Nếu >5 active actions → hiển thị 3 đầu + "(+N nữa)". Mỗi blocked action kèm lý do ngắn từ field `description` hoặc 1 dòng đầu của body.
- Ongoing: KI status (healthy/warning/critical), trend so với review trước. Practices streak hiện tại (đọc từ `plan.md` body).
- Inbox: số items pending
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "Active: A001 \(doing\)|Actions group by KR|Actions: 3 done \| 2 doing" skills/okr-track/SKILL.md
```

Expected: ≥3 match.

Run:

```bash
grep -c "    Actions:" skills/okr-track/SKILL.md
```

Expected: 3 (1 cho mỗi KR row trong dashboard mẫu).

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-track): add KR↔Action sub-line to dashboard (G4)

Dashboard chỉ hiển thị tổng count actions, KR at-risk không thấy
ngay actions nào thuộc KR đó. Phase 1 thêm logic group actions by
key_result, Phase 2 mỗi KR row kèm sub-line:
- Count theo status (done/doing/blocked/pending)
- Active IDs (ưu tiên blocked > doing > pending), >5 thì "(+N nữa)"
- Blocked kèm lý do ngắn

User nhìn KR2 at-risk biết ngay 3 actions liên quan, follow-up
trực tiếp thay vì phải scroll list 12 actions.

Refs: deep-insight/okr-skill-review-260510.md G4
EOF
)"
```

---

## Task 6: G5.a — `okr-capture` document staleness compute (data-format + SKILL.md note)

**Files:**

- Modify: `skills/okr-capture/references/data-format.md` (thêm section "## Inbox Aging" trước "## Quy tắc")
- Modify: `skills/okr-capture/SKILL.md` (thêm note ngắn về staleness ở Quy tắc cuối file nếu có; nếu không thì bỏ qua)

**Mục tiêu:** Document quy tắc compute `staleness_days = today - captured_at` (compute on-the-fly khi đọc, KHÔNG ghi vào frontmatter). `okr-track` Phase 5 dùng để cảnh báo items cũ. Capture chỉ tạo file, không lưu staleness.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -n "staleness_days\|Inbox Aging" skills/okr-capture/references/data-format.md skills/okr-capture/SKILL.md
```

Expected: KHÔNG có match.

- [ ] **Step 2: Thêm section "## Inbox Aging" vào `data-format.md`**

Edit `skills/okr-capture/references/data-format.md`:

old_string:

```
> **Migrate dữ liệu cũ**: File inbox cũ với `type: idea` hoặc `type: note` → coi như `type: thought` khi xử lý. `okr-track` Phase 5 (inbox processing) phát hiện type cũ → tự update sang `thought` khi đổi `status: processed`.

Sau khi xử lý: đổi status thành `processed`. File giữ nguyên trong inbox (lịch sử).

## Quy tắc
```

new_string:

```
> **Migrate dữ liệu cũ**: File inbox cũ với `type: idea` hoặc `type: note` → coi như `type: thought` khi xử lý. `okr-track` Phase 5 (inbox processing) phát hiện type cũ → tự update sang `thought` khi đổi `status: processed`.

Sau khi xử lý: đổi status thành `processed`. File giữ nguyên trong inbox (lịch sử).

## Inbox Aging

Trường `staleness_days = today - captured_at` được compute **on-the-fly** khi đọc, KHÔNG lưu vào frontmatter. Lý do: captured_at là dữ liệu gốc, staleness phụ thuộc thời điểm đọc → tính lại mỗi lần đọc tránh stale-of-stale.

Ngưỡng:

| Phạm vi | Nghĩa | Hành vi của okr-track |
|---------|-------|------------------------|
| `staleness_days ≤ 7` | Mới | Hiển thị bình thường, không cảnh báo |
| `7 < staleness_days ≤ 30` | Đang chờ xử lý | Hiển thị bình thường, sort lên đầu nếu nhiều items |
| `staleness_days > 30` | Cũ | Phase 5 cảnh báo "Inbox cũ ≥30 ngày: [list]. Còn relevant không? (giữ/bỏ)". Không auto-discard. |

Phạm vi áp dụng: chỉ items có `status: pending`. Items đã `processed` / `discarded` không đếm.

## Quy tắc
```

- [ ] **Step 3: Thêm bullet vào `okr-capture/SKILL.md` Quy tắc**

Edit `skills/okr-capture/SKILL.md`:

old_string:

```
- Nếu `.okr/` chưa có → vẫn tạo `.okr/inbox/` + cảnh báo chạy init.
```

new_string:

```
- Nếu `.okr/` chưa có → vẫn tạo `.okr/inbox/` + cảnh báo chạy init.
- KHÔNG ghi `staleness_days` vào frontmatter. Capture chỉ lưu `captured_at`. Staleness được compute on-the-fly bởi `okr-track` Phase 5 (xem `references/data-format.md` section "Inbox Aging").
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "## Inbox Aging|staleness_days = today - captured_at|on-the-fly" skills/okr-capture/references/data-format.md
```

Expected: ≥3 match.

Run:

```bash
awk '/^## /{print NR, $0}' skills/okr-capture/references/data-format.md
```

Expected: thứ tự `Inbox item file` → `Status values` → `Xử lý inbox` → `Inbox Aging` → `Quy tắc`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-capture/references/data-format.md skills/okr-capture/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-capture): document inbox staleness compute (G5)

Inbox dễ tích lũy items pending mãi, capture không có cách báo
cũ. Document quy tắc:
- staleness_days = today - captured_at, compute on-the-fly
- KHÔNG lưu vào frontmatter (tránh stale-of-stale)
- 3 ngưỡng: ≤7 (mới), 7-30 (đang chờ), >30 (cũ → track Phase 5
  cảnh báo)
- Chỉ áp dụng cho status: pending

Capture vẫn chỉ tạo file. Logic cảnh báo + hỏi giữ/bỏ ở
okr-track (Task 7 G5.b).

Refs: deep-insight/okr-skill-review-260510.md G5
EOF
)"
```

---

## Task 7: G5.b — `okr-track/SKILL.md` Phase 5 cảnh báo stale items >30 ngày

**Files:**

- Modify: `skills/okr-track/SKILL.md:251-268` (Phase 5 Bước 1 + Bước 2)

**Mục tiêu:** Phase 5 inbox processing đọc `captured_at` → compute staleness. Trước khi hiển thị bảng inbox bình thường, nếu có items stale >30 ngày → block cảnh báo riêng "Inbox cũ ≥30 ngày" hỏi user giữ/bỏ. Không auto-discard.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '247,270p' skills/okr-track/SKILL.md
```

Expected: thấy "### Phase 5: Inbox Processing Flow" → "#### Bước 1: Đọc inbox" → "#### Bước 2: Hiển thị inbox + gợi ý xử lý" với bảng 3 row mẫu.

- [ ] **Step 2: Thay Bước 1 (thêm compute staleness) + thêm Bước 1.5 cảnh báo stale**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
#### Bước 1: Đọc inbox

Đọc tất cả `.okr/inbox/*.md` có `status: pending`. Nếu không có → skip.

#### Bước 2: Hiển thị inbox + gợi ý xử lý
```

new_string:

```
#### Bước 1: Đọc inbox + compute staleness

Đọc tất cả `.okr/inbox/*.md` có `status: pending`. Nếu không có → skip toàn bộ Phase 5.

Với mỗi item, compute `staleness_days = today - captured_at` (xem `skills/okr-capture/references/data-format.md` section "Inbox Aging"). Phân loại:

- Mới (≤7 ngày): xử lý bình thường ở Bước 2.
- Đang chờ (7 < staleness ≤ 30): xử lý bình thường, nhưng sort lên đầu trong bảng Bước 2.
- Cũ (>30 ngày): chuyển sang Bước 1.5 xử lý riêng trước.

#### Bước 1.5: Cảnh báo stale items (chỉ chạy nếu có items >30 ngày)

Hiển thị block riêng TRƯỚC bảng Bước 2:

```
⚠️ Inbox cũ ≥30 ngày (3 items)
| # | Type     | Title                        | Captured  | Staleness |
|---|----------|------------------------------|-----------|-----------|
| 1 | thought  | Thử framework X cho frontend | 2026-04-01 | 40 ngày  |
| 2 | action   | Viết blog post về Y          | 2026-03-25 | 47 ngày  |
| 3 | resource | Library Z hỗ trợ chart       | 2026-03-15 | 57 ngày  |

Còn relevant không? (vd: "1 giữ, 2 bỏ, 3 giữ" / all giữ / all bỏ)
```

User trả lời:
- `<N> giữ`: giữ `status: pending` (không thay đổi). Item đi tiếp vào Bước 2 xử lý bình thường.
- `<N> bỏ`: đổi `status: discarded` ngay (không cần qua Bước 2-3).
- `all giữ`: giữ tất cả, đẩy hết sang Bước 2.
- `all bỏ`: discard tất cả.

KHÔNG auto-discard. User quyết định cuối.

Sau Bước 1.5, đi tiếp Bước 2 với items đã filter (đã giữ + items ≤30 ngày).

#### Bước 2: Hiển thị inbox + gợi ý xử lý
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "compute staleness|Bước 1.5: Cảnh báo stale|Inbox cũ ≥30 ngày|all giữ \| all bỏ" skills/okr-track/SKILL.md
```

Expected: ≥3 match (cho phép syntax `|` không escape match).

Run:

```bash
grep -nE "^#### Bước 1: Đọc inbox \+ compute staleness|^#### Bước 1.5: Cảnh báo stale items|^#### Bước 2: Hiển thị inbox" skills/okr-track/SKILL.md
```

Expected: 3 match (3 heading liền nhau).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-track): warn stale inbox items >30 days (G5)

Inbox tích lũy items pending mãi, không có cảnh báo. Phase 5:
- Bước 1 thêm compute staleness_days từ captured_at, phân loại
  3 nhóm (mới ≤7 / chờ 7-30 / cũ >30).
- Bước 1.5 mới: items cũ >30 ngày hiển thị riêng, hỏi giữ/bỏ
  trước bảng xử lý chính. Không auto-discard.
- Bước 2 (cũ) giữ nguyên, chỉ chạy với items đã filter.

User quyết định cuối, agent chỉ surface stale items để user
khỏi quên review.

Refs: deep-insight/okr-skill-review-260510.md G5
EOF
)"
```

---

## Task 8: G9.a — `okr-track/SKILL.md` Phase 4b Bước 5 delegate payload có reason

**Files:**

- Modify: `skills/okr-track/SKILL.md:204-216` (Phase 4b Bước 5: Delegate sang skill phù hợp)

**Mục tiêu:** Document payload format khi track `deep` delegate sang init/plan. Payload có `changes`, `reason`, `source_review` để skill nhận biết "ai gửi context và vì sao". Skill nhận sẽ hiển thị `reason` trong phase confirm (Task 9 + 10).

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '204,216p' skills/okr-track/SKILL.md
```

Expected: thấy "#### Bước 5: Delegate sang skill phù hợp" với mô tả ngắn về delegate, KHÔNG có format payload chi tiết.

- [ ] **Step 2: Thay Bước 5 (thêm format payload)**

Edit `skills/okr-track/SKILL.md`:

old_string:

```
#### Bước 5: Delegate sang skill phù hợp

Gom đề xuất user đồng ý theo skill target:

- Đề xuất gắn `okr-init update-objective` → kích hoạt `okr-init` mode `update-objective` với danh sách thay đổi KR/period.
- Đề xuất gắn `okr-init update-resource` → kích hoạt `okr-init` mode `update-resource` với danh sách thay đổi PIC/khả dụng/tool.
- Đề xuất gắn `okr-plan update` → kích hoạt `okr-plan` mode `update` với danh sách thay đổi action/milestone.

Skill được delegate sẽ tự chạy phase confirm + ghi file (theo flow của riêng nó). Track CHỈ truyền context (lý do + giá trị mới), KHÔNG tự ghi SOT objective/plan.
```

new_string:

```
#### Bước 5: Delegate sang skill phù hợp

Gom đề xuất user đồng ý theo skill target. Mỗi delegate KÈM payload format:

```yaml
delegate_to: okr-init update-objective | okr-init update-resource | okr-plan update
context:
  changes:
    - field: <vd: KR2.target | A005.due_date | tool.X>
      from: <giá trị cũ>
      to: <giá trị mới>
    - field: ...
      from: ...
      to: ...
  reason: "<1-2 câu giải thích vì sao điều chỉnh, lấy từ Bước 2 root cause>"
  source_review: log/reviews/YYYY-MM-DD.md
  pre_confirmed: true
```

Field meanings:

| Field | Bắt buộc | Mô tả |
|-------|----------|-------|
| `delegate_to` | có | Skill + mode đích. 1 trong 3 giá trị. |
| `context.changes[]` | có | Danh sách thay đổi (field, from, to). 1 payload có thể chứa nhiều changes nếu cùng skill đích. |
| `context.reason` | có | Lý do GỐC từ Bước 2 root cause. Track viết vào, init/plan đọc + hiển thị trong CONFIRM diff. |
| `context.source_review` | có | Path file review log để init/plan trace lại. Mặc định `log/reviews/<today>.md`. |
| `context.pre_confirmed` | optional | `true` nếu user đã confirm tại Bước 4 ở track. Skill nhận có thể skip phase confirm riêng (xem T4 ở Đợt 4). Mặc định `false`. |

Ví dụ payload thực tế (KR2 giảm target do market shift):

```yaml
delegate_to: okr-init update-objective
context:
  changes:
    - field: KR2.target
      from: 50
      to: 35
  reason: "Market shift Q4 (từ root cause Bước 2): tăng trưởng ngành chậm 30%, target 50 không khả thi."
  source_review: log/reviews/2026-12-01.md
  pre_confirmed: true
```

Skill được delegate sẽ tự chạy phase confirm + ghi file (theo flow của riêng nó). CONFIRM phase BẮT BUỘC hiển thị `reason` cùng diff (xem `skills/okr-init/SKILL.md` Phase 6 update-objective + `skills/okr-plan/SKILL.md` Phase 4 update). Track CHỈ truyền context, KHÔNG tự ghi SOT objective/plan.
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "delegate_to: okr-init update-objective \| okr-init update-resource \| okr-plan update|context.reason|source_review|pre_confirmed" skills/okr-track/SKILL.md
```

Expected: ≥3 match (cho phép spaces tolerance).

Run:

```bash
grep -nE "Market shift Q4.*từ root cause Bước 2" skills/okr-track/SKILL.md
```

Expected: 1 match (ví dụ payload).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-track): structured delegate payload with reason (G9)

Trước đây track delegate sang init/plan chỉ truyền "changes",
mất reason gốc. Skill nhận hiển thị diff không có context lý do
→ user phải scroll log review để hiểu vì sao bị hỏi confirm.

Phase 4b Bước 5 thêm format payload:
- changes[] (field, from, to)
- reason (1-2 câu từ root cause Bước 2)
- source_review (path log review để trace)
- pre_confirmed (optional, foundation cho Đợt 4 T4)

Skill nhận BẮT BUỘC hiển thị reason trong CONFIRM (xem Task 9
+ 10 cho init + plan).

Refs: deep-insight/okr-skill-review-260510.md G9
EOF
)"
```

---

## Task 10: G9.c — `okr-plan/SKILL.md` Phase 4 CONFIRM diff hiển thị reason

> **Lưu ý thứ tự**: Task 10 chạy trước Task 9 (G9.b okr-init) vì Task 12 (M4) sẽ renumber phase trong mode update-objective khiến Task 9 phải edit ở vị trí mới. Plan chạy theo thứ tự enumerated nhưng task numbers giữ nguyên cho dễ tham chiếu review doc.

**Files:**

- Modify: `skills/okr-plan/SKILL.md:160-176` (Phase 4 CONFIRM diff trong Mode UPDATE)

**Mục tiêu:** Phase 4 CONFIRM diff hiển thị `reason` + `source_review` (nếu nhận được payload từ track delegate). Nếu nhận trực tiếp từ user (không qua track), reason để trống → không hiển thị.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '160,180p' skills/okr-plan/SKILL.md
```

Expected: thấy "### Phase 4: CONFIRM diff (BẮT BUỘC)" với code block "Thay đổi sắp áp dụng" + "Tác động" + "Xác nhận?" KHÔNG có row Lý do hoặc source_review.

- [ ] **Step 2: Thay Phase 4 (thêm reason display)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |
| Đổi PIC A007| An                 | Dũng                      |

Tác động
- A013 mới → resources.md cập nhật cột Actions của Marketing
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```
```

new_string:

```
### Phase 4: CONFIRM diff (BẮT BUỘC)

Nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:

```
Lý do điều chỉnh (từ track deep)
  Market shift Q4 (root cause Bước 2): tăng trưởng ngành chậm 30%,
  cần thêm marketing để đẩy KR2.
  Source: log/reviews/2026-12-01.md
```

Sau đó luôn hiển thị bảng diff:

```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |

Tác động
- A013 mới → 1 action mới gắn KR2
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```

Quy tắc reason display:
- Nếu `context.reason` rỗng hoặc không có (user vào trực tiếp `/okr plan update`) → KHÔNG render block "Lý do điều chỉnh".
- `source_review` luôn đi kèm reason (cùng block).
- Reason là plain text 1-3 câu, KHÔNG markdown đặc biệt (giữ readable trong terminal).
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "Lý do điều chỉnh \(từ track deep\)|context.reason|source_review|Source: log/reviews" skills/okr-plan/SKILL.md
```

Expected: ≥3 match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): show delegate reason in Phase 4 CONFIRM (G9)

Skill nhận delegate từ track deep → CONFIRM diff hiện tại chỉ
hiển thị changes, mất reason. User phải scroll log review để
hiểu vì sao bị hỏi confirm. Phase 4 thêm block "Lý do điều
chỉnh (từ track deep)" trước bảng diff (chỉ render khi
context.reason có giá trị). Source path đi kèm để trace.

Refs: deep-insight/okr-skill-review-260510.md G9
EOF
)"
```

---

## Task 12: M4 — `okr-init/SKILL.md` Mode UPDATE-OBJECTIVE thêm Impact Check phase

> **Thứ tự**: Task 12 chạy trước Task 9 (G9.b) vì Task 12 renumber phase trong mode update-objective. Sau Task 12, Phase 5 cũ (CONFIRM diff) thành Phase 6.

**Files:**

- Modify: `skills/okr-init/SKILL.md:240-273` (Phase 4 Re-validate SMART + Phase 5 CONFIRM diff + Phase 6 Ghi file)

**Mục tiêu:** Thêm Phase 5 mới "Impact Check" giữa Phase 4 SMART và Phase 6 CONFIRM (cũ là Phase 5). Quét `plan.md` + `actions/*.md` cho mỗi field thay đổi → liệt kê tác động cụ thể (vd: đổi `end_date` → actions có `due_date` sau end_date mới). Sau đó renumber Phase 5 → 6 và Phase 6 → 7.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '238,275p' skills/okr-init/SKILL.md
```

Expected: thấy "### Phase 4: Re-validate SMART (BẮT BUỘC trước CONFIRM)" → "### Phase 5: CONFIRM diff (BẮT BUỘC)" → "### Phase 6: Ghi file" trong section "## Mode UPDATE-OBJECTIVE".

- [ ] **Step 2: Thay block 3 phase (4 → 7) — chèn Phase 5 Impact Check + renumber**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
### Phase 4: Re-validate SMART (BẮT BUỘC trước CONFIRM)

Trước khi hiển thị bảng confirm, agent re-validate SMART cho mọi KR/KI có thay đổi (target, baseline, period). Tiêu chí xem `references/okr-guide.md`:

- **S**pecific: KR mô tả rõ chỉ số nào, sản phẩm/kênh nào.
- **M**easurable: có baseline + target số (Project) hoặc ngưỡng (Ongoing).
- **A**chievable: target stretch nhưng khả thi với capacity hiện tại.
- **R**elevant: KR/KI vẫn phục vụ Objective.
- **T**ime-bound: có deadline (Project) hoặc review_cycle (Ongoing).

Nếu KR/KI thay đổi fail tiêu chí nào → liệt kê warning trong bảng confirm. KHÔNG block, user vẫn quyết.

### Phase 5: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |

Cảnh báo SMART (KR/KI thay đổi)
- KR2 (MRR 200M > 250M): ⚠️ Achievable? Tốc độ MRR hiện tại
  ~10M/tháng × 4 tháng = 40M, gap target mới 50M. Hơi stretch.
- End 2027-01-15: Time-bound OK, nhưng cross năm tài chính. Cân
  nhắc đặt thêm milestone Q4 closure trước.

Xác nhận? (y / sửa / huỷ / bỏ qua cảnh báo)
```

### Phase 6: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr` để check tác động sang plan."
```

new_string:

```
### Phase 4: Re-validate SMART (BẮT BUỘC trước CONFIRM)

Trước khi sang Impact Check, agent re-validate SMART cho mọi KR/KI có thay đổi (target, baseline, period). Tiêu chí xem `references/okr-guide.md`:

- **S**pecific: KR mô tả rõ chỉ số nào, sản phẩm/kênh nào.
- **M**easurable: có baseline + target số (Project) hoặc ngưỡng (Ongoing).
- **A**chievable: target stretch nhưng khả thi với capacity hiện tại.
- **R**elevant: KR/KI vẫn phục vụ Objective.
- **T**ime-bound: có deadline (Project) hoặc review_cycle (Ongoing).

Nếu KR/KI thay đổi fail tiêu chí nào → ghi nhớ để liệt kê trong bảng confirm Phase 6. KHÔNG block, user vẫn quyết.

### Phase 5: Impact Check (BẮT BUỘC trước CONFIRM)

Quét `.okr/plan.md` (frontmatter milestones + body Practices nếu Ongoing) và `.okr/actions/*.md` (frontmatter; **không đệ quy** archive). Với mỗi field thay đổi trong Phase 3, xác định tác động cụ thể:

| Field thay đổi | Cách quét tác động |
|----------------|--------------------|
| `end_date` rút ngắn | Liệt kê actions có `due_date > end_date_mới`. Liệt kê milestones có `target_date > end_date_mới`. |
| `end_date` extend | Note "Period mở rộng N ngày, có thể thêm milestone/actions mới qua `/okr plan update`". |
| `start_date` lùi | Liệt kê actions/milestones có ngày sớm hơn `start_date_mới`. |
| `KR<N>.target` đổi | Liệt kê actions có `key_result: KR<N>` (count + IDs). User cần cân nhắc thêm/bỏ actions nếu target thay đổi nhiều (>20% so với cũ). |
| `KR<N>.baseline` đổi | Cảnh báo recompute % progress. Action không bị ảnh hưởng trực tiếp. |
| Xoá KR<N> | Liệt kê actions có `key_result: KR<N>`. Đề xuất reassign sang KR khác hoặc xoá. |
| Đổi `objective` text / WHY | Note "Cân nhắc review lại practices và actions xem có còn align không". |
| Đổi `review_cycle` (Ongoing) | Note "Track sẽ chuyển sang chu kỳ mới từ lần check-in tới". |
| Đổi `status` | Nếu chuyển sang `completed/cancelled/archived` → liệt kê actions chưa done + đề xuất archive cùng. |

Hiển thị block tác động:

```
Tác động sang plan + actions
- end_date 2026-12-31 → 2026-11-30 (rút ngắn 31 ngày):
  ⚠️ 3 actions due_date sau 2026-11-30: A007 (2026-12-15), A009 (2026-12-20), A011 (2026-12-28)
  ⚠️ 1 milestone target_date sau: M3 (2026-12-15)
  Đề xuất: chạy `/okr plan update` để dời actions/milestone, hoặc giảm scope.
- KR2.target 200M → 250M (+25%):
  3 actions thuộc KR2: A005, A006, A013. Cân nhắc thêm 1-2 actions marketing.
```

KHÔNG block. User vẫn có thể confirm và xử lý plan sau qua `/okr plan update`. Mục tiêu là surface tác động để user không quên.

### Phase 6: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |

Cảnh báo SMART (KR/KI thay đổi)
- KR2 (MRR 200M > 250M): ⚠️ Achievable? Tốc độ MRR hiện tại
  ~10M/tháng × 4 tháng = 40M, gap target mới 50M. Hơi stretch.
- End 2027-01-15: Time-bound OK, nhưng cross năm tài chính. Cân
  nhắc đặt thêm milestone Q4 closure trước.

Tác động sang plan + actions (từ Phase 5)
- KR2.target 200M → 250M: 3 actions thuộc KR2. Cân nhắc thêm
  1-2 actions marketing qua `/okr plan update` sau.

Xác nhận? (y / sửa / huỷ / bỏ qua cảnh báo)
```

### Phase 7: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr plan update` để áp dụng tác động sang actions/milestones (Phase 5 Impact Check đã liệt kê)."
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "^### Phase 4: Re-validate SMART|^### Phase 5: Impact Check|^### Phase 6: CONFIRM diff|^### Phase 7: Ghi file" skills/okr-init/SKILL.md
```

Expected: 4 match (4 phase liền nhau trong mode UPDATE-OBJECTIVE).

Run:

```bash
awk '/^## Mode UPDATE-OBJECTIVE/,/^## Mode UPDATE-RESOURCE/' skills/okr-init/SKILL.md | grep -E "^### "
```

Expected: thứ tự `Phase 1` → `Phase 2` → `Phase 3` → `Phase 4: Re-validate SMART` → `Phase 5: Impact Check` → `Phase 6: CONFIRM diff` → `Phase 7: Ghi file`.

Run:

```bash
grep -nE "Tác động sang plan \+ actions|end_date 2026-12-31 → 2026-11-30 \(rút ngắn 31 ngày\)|Cách quét tác động" skills/okr-init/SKILL.md
```

Expected: ≥3 match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): add Impact Check phase to update-objective (M4)

Mode update-objective thay đổi field như end_date, KR.target có
thể làm actions/milestones không khớp. Hiện tại không có check,
user phải tự nhớ chạy /okr plan update sau. Thêm Phase 5 mới
"Impact Check": quét plan.md + actions/*.md cho từng field thay
đổi, liệt kê tác động cụ thể (actions due sau end_date mới,
milestone bị ảnh hưởng, count actions thuộc KR đổi target).
Phase 5 cũ (CONFIRM) → Phase 6, Phase 6 cũ (Ghi file) → Phase 7.
Phase 6 CONFIRM mới gồm cả tác động từ Phase 5 trong block.

Không block, user vẫn confirm. Mục tiêu surface để không quên.

Refs: deep-insight/okr-skill-review-260510.md M4
EOF
)"
```

---

## Task 9: G9.b — `okr-init/SKILL.md` Phase 6 CONFIRM diff hiển thị reason

> **Thứ tự**: Chạy SAU Task 12 (M4) vì Task 12 renumber CONFIRM thành Phase 6. Trước Task 12, CONFIRM là Phase 5.

**Files:**

- Modify: `skills/okr-init/SKILL.md` Phase 6 CONFIRM diff (sau Task 12 đã renumber)

**Mục tiêu:** Phase 6 CONFIRM diff hiển thị `reason` + `source_review` (nếu nhận được từ track delegate). Nếu user vào trực tiếp `/okr init update-objective` → reason rỗng → không render.

- [ ] **Step 1: Verify state hiện tại (sau Task 12)**

Run:

```bash
sed -n '/^### Phase 6: CONFIRM diff/,/^### Phase 7: Ghi file/p' skills/okr-init/SKILL.md
```

Expected: block CONFIRM diff (Phase 6, từ Task 12) gồm bảng "Thay đổi sắp áp dụng" + "Cảnh báo SMART" + "Tác động sang plan + actions" + "Xác nhận?". KHÔNG có "Lý do điều chỉnh" hoặc "context.reason".

- [ ] **Step 2: Thêm block "Lý do điều chỉnh" trước bảng "Thay đổi sắp áp dụng"**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
### Phase 6: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |
```

new_string:

```
### Phase 6: CONFIRM diff (BẮT BUỘC)

Nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:

```
Lý do điều chỉnh (từ track deep)
  Market shift Q4 (root cause Bước 2): tăng trưởng ngành chậm 30%,
  target 50 không khả thi.
  Source: log/reviews/2026-12-01.md
```

Quy tắc:
- Reason rỗng hoặc không có → KHÔNG render block.
- Plain text 1-3 câu, không markdown.
- `source_review` cùng block với reason.

Sau đó luôn hiển thị bảng:

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "Lý do điều chỉnh \(từ track deep\)|Market shift Q4.*target 50 không khả thi|Source: log/reviews/2026-12-01.md" skills/okr-init/SKILL.md
```

Expected: ≥2 match (block reason ở Phase 6 update-objective).

Run:

```bash
awk '/^### Phase 6: CONFIRM diff/,/^### Phase 7: Ghi file/' skills/okr-init/SKILL.md | grep -cE "Lý do điều chỉnh|Thay đổi sắp áp dụng|Cảnh báo SMART|Tác động sang plan"
```

Expected: 4 (4 block trong Phase 6).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): show delegate reason in update-objective CONFIRM (G9)

Đồng bộ với okr-plan: Phase 6 update-objective CONFIRM hiển thị
"Lý do điều chỉnh (từ track deep)" trước bảng diff khi nhận
context.reason từ track. User vào trực tiếp /okr init
update-objective → reason rỗng → không render. Source path đi
kèm để trace log review.

Refs: deep-insight/okr-skill-review-260510.md G9
EOF
)"
```

---

## Task 11: G11 — `okr-init/SKILL.md` Mode NEW Phase 0 đọc inbox + Phase 7 suggest map

**Files:**

- Modify: `skills/okr-init/SKILL.md` (sau "## Mode NEW: tạo `.okr/` từ đầu" header, thêm Phase 0 trước Phase 1)
- Modify: `skills/okr-init/SKILL.md` Phase 7 (Ghi file) — thêm Phase 8 mới: "Post-init: gợi ý map related_kr cho inbox"

**Mục tiêu:** Khi user chạy `okr-init new`, nếu `.okr/inbox/` đã có items (user dùng `okr-capture` trước khi có objective), Phase 0 đọc inbox làm CONTEXT cho đề xuất KR/actions ban đầu. Sau khi tạo xong objective + resources (Phase 7), Phase 8 mới gợi ý map `related_kr` cho từng item inbox.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
sed -n '49,55p' skills/okr-init/SKILL.md
```

Expected: thấy "## Mode NEW: tạo `.okr/` từ đầu" → "### Phase 1: Loại mục tiêu". KHÔNG có Phase 0.

Run:

```bash
sed -n '212,220p' skills/okr-init/SKILL.md
```

Expected: thấy "### Phase 7: Ghi file" với 4 step (tạo `.okr/`, ghi objective.md, ghi resources.md, hiển thị thông báo). KHÔNG có Phase 8.

- [ ] **Step 2: Thêm Phase 0 "Đọc inbox làm context (nếu có)" trước Phase 1**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
## Mode NEW: tạo `.okr/` từ đầu

### Phase 1: Loại mục tiêu
```

new_string:

```
## Mode NEW: tạo `.okr/` từ đầu

### Phase 0: Đọc inbox làm context (nếu có)

Trước khi hỏi loại mục tiêu, kiểm tra `.okr/inbox/*.md`:

- Nếu thư mục `.okr/inbox/` chưa có hoặc rỗng → skip Phase 0, sang Phase 1.
- Nếu có items → đọc tất cả file (frontmatter + body), nhóm theo `type`:
  - `action` items: gợi ý cho user đây có thể là việc cần làm cho objective sắp tạo.
  - `resource` items: tool/tài liệu sẵn có, dùng làm input cho Phase 5 Resource.
  - `blocker` items: rủi ro/thiếu hụt, dùng làm input cho Phase 5 mục Rủi ro.
  - `thought` items: ý tưởng nền, có thể chuyển thành KR hoặc actions.

Hiển thị block context:

```
Inbox sẵn có (5 items, captured trước khi tạo objective)
- 2 action: "Viết blog post về X", "Setup analytics tracking"
- 1 resource: "Library Y hỗ trợ chart"
- 1 blocker: "Server staging không stable"
- 1 thought: "Có thể test pricing model 2 tier?"

Mình sẽ dùng các items này làm context khi đề xuất KR (Phase 3),
actions (qua /okr plan sau), và resources (Phase 5). Sau khi
objective tạo xong, mình sẽ hỏi map từng item vào KR phù hợp.
Tiếp tục Phase 1? (y/sửa context/skip context)
```

User trả lời:
- `y` → đi tiếp Phase 1, agent giữ items làm context nội bộ.
- `sửa context` → user bổ sung/loại trừ items nào.
- `skip context` → đi tiếp Phase 1 nhưng không dùng items (vẫn giữ inbox cho Phase 8).

### Phase 1: Loại mục tiêu
```

- [ ] **Step 3: Renumber Phase 7 thành Phase 7 (giữ) + thêm Phase 8 mới sau Phase 7**

Edit `skills/okr-init/SKILL.md`:

old_string:

```
### Phase 7: Ghi file

1. Tạo `.okr/`.
2. Ghi `.okr/objective.md` theo schema.
3. Ghi `.okr/resources.md` theo schema (giữ section header dù rỗng).
4. Hiển thị: "Đã tạo. Chạy `/okr` để lập plan."
```

new_string:

```
### Phase 7: Ghi file

1. Tạo `.okr/`.
2. Ghi `.okr/objective.md` theo schema.
3. Ghi `.okr/resources.md` theo schema (giữ section header dù rỗng).
4. Hiển thị: "Đã tạo objective + resources. Tiếp tục Phase 8 nếu có inbox sẵn, hoặc chạy `/okr` để lập plan."

### Phase 8: Post-init — gợi ý map inbox vào KR (chỉ chạy nếu Phase 0 đã đọc inbox)

Skip nếu Phase 0 trả về rỗng. Nếu có items:

Với mỗi inbox item có `related_kr: null`, agent đề xuất KR phù hợp dựa trên KR vừa tạo + nội dung item:

```
Map inbox items vào KR (5 items pending)
| # | Type     | Title                          | Đề xuất related_kr | Lý do                          |
|---|----------|--------------------------------|--------------------|--------------------------------|
| 1 | action   | Viết blog post về X            | KR1                | KR1 về content marketing       |
| 2 | action   | Setup analytics tracking       | KR2                | KR2 cần đo conversion          |
| 3 | resource | Library Y hỗ trợ chart         | (không cần map)    | Resource, không link KR        |
| 4 | blocker  | Server staging không stable    | KR2                | Blocker A007 thuộc KR2         |
| 5 | thought  | Có thể test pricing 2 tier?    | KR2                | Liên quan revenue              |

Apply gợi ý? (vd: "1,2,4,5 apply / 3 skip" / all / sửa N: KR<X> / huỷ)
```

User trả lời:
- `<N> apply`: ghi `related_kr: KR<X>` vào frontmatter inbox item.
- `<N> skip`: giữ `related_kr: null`.
- `all`: apply hết.
- `sửa N: KR<X>`: override gợi ý.
- `huỷ`: bỏ hết, không apply.

Sau khi apply: hiển thị "Đã map N items. Chạy `/okr` để lập plan và xử lý inbox khi track."
```

- [ ] **Step 4: Verify**

Run:

```bash
grep -nE "^### Phase 0: Đọc inbox làm context|^### Phase 8: Post-init|Inbox sẵn có \(5 items|Apply gợi ý\?" skills/okr-init/SKILL.md
```

Expected: ≥3 match.

Run:

```bash
awk '/^## Mode NEW/,/^## Mode UPDATE-OBJECTIVE/' skills/okr-init/SKILL.md | grep -E "^### Phase "
```

Expected: thứ tự `Phase 0` → `Phase 1` → `Phase 2` → ... → `Phase 7` → `Phase 8`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): scan inbox in NEW mode + post-init map (G11)

User capture items trước khi có objective → inbox có items với
related_kr: null. Init mode new bỏ qua, sau đó user phải tự nhớ
quay lại map.

Mode NEW thêm 2 phase:
- Phase 0 "Đọc inbox làm context": trước Phase 1, scan inbox,
  nhóm theo type (action/resource/blocker/thought), hiển thị
  cho user quyết dùng làm context hay skip.
- Phase 8 "Post-init suggest map": sau Phase 7 ghi file, gợi
  ý related_kr cho từng item inbox dựa trên KR vừa tạo. User
  apply / skip / sửa từng cái.

Refs: deep-insight/okr-skill-review-260510.md G11
EOF
)"
```

---

## Task 13: PIC sweep — `okr/SKILL.md` (5 mention)

**Files:**

- Modify: `skills/okr/SKILL.md` (description line 3, status table line 31, Resource line 61, routing tables line 72 + 83)

**Mục tiêu:** Bỏ mọi mention "PIC" khỏi orchestrator (description, phân vai SOT table, status dashboard, routing keywords). Đồng thời bỏ "nhân sự" team-bias keyword. Đổi sang "capacity / skills / Solo Profile".

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -nE "PIC|nhân sự" skills/okr/SKILL.md
```

Expected: ≥5 match (các line 3, 31, 61, 72, 83 + có thể nhân sự thêm).

- [ ] **Step 2: Sửa description (line 3)**

Edit `skills/okr/SKILL.md`:

old_string:

```
description: "Entry point chính cho mọi tương tác OKR. LUÔN dùng skill này đầu tiên khi user nhắc đến mục tiêu, dự án, OKR, tracking, kế hoạch, tài nguyên, PIC, inbox, capture, ghi nhanh, hoặc gọi `/okr`. Skill tự đánh giá trạng thái `.okr/` hiện tại và chủ động kích hoạt skill phù hợp (init/plan/track/capture) với context user cung cấp. User KHÔNG cần biết tên skill con, chỉ làm việc với `/okr`."
```

new_string:

```
description: "Entry point chính cho mọi tương tác OKR. LUÔN dùng skill này đầu tiên khi user nhắc đến mục tiêu, dự án, OKR, tracking, kế hoạch, tài nguyên, capacity, skill, inbox, capture, ghi nhanh, hoặc gọi `/okr`. Skill tự đánh giá trạng thái `.okr/` hiện tại và chủ động kích hoạt skill phù hợp (init/plan/track/capture) với context user cung cấp. User KHÔNG cần biết tên skill con, chỉ làm việc với `/okr`."
```

- [ ] **Step 3: Sửa Phân vai SOT table row Resource (line 31)**

Edit `skills/okr/SKILL.md`:

old_string:

```
| Người, tool, ngân sách, PIC, khả dụng | `okr-init` `update-resource` |
```

new_string:

```
| Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
```

- [ ] **Step 4: Sửa Resource line trong status dashboard (line 61)**

Edit `skills/okr/SKILL.md`:

old_string:

```
  Resource  : [đã có / thiếu PIC: tên action]
```

new_string:

```
  Resource  : [đã có (capacity Xh/tuần, N skills, M tool) / thiếu Solo Profile / thiếu capacity]
```

- [ ] **Step 5: Sửa routing table (line 72) và keyword routing (line 83)**

Edit `skills/okr/SKILL.md`:

old_string:

```
| User nhắc người/tài nguyên/PIC/ngân sách/tool | `okr-init` mode `update-resource` |
```

new_string:

```
| User nhắc capacity/skill/tài nguyên/ngân sách/tool | `okr-init` mode `update-resource` |
```

old_string:

```
- "tài liệu / tài nguyên / nhân sự / công cụ / PIC / ngân sách" → `okr-init` `update-resource`
```

new_string:

```
- "tài liệu / tài nguyên / capacity / skill / công cụ / ngân sách / Solo Profile" → `okr-init` `update-resource`
```

- [ ] **Step 6: Verify**

Run:

```bash
grep -nE "PIC|nhân sự" skills/okr/SKILL.md
```

Expected: KHÔNG có match.

Run:

```bash
grep -nE "capacity, skills|Solo Profile|capacity Xh/tuần" skills/okr/SKILL.md
```

Expected: ≥3 match.

- [ ] **Step 7: Commit**

```bash
git add skills/okr/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr): sweep PIC mentions from orchestrator (Wave 3 cleanup)

Đợt 2 đã solo-only nhưng okr/SKILL.md vẫn còn 5 mention "PIC"
(description, phân vai SOT, Resource line, 2 routing). Sweep:
- description: PIC → capacity, skill
- Phân vai SOT row Resource: "Người, tool, ngân sách, PIC, khả
  dụng" → "Solo Profile (capacity, skills), tool, ngân sách"
- Status dashboard Resource line: "thiếu PIC: tên action" →
  "thiếu Solo Profile / thiếu capacity"
- Routing table + keyword: bỏ "PIC", "nhân sự"; thêm
  "capacity", "skill", "Solo Profile"

Solo user keyword routing rõ ràng hơn, không còn UI inconsistency.

Refs: deep-insight/okr-skill-review-260510.md (PIC sweep defer Đợt 3)
EOF
)"
```

---

## Task 14: PIC sweep — `okr-track/references/data-format.md:65` (1 line)

**Files:**

- Modify: `skills/okr-track/references/data-format.md:65` (Resource row trong "Structure fields" table)

**Mục tiêu:** Sửa 1 dòng còn list "Người, tool, ngân sách, PIC, khả dụng" cho khớp với phân vai SOT đã sweep ở Task 13.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -n "PIC, khả dụng\|người, tool, ngân sách, PIC" skills/okr-track/references/data-format.md
```

Expected: 1 match line ~65.

- [ ] **Step 2: Sửa row Resource**

Edit `skills/okr-track/references/data-format.md`:

old_string:

```
| Resource: người, tool, ngân sách, PIC, khả dụng | `okr-init` `update-resource` |
```

new_string:

```
| Resource: Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -nE "PIC|khả dụng" skills/okr-track/references/data-format.md
```

Expected: KHÔNG có match.

Run:

```bash
grep -n "Solo Profile (capacity, skills), tool, ngân sách" skills/okr-track/references/data-format.md
```

Expected: 1 match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-track/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-track): sweep PIC from data-format Resource row (Wave 3 cleanup)

Structure fields table còn 1 row list "Người, tool, ngân sách,
PIC, khả dụng" — sót sau Đợt 2. Đồng bộ với okr/SKILL.md (Task
13): "Solo Profile (capacity, skills), tool, ngân sách".

Refs: deep-insight/okr-skill-review-260510.md (PIC sweep defer Đợt 3)
EOF
)"
```

---

## Task 15: PIC sweep — `okr-plan/SKILL.md` (17 mention, task lớn)

**Files:**

- Modify: `skills/okr-plan/SKILL.md` (rải khắp file: prerequisite, Quality Gate, Phase 1-5 NEW + UPDATE)

**Mục tiêu:** Sweep tất cả mention "PIC" team-style trong `okr-plan/SKILL.md`. Sau Đợt 2, schema đã set `pic: self` mặc định và bỏ field `verifier`, nhưng SKILL.md flow vẫn còn nhiều chỗ hỏi PIC, gán PIC, đổi PIC hàng loạt. Sweep theo phạm vi:

1. Prerequisite + Quality Gate: bỏ "assign PIC", "PIC trống", "PIC 50% available".
2. Phase 1 NEW: bỏ "PIC available" trong context.
3. Phase 2 NEW step 5: bỏ "Đề xuất PIC từ resources.md".
4. Deepening Techniques table: bỏ row PIC.
5. Phase 3 NEW CONFIRM: bỏ cột PIC khỏi bảng actions, bỏ "PIC tải nhiều nhất", "actions chưa có PIC (TBD)".
6. Phase 4 NEW Ghi file: bỏ step 5 sync `resources.md` cột Actions.
7. Mode UPDATE Phase 1 dashboard: KHÔNG có "PIC" trực tiếp, kiểm tra.
8. Mode UPDATE Phase 2 menu: bỏ item 6 "Đổi PIC hàng loạt", sửa item 2 "Sửa action (..., PIC, ...)" → bỏ PIC.
9. Mode UPDATE Phase 4 CONFIRM: bỏ row "Đổi PIC A007", bỏ "Tác động" reference resources.md mapping.
10. Mode UPDATE Phase 5 áp dụng: bỏ step 3 sync resources.md mapping.
11. Quy tắc cuối: giữ "PIC (default `self` cho solo)" — đã đúng. Giữ "DoD, output, PIC, effort" trong line 206 (Ongoing) → có thể giữ vì pic=self valid.

- [ ] **Step 1: Verify state hiện tại**

Run:

```bash
grep -cE "PIC|pic" skills/okr-plan/SKILL.md
```

Expected: 17-18 (mention).

Run:

```bash
grep -nE "PIC|pic" skills/okr-plan/SKILL.md
```

Note: ghi nhớ tất cả line. Plan này sẽ sửa từng vùng tuần tự.

- [ ] **Step 2: Sửa prerequisite (line 13)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
- `.okr/resources.md` tồn tại. Thiếu → hỏi user có bổ sung trước (`/okr` → `okr-init` mode `update-resource`) hay tiếp tục với PIC trống.
```

new_string:

```
- `.okr/resources.md` tồn tại. Thiếu → hỏi user có bổ sung trước (`/okr` → `okr-init` mode `update-resource`) hay tiếp tục với Solo Profile mặc định (capacity 0h/tuần, skills rỗng — sẽ cảnh báo cross-check fit ở Phase 3 confirm).
```

- [ ] **Step 3: Sửa Quality Gate (line 17 + 21)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
Mỗi khi user trả lời (chọn initiative, confirm action, assign PIC...), agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành action có deliverable đo được không? Vd: "Nghiên cứu thêm" → FAIL (output là gì? đo bằng gì?).
2. **Giả định ẩn?** User bỏ qua constraint quan trọng không? Vd: thêm 5 actions mới nhưng không nói ai làm.
3. **Mâu thuẫn?** Có xung đột với resource/timeline đã có không? Vd: PIC 50% available nhưng gán 6 actions cùng deadline.
```

new_string:

```
Mỗi khi user trả lời (chọn initiative, confirm action, ước effort...), agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành action có deliverable đo được không? Vd: "Nghiên cứu thêm" → FAIL (output là gì? đo bằng gì?).
2. **Giả định ẩn?** User bỏ qua constraint quan trọng không? Vd: thêm 5 actions mới nhưng không nói effort tổng bao nhiêu giờ.
3. **Mâu thuẫn?** Có xung đột với capacity/timeline đã có không? Vd: capacity 10h/tuần nhưng gán 6 actions effort `m` (1-2 ngày mỗi cái) cùng deadline tuần sau.
```

- [ ] **Step 4: Sửa Phase 1 + Phase 2 NEW (line 47 + 56 + 64)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
- `.okr/resources.md`: PIC available + công cụ + ngân sách.
```

new_string:

```
- `.okr/resources.md`: Solo Profile (capacity h/tuần, skills) + công cụ + ngân sách.
```

old_string:

```
Với mỗi KR:
1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Đề xuất PIC từ resources.md cho mỗi action.
```

new_string:

```
Với mỗi KR:
1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Mặc định mỗi action `pic: self`. Cảnh báo nếu action cần skill chưa có trong Solo Profile (đề xuất thêm action học/outsource trước).
```

old_string:

```
| **PIC** | Challenge khả dụng thực tế | "Bình 50% available, đang có 3 actions. Thêm action này thì Bình có kịp không?" |
```

new_string:

```
| **Skill match** | Challenge action có skill chưa có | "Action 'Design Figma' nhưng Solo Profile chưa có skill design. Có học trước, outsource, hay đổi approach?" |
```

- [ ] **Step 5: Sửa Phase 3 NEW CONFIRM (line 83-97)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
Actions (12 total)
| ID    | Title              | Milestone | PIC  | Deps     | Output            |
|-------|--------------------|-----------|------|----------|-------------------|
| A001  | Khảo sát user      | M1        | An   | -        | survey-report.md  |
| A002  | Phân tích đối thủ  | M1        | Bình | -        | competitor.xlsx   |
| A003  | Spec MVP           | M2        | An   | A001,A002| spec.md           |

Đánh giá nhanh
  Resource fit:
  - Tổng capacity: [X] person-months. Tổng actions: [Y]. Fit: [✅/⚠️]
  - PIC tải nhiều nhất: [tên], [N] actions, [%] available
  Timeline:
  - Critical path: [A001 → A003 → A007]. Độ dài: [X tuần]. Buffer: [Y tuần]
  Rủi ro:
  - [vd: A005 có dependency ngoài team, chưa có backup plan]
  - [vd: ⚠️ 3 actions chưa có PIC (TBD)]
```

new_string:

```
Actions (12 total, mọi action `pic: self`)
| ID    | Title              | Milestone | Effort | Deps      | Output            |
|-------|--------------------|-----------|--------|-----------|-------------------|
| A001  | Khảo sát user      | M1        | m      | -         | survey-report.md  |
| A002  | Phân tích đối thủ  | M1        | l      | -         | competitor.xlsx   |
| A003  | Spec MVP           | M2        | l      | A001,A002 | spec.md           |

Đánh giá nhanh
  Capacity fit:
  - Tổng effort ước tính: [X] giờ. Capacity: [10h/tuần × N tuần] = [Y] giờ. Fit: [✅/⚠️]
  - Tuần dồn việc nhất: [tuần W], [N] actions cùng deadline.
  Timeline:
  - Critical path: [A001 → A003 → A007]. Độ dài: [X tuần]. Buffer: [Y tuần]
  Rủi ro:
  - [vd: A005 dependency ngoài (chờ stakeholder), chưa có backup plan]
  - [vd: ⚠️ 2 actions cần skill chưa có trong Solo Profile (Figma, SQL)]
```

- [ ] **Step 6: Sửa Phase 3 đánh giá quy tắc (line 102-106)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
Quy tắc đánh giá:
- **Resource fit**: tính tổng capacity vs. số actions. PIC nào gánh nhiều nhất?
- **Timeline**: xác định critical path dài nhất, còn buffer không?
- **Rủi ro**: liệt kê field TBD, dependency ngoài, bottleneck tiềm ẩn.
- Nếu có rủi ro nghiêm trọng → đề xuất giải pháp cụ thể (thêm người, dời deadline, cắt scope).
```

new_string:

```
Quy tắc đánh giá:
- **Capacity fit**: tính tổng effort ước tính (giờ) vs. capacity còn lại. Tuần nào dồn việc nhất?
- **Timeline**: xác định critical path dài nhất, còn buffer không?
- **Rủi ro**: liệt kê field TBD, dependency ngoài, skill chưa có trong Solo Profile.
- Nếu có rủi ro nghiêm trọng → đề xuất giải pháp cụ thể (cắt scope, dời deadline, học skill trước, outsource).
```

- [ ] **Step 7: Sửa Phase 4 NEW Ghi file (line 110-115)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`.
4. Cập nhật counters trong `plan.md`.
5. Cập nhật cột `Actions` trong `resources.md` (mapping PIC → action IDs) + `last_updated`.
```

new_string:

```
### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`. Mọi action `pic: self`.
4. Cập nhật counters trong `plan.md`.
```

- [ ] **Step 8: Sửa Mode UPDATE Phase 2 menu (line 150 + 154)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
Menu nếu user vào trực tiếp:
1. Thêm action mới
2. Sửa action (title, PIC, due_date, status, deps, deliverable)
3. Xoá action
4. Dời deadline milestone
5. Thêm/xoá milestone
6. Đổi PIC hàng loạt
```

new_string:

```
Menu nếu user vào trực tiếp:
1. Thêm action mới
2. Sửa action (title, due_date, status, deps, deliverable, effort, priority)
3. Xoá action
4. Dời deadline milestone
5. Thêm/xoá milestone
```

- [ ] **Step 9: Sửa Mode UPDATE Phase 4 CONFIRM (line 162-176)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |

Tác động
- A013 mới → 1 action mới gắn KR2
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```
```

new_string:

```
```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |

Tác động
- A013 mới → 1 action mới gắn KR2 (effort `m`, +8h capacity tuần W47)
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```
```

(Note: nội dung cốt lõi không đổi vì block đã không còn row "Đổi PIC A007" sau khi Task 10 chỉnh — verify lại bằng grep.)

- [ ] **Step 10: Sửa Mode UPDATE Phase 5 (line 178-183)**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`.
3. Sync `resources.md` (cột Actions của PIC) + `last_updated`.
4. Đề xuất chạy `okr-track` mode `light` để confirm state mới.
```

new_string:

```
### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`. Mọi action mới `pic: self`.
3. Đề xuất chạy `okr-track` mode `light` để confirm state mới.
```

- [ ] **Step 11: Sửa line 206 — Quy tắc Ongoing còn mention "PIC" in hoa**

Edit `skills/okr-plan/SKILL.md`:

old_string:

```
- Ongoing type: plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI (vd: "Mua đồ tập gym", "Đặt lịch khám sức khoẻ"). Actions này vẫn tuân quy tắc action bình thường (DoD, output, PIC, effort). Milestones không bắt buộc với Ongoing.
```

new_string:

```
- Ongoing type: plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI (vd: "Mua đồ tập gym", "Đặt lịch khám sức khoẻ"). Actions này vẫn tuân quy tắc action bình thường (DoD, output, effort, `pic: self`). Milestones không bắt buộc với Ongoing.
```

- [ ] **Step 12: Final verify**

Run:

```bash
grep -cE "PIC" skills/okr-plan/SKILL.md
```

Expected: 0.

Run:

```bash
grep -nE "pic: self|Solo Profile|Capacity fit|Skill match" skills/okr-plan/SKILL.md
```

Expected: ≥4 match.

- [ ] **Step 13: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): sweep 17 PIC mentions (Wave 3 cleanup)

Đợt 2 đã set pic=self mặc định nhưng SKILL.md flow vẫn còn 17
mention "PIC" team-style (assign PIC, đổi PIC hàng loạt, PIC tải
nhiều nhất, etc.). Sweep theo phạm vi:
- Prerequisite: PIC trống → Solo Profile mặc định
- Quality Gate: assign PIC → ước effort; PIC 50% → capacity 10h
- Phase 1 NEW: PIC available → Solo Profile (capacity, skills)
- Phase 2 NEW step 5: đề xuất PIC → mặc định pic=self + cảnh báo
  skill match
- Deepening Techniques: row PIC → row Skill match
- Phase 3 NEW CONFIRM: bỏ cột PIC khỏi bảng actions, đổi
  Resource fit → Capacity fit (giờ thay vì person-months), bỏ
  PIC tải nhiều nhất, bỏ "actions chưa có PIC (TBD)"
- Phase 4 NEW Ghi file: bỏ step 5 sync resources.md mapping PIC,
  thêm note "mọi action pic: self"
- Mode UPDATE Phase 2 menu: bỏ item 6 "Đổi PIC hàng loạt", item
  2 bỏ "PIC" khỏi sửa action
- Mode UPDATE Phase 4 CONFIRM: bỏ row "Đổi PIC", thay tác động
  resources.md mapping bằng capacity tuần
- Mode UPDATE Phase 5: bỏ step 3 sync resources.md cột Actions
- Quy tắc Ongoing: "DoD, output, PIC, effort" → "DoD, output,
  effort, pic: self"

okr-plan flow nhất quán solo-only, không còn UI hỏi PIC.

Refs: deep-insight/okr-skill-review-260510.md (PIC sweep defer Đợt 3)
EOF
)"
```

---

## Task 16: Final cross-check toàn Đợt 3

**Mục tiêu:** Quét lại toàn repo đảm bảo: (1) effort xl checkpoint rule có ở task-format + action-guide, (2) practice streak update có ở okr-track Phase 4a, (3) period overdue có ở dashboard + metrics, (4) KR↔Action sub-line có trong Phase 2, (5) inbox aging compute + Phase 5 cảnh báo, (6) delegate payload reason ở 3 file (track + init + plan), (7) Phase 0 + Phase 8 init NEW, (8) Phase 5 Impact Check ở update-objective, (9) PIC sweep sạch ở 3 file đã enumerate. Cuối cùng verify ~15 commit Wave 3 nằm trên branch.

- [ ] **Step 1: Quét M6/K4 effort xl checkpoints**

Run:

```bash
grep -rnE "BẮT BUỘC.*Checkpoints|Quy tắc cho effort = xl" skills/okr-plan/references/
```

Expected: ≥3 match (task-format.md template + table + action-guide.md section).

- [ ] **Step 2: Quét G2/M9 practice streak update**

Run:

```bash
grep -nE "Update practice streak|streak \+= 1|P1 → KI1 healthy" skills/okr-track/SKILL.md
```

Expected: ≥3 match.

Run:

```bash
grep -n "TODO (wave 3" skills/okr-plan/references/data-format.md
```

Expected: KHÔNG có match.

- [ ] **Step 3: Quét G3 period overdue**

Run:

```bash
grep -nE "period_overdue_days|Period đã qua|Period Overdue \(Project type\)" skills/okr-track/SKILL.md skills/okr-track/references/metrics.md
```

Expected: ≥3 match.

- [ ] **Step 4: Quét G4 KR↔Action sub-line**

Run:

```bash
grep -nE "Active: A001 \(doing\)|Actions group by KR" skills/okr-track/SKILL.md
```

Expected: ≥2 match.

- [ ] **Step 5: Quét G5 inbox aging**

Run:

```bash
grep -nE "## Inbox Aging|staleness_days = today - captured_at|Bước 1.5: Cảnh báo stale items|Inbox cũ ≥30 ngày" skills/okr-capture/references/data-format.md skills/okr-track/SKILL.md
```

Expected: ≥4 match.

- [ ] **Step 6: Quét G9 delegate reason ở 3 file**

Run:

```bash
grep -nE "context.reason|Lý do điều chỉnh \(từ track deep\)|source_review" skills/okr-track/SKILL.md skills/okr-init/SKILL.md skills/okr-plan/SKILL.md
```

Expected: ≥3 match (mỗi file 1 mention trở lên).

- [ ] **Step 7: Quét G11 Phase 0 + Phase 8 init NEW**

Run:

```bash
grep -nE "^### Phase 0: Đọc inbox làm context|^### Phase 8: Post-init|Inbox sẵn có \(5 items|Apply gợi ý\?" skills/okr-init/SKILL.md
```

Expected: ≥3 match.

- [ ] **Step 8: Quét M4 Impact Check ở update-objective**

Run:

```bash
awk '/^## Mode UPDATE-OBJECTIVE/,/^## Mode UPDATE-RESOURCE/' skills/okr-init/SKILL.md | grep -E "^### Phase "
```

Expected: thứ tự `Phase 1` → `Phase 2` → `Phase 3` → `Phase 4: Re-validate SMART` → `Phase 5: Impact Check` → `Phase 6: CONFIRM diff` → `Phase 7: Ghi file` (7 phase).

Run:

```bash
grep -nE "^### Phase 5: Impact Check|Tác động sang plan \+ actions|Cách quét tác động" skills/okr-init/SKILL.md
```

Expected: ≥2 match.

- [ ] **Step 9: Quét PIC sweep sạch ở 3 file**

Run:

```bash
grep -cE "PIC" skills/okr/SKILL.md skills/okr-track/references/data-format.md skills/okr-plan/SKILL.md
```

Expected: 0 ở mỗi file.

Run:

```bash
grep -rnE "khả dụng %|Khả dụng %" skills/okr/SKILL.md skills/okr-track/references/data-format.md skills/okr-plan/SKILL.md
```

Expected: KHÔNG có match.

- [ ] **Step 10: Quét toàn skills/ còn rác legacy team không**

Run:

```bash
grep -rnE "person-months|FTE|Đổi PIC hàng loạt|Mapping PIC vào actions|Cùng PIC có ≥3 actions|PIC khả dụng <50%|nhân sự" skills/
```

Expected: KHÔNG có match. (Mention "Solo Profile" / "capacity" / "skills" được giữ.)

- [ ] **Step 11: Verify lịch sử git**

Run:

```bash
git log --oneline -15
```

Expected: 15 commit gần nhất chứa các tag (theo thứ tự task chạy):
- `(M6/K4)` × 2 (Task 1, 2)
- `(G2/M9)` (Task 3)
- `(G3)` (Task 4)
- `(G4)` (Task 5)
- `(G5)` × 2 (Task 6, 7)
- `(G9)` × 3 (Task 8, 10, 9 — track + plan + init)
- `(M4)` (Task 12)
- `(G11)` (Task 11)
- `(Wave 3 cleanup)` × 3 (Task 13, 14, 15 PIC sweep)

Tổng 15 commit. Commit thứ 16 trở về trước = commit cuối Đợt 2 (Task 12 final cross-check Wave 2 không tạo commit, vậy commit thứ 16 là Task 11 Đợt 2 K2 SMART re-validate).

- [ ] **Step 12: Status sạch**

Run:

```bash
git status
```

Expected: `nothing to commit, working tree clean` (ngoại trừ untracked files đã có trước plan: `deep-insight/`, `docs/okr-system-review.md`, file plan này, file plan Đợt 1+2).

---

## Tiêu chí hoàn thành Đợt 3

- [ ] 15 commit nhỏ trên branch hiện tại, mỗi commit reference review item.
- [ ] `task-format.md` body template có `## Checkpoints (BẮT BUỘC khi effort=xl, optional khác)`. Bảng Effort dòng `xl` có cột "Quy tắc đặc biệt" ghi BẮT BUỘC Checkpoints.
- [ ] `action-guide.md` có section "### Quy tắc cho effort = xl" với flow 3 bước (hỏi tách → giữ xl thì bắt buộc Checkpoints → từ chối thì quay lại tách).
- [ ] `okr-track/SKILL.md` Phase 4a Ongoing có step 4 "Update practice streak" hỏi y/n/skip cho mỗi practice, update `current_streak`.
- [ ] `okr-track/SKILL.md` Phase 2 Project có dashboard mẫu 2 (period đã qua) với block "⚠️ Period đã qua N ngày".
- [ ] `okr-track/SKILL.md` Phase 2 Project mỗi KR row trong dashboard có sub-line "Actions: ... | Active: A001 (doing), ...".
- [ ] `okr-track/SKILL.md` Phase 5 Bước 1.5 cảnh báo stale items >30 ngày, hỏi giữ/bỏ trước Bước 2.
- [ ] `okr-track/SKILL.md` Phase 4b Bước 5 có format payload YAML với `delegate_to`, `context.changes`, `context.reason`, `context.source_review`, `context.pre_confirmed`.
- [ ] `okr-init/SKILL.md` mode `update-objective` có 7 phase: 1 → 2 → 3 → 4 SMART → 5 Impact Check → 6 CONFIRM → 7 Ghi.
- [ ] `okr-init/SKILL.md` mode `new` có Phase 0 (đọc inbox làm context) và Phase 8 (post-init suggest map related_kr).
- [ ] `okr-init/SKILL.md` Phase 6 CONFIRM (update-objective) hiển thị "Lý do điều chỉnh (từ track deep)" khi nhận `context.reason`.
- [ ] `okr-plan/SKILL.md` Phase 4 CONFIRM (update) hiển thị "Lý do điều chỉnh (từ track deep)" khi nhận `context.reason`.
- [ ] `okr-capture/references/data-format.md` có section "## Inbox Aging" doc compute on-the-fly + 3 ngưỡng (≤7, 7-30, >30).
- [ ] `okr-track/references/metrics.md` có section "## Period Overdue (Project type)" với formula + bảng đề xuất hành động.
- [ ] `grep "PIC"` toàn `skills/okr/SKILL.md`, `skills/okr-track/references/data-format.md`, `skills/okr-plan/SKILL.md` ra rỗng.
- [ ] Không còn "person-months", "FTE", "Đổi PIC hàng loạt", "Mapping PIC vào actions", "Cùng PIC có ≥3 actions", "PIC khả dụng <50%", "nhân sự" toàn `skills/`.

## Sau Đợt 3

Mở plan Đợt 4 (Dọn thừa, ~6 items): T1 merge State+Keyword routing, T2 Quality Gate shared file (đã có pattern ở okr-init + okr-plan từ Đợt 2 M3), T3 status table single source ở CLAUDE.md, T4 gom confirm track deep + init/plan skip phase confirm khi `pre_confirmed: true` (foundation đã làm Task 8 Đợt 3 G9.a), T5 merge capture Phase 5, M5 capture đơn giản + validate ở track (foundation Đợt 2 K5), M7 plan update bypass menu khi từ track (foundation Đợt 3 G9 reason payload). Đợt 4 chủ yếu refactor + dọn duplication, nên rủi ro thấp hơn Đợt 3.
