# OKR Skill — Đợt 2: Solo Defaults Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Gỡ bias team-oriented + check-in cứng nhắc khỏi default flow của bộ skill OKR. Sau đợt này, "1 user solo + objective mơ hồ + check-in tự nhiên" trở thành persona DUY NHẤT, không còn hỏi/ép field cho team. Sửa 6 review items: G10/K3 (bỏ verifier), K6 (resource form solo), K5 (capture 4 type), K1 (track quick confirm 1 dòng), M3 (Quality Gate transparent), K2 (re-validate SMART trên update).

**Architecture:** Schema-breaking refactor + UX flow refactor. Mỗi task = sửa 1 file hoặc 1 phạm vi nhỏ trong 1 file, verify bằng `grep` trước/sau, commit ngay khi xanh. Không viết migration script: chưa có user thật, chỉ thêm note inline trong skill files để LLM tương lai gặp data cũ (action có `verifier:` field, inbox có `type: idea | note`, resources.md kiểu team) thì biết bỏ qua hoặc map.

**Tech Stack:** Markdown + YAML frontmatter. Verify bằng `grep -nE` / `awk`. Không cần test framework.

**Phạm vi files (8 files):**
- `skills/okr-plan/references/task-format.md` (G10/K3 — bỏ verifier khỏi template YAML)
- `skills/okr-plan/references/action-guide.md` (G10/K3 — bỏ section "Verifier")
- `skills/okr-plan/SKILL.md` (G10/K3 — bỏ "Ai verify output?" khỏi Quy tắc; M3 — Quality Gate transparent)
- `skills/okr-init/references/data-format.md` (K6 — schema resources.md solo profile, bỏ cột "Người quản lý")
- `skills/okr-init/SKILL.md` (K6 — Phase 5 NEW + Mode UPDATE-RESOURCE; M3 — Quality Gate transparent; K2 — Phase 4 update-objective SMART re-validate)
- `skills/okr-capture/SKILL.md` (K5 — Phase 2 bảng 4 type)
- `skills/okr-capture/references/data-format.md` (K5 — frontmatter type values + xử lý inbox bảng)
- `skills/okr-track/SKILL.md` (K1 — Phase 4a confirm 1 dòng khi ≤2 field)

**Out of scope (sẽ làm ở đợt sau):**
- M4 Impact check khi update-objective (Đợt 3)
- G11 Init quét inbox (Đợt 3)
- M6/K4 Effort xl checkpoints (Đợt 3)
- M5 Capture đơn giản, validate ở track (Đợt 4)
- M7 Plan update bypass menu khi từ track (Đợt 4)
- T1-T5 dọn thừa (Đợt 4)

---

## File Structure

| File | Phạm vi sửa | Lý do |
|------|-------------|-------|
| `skills/okr-plan/references/task-format.md` | Block YAML template (line 22) | Field `verifier: ""` không phù hợp solo. |
| `skills/okr-plan/references/action-guide.md` | Bỏ section "## Verifier (ai verify output)" (line 39-47) | Không hỏi field này khi tạo action. |
| `skills/okr-plan/SKILL.md` | Quy tắc "Ai verify output?" (line 200-203) + Quality Gate (line 25-30) | Bỏ verifier yêu cầu; Quality Gate fail giải thích lý do trước follow-up. |
| `skills/okr-init/references/data-format.md` | Section schema `resources.md` body (line 62-69) | Đổi "Nhân sự" thành "Solo Profile", bỏ cột "Người quản lý" khỏi "Công cụ". |
| `skills/okr-init/SKILL.md` | Phase 5+6 mode NEW (line 164-217), Mode UPDATE-RESOURCE Phase 1+2 (line 268-302), Quality Gate (line 27-33), Mode UPDATE-OBJECTIVE Phase 4 (line 246-257) | Solo flow only; Quality Gate transparent; SMART re-validate trên update. |
| `skills/okr-capture/SKILL.md` | Phase 2 bảng type (line 32-47) | 5 type → 4 type (gộp idea+note → thought). |
| `skills/okr-capture/references/data-format.md` | Frontmatter type values (line 17) + bảng "Xử lý inbox" (line 49-56) | Sync 4 type. |
| `skills/okr-track/SKILL.md` | Phase 4a Project step 2 (line 113-119) + Ongoing step 5 (line 138) | Đếm field thay đổi, ≤2 → confirm 1 dòng, ≥3 → confirm bảng. |

---

## Thứ tự thực thi

Tasks 5, 10, 12 đều sửa `skills/okr-init/SKILL.md` ở 3 vị trí khác nhau (Phase 5/6 + UPDATE-RESOURCE; Quality Gate; Phase 4 update-objective). Làm tuần tự theo thứ tự trong plan để tránh edit conflict do line number shift. Tasks khác sửa file khác → có thể song song nếu agent muốn, nhưng plan này giả định tuần tự.

---

## Task 1: G10/K3.a — Bỏ field `verifier` khỏi `task-format.md` template

**Files:**
- Modify: `skills/okr-plan/references/task-format.md:11-26` (block YAML template)

**Mục tiêu:** Template action file không còn field `verifier: ""`. Solo user verify qua DoD checklist là đủ.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "verifier" skills/okr-plan/references/task-format.md
```

Expected: 1 match ở line 22 (`verifier: ""`).

- [ ] **Step 2: Bỏ dòng `verifier: ""` khỏi YAML template**

Edit `skills/okr-plan/references/task-format.md`:

old_string:
```
pic: ""
verifier: ""
due_date: YYYY-MM-DD
```

new_string:
```
pic: ""
due_date: YYYY-MM-DD
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "verifier" skills/okr-plan/references/task-format.md
```

Expected: KHÔNG có match.

Run:
```bash
sed -n '11,26p' skills/okr-plan/references/task-format.md
```

Expected: YAML template gồm các field id, title, description, key_result, milestone, status, priority, effort, pic, due_date, depends_on. KHÔNG có verifier.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/references/task-format.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): drop verifier field from action template (G10/K3)

Solo user mặc định verify qua DoD checklist, không cần field
verifier riêng. Bỏ khỏi YAML template trong task-format.md.

Refs: deep-insight/okr-skill-review-260510.md G10, K3
EOF
)"
```

---

## Task 2: G10/K3.b — Bỏ section "Verifier (ai verify output)" khỏi `action-guide.md`

**Files:**
- Modify: `skills/okr-plan/references/action-guide.md:39-47` (section Verifier)

**Mục tiêu:** `action-guide.md` không còn hỏi/document verifier. Thay bằng note ngắn "Verifier mặc định là user tự verify qua DoD checklist".

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "Verifier\|verify" skills/okr-plan/references/action-guide.md
```

Expected: tìm thấy:
- line 39: `## Verifier (ai verify output)`
- line 40: text giới thiệu
- line 42-47: bảng "Cách verify"
- line 53-55: 2 mention trong anti-patterns ("ai measure", không liên quan section Verifier)

- [ ] **Step 2: Thay section "Verifier (ai verify output)" bằng note ngắn**

Edit `skills/okr-plan/references/action-guide.md`:

old_string:
```
## Verifier (ai verify output)

Mỗi action cần rõ ai kiểm tra kết quả. Tránh tình trạng "tự làm tự đánh giá".

| Cách verify | Khi nào dùng | Ví dụ |
|------------|-------------|-------|
| **Người cụ thể** | Có team, output cần review | "An (PM) review report" |
| **User tự verify** | Solo project, output đo được | "Chạy test suite, ≥80% pass" |
| **Metric tự động** | Có hệ thống đo | "GA4 tracking confirm DAU ≥ target" |
| **Checklist DoD** | Output rõ ràng, check từng tiêu chí | "Đánh dấu hoàn thành mọi item trong DoD" |

## Anti-patterns (action xấu)
```

new_string:
```
## Verify output

Bộ skill OKR mặc định **user tự verify qua DoD checklist**: mỗi tiêu chí trong `## Definition of Done` phải đo được, check được. Action xong khi mọi item DoD tick xanh. Không cần field `verifier` riêng (đã bỏ khỏi schema).

Hệ quả khi viết DoD:
- Mỗi item DoD phải verify được không cần ai khác (vd: "Test suite chạy ≥80% pass", "File xuất hiện tại path X", "Số liệu trong dashboard đạt ngưỡng Y").
- Tránh DoD mơ hồ kiểu "Code đẹp", "Doc tốt" (không đo được).
- Nếu output thực sự cần người khác review (rare cho solo) → ghi rõ trong `## Definition of Done` bằng item dạng "Reviewed by [tên]".

## Anti-patterns (action xấu)
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "## Verifier (ai verify output)\|^| \*\*Người cụ thể\*\*\|^| \*\*Metric tự động\*\*" skills/okr-plan/references/action-guide.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "## Verify output|user tự verify qua DoD checklist|Hệ quả khi viết DoD" skills/okr-plan/references/action-guide.md
```

Expected: 3 match (heading + nội dung).

Run:
```bash
awk '/^## /{print NR, $0}' skills/okr-plan/references/action-guide.md
```

Expected: section order vẫn là `5 tiêu chí bắt buộc → Effort → Priority → Verify output → Anti-patterns → Ongoing type`.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/references/action-guide.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): rewrite Verifier section as DoD-based self-verify (G10/K3)

Solo persona không cần field verifier riêng. Đổi section
"Verifier (ai verify output)" thành "Verify output": user tự
verify qua DoD checklist, mỗi item DoD phải đo được. Bỏ bảng
4 cách verify (Người cụ thể/Metric/etc) vốn giả định team.

Refs: deep-insight/okr-skill-review-260510.md G10, K3
EOF
)"
```

---

## Task 3: G10/K3.c — Bỏ "Ai verify output?" khỏi Quy tắc trong `okr-plan/SKILL.md`

**Files:**
- Modify: `skills/okr-plan/SKILL.md:191` (Schema link mention verifier)
- Modify: `skills/okr-plan/SKILL.md:196-203` (Quy tắc "Mỗi action BẮT BUỘC có:" + 2 dòng "Ai verify output")

**Mục tiêu:** Quy tắc không còn yêu cầu "Ai verify output?" hoặc nhắc "verifier" như field bắt buộc.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "verify\|verifier" skills/okr-plan/SKILL.md
```

Expected: tìm thấy:
- line 191: `... 5 tiêu chí bắt buộc, effort, priority, verifier, anti-patterns) ...`
- line 200: `- **Ai verify output?** (người hoặc cơ chế kiểm tra kết quả)`
- line 203: `- Nếu user không trả lời được "ai verify" hoặc "output đo bằng gì" → action chưa đủ rõ, cần refine trước khi ghi.`

- [ ] **Step 2: Sửa Schema link section (line 191)**

Edit `skills/okr-plan/SKILL.md`:

old_string:
```
- `references/action-guide.md`: hướng dẫn viết action chất lượng (5 tiêu chí bắt buộc, effort, priority, verifier, anti-patterns). Đọc trước khi tạo/review actions.
```

new_string:
```
- `references/action-guide.md`: hướng dẫn viết action chất lượng (5 tiêu chí bắt buộc, effort, priority, DoD-based verify, anti-patterns). Đọc trước khi tạo/review actions.
```

- [ ] **Step 3: Sửa Quy tắc "Mỗi action BẮT BUỘC có:" (line 196-203)**

Edit `skills/okr-plan/SKILL.md`:

old_string:
```
- Mỗi action BẮT BUỘC có:
  - Definition of Done rõ ràng
  - Output/Deliverable cụ thể (file, số liệu, sự kiện)
  - PIC (hoặc `unassigned` nếu resource thiếu)
  - **Ai verify output?** (người hoặc cơ chế kiểm tra kết quả)
  - **Tiêu chí chất lượng** (output đạt khi nào? đo bằng gì?)
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output đo được → CẤM. Agent phải follow-up: "Output cụ thể là gì? Ai đọc/dùng output này?"
- Nếu user không trả lời được "ai verify" hoặc "output đo bằng gì" → action chưa đủ rõ, cần refine trước khi ghi.
```

new_string:
```
- Mỗi action BẮT BUỘC có:
  - Definition of Done rõ ràng (mỗi item đo được, user tự check được)
  - Output/Deliverable cụ thể (file, số liệu, sự kiện)
  - PIC (default `self` cho solo)
  - **Tiêu chí chất lượng** (output đạt khi nào? đo bằng gì?)
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output đo được → CẤM. Agent phải follow-up: "Output cụ thể là gì?"
- Nếu user không trả lời được "output đo bằng gì" → action chưa đủ rõ, cần refine trước khi ghi.
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -n "verify\|verifier" skills/okr-plan/SKILL.md
```

Expected: 0 match (toàn bộ mention verifier/verify đã bỏ).

Run:
```bash
grep -nE "DoD-based verify|default \`self\` cho solo|Definition of Done rõ ràng \(mỗi item đo được" skills/okr-plan/SKILL.md
```

Expected: 3 match.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): drop verifier requirement from action rules (G10/K3)

Quy tắc "Mỗi action BẮT BUỘC có ai verify output" giả định team
review. Solo persona verify qua DoD checklist tự check được. Bỏ
yêu cầu verifier khỏi Quy tắc và Schema link, default PIC=self.

Refs: deep-insight/okr-skill-review-260510.md G10, K3
EOF
)"
```

---

## Task 4: K6.a — Schema `resources.md` solo profile + bỏ "Người quản lý"

**Files:**
- Modify: `skills/okr-init/references/data-format.md:54-69` (section `## resources.md (SOT)` đến hết bullet body sections)

**Mục tiêu:** Schema `resources.md` body section "Nhân sự" đổi thành "Solo Profile" (chỉ tên + capacity h/tuần + skills). Section "Công cụ" bỏ cột "Người quản lý". Các section còn lại giữ nguyên.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '54,70p' skills/okr-init/references/data-format.md
```

Expected: thấy block `## resources.md (SOT)` với:
- line 62-67 list 5 bullet sections
- line 63 mention "Họ Tên, Liên lạc (Zalo/FB/SĐT/Địa chỉ), Vai trò & Trách nhiệm, Ngày tham gia, Khả dụng %, Actions"
- line 64 mention "Người quản lý"

- [ ] **Step 2: Sửa block bullet body sections**

Edit `skills/okr-init/references/data-format.md`:

old_string:
```
Body sections (giữ section header dù rỗng):
- `## Nhân sự (Vai trò & Trách nhiệm)` (bảng: Họ Tên, Liên lạc (Zalo/FB/SĐT/Địa chỉ), Vai trò & Trách nhiệm, Ngày tham gia, Khả dụng %, Actions)
- `## Công cụ` (bảng: Tên công cụ, Khi nào dùng, Mục đích, Resource (URL/Account), Người quản lý)
- `## Tài liệu & Knowledge Base` (bảng: Tên/Loại, Vị trí (Link/Folder/File), Mục đích, Status)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách bullet)

Tại thời điểm `mode: new`, cột `Actions` trong section "Nhân sự" để trống (plan chưa có). Mode `update-resource` sẽ map sau khi `okr-plan` tạo actions.
```

new_string:
```
Body sections (giữ section header dù rỗng):
- `## Solo Profile` (bảng 1 dòng: Tên, Capacity (giờ/tuần), Skills)
- `## Công cụ` (bảng: Tên công cụ, Khi nào dùng, Mục đích, Resource (URL/Account))
- `## Tài liệu & Knowledge Base` (bảng: Tên/Loại, Vị trí (Link/Folder/File), Mục đích, Status)
- `## Ngân sách` (bảng: Khoản mục, Dự kiến, Thực tế, Ghi chú)
- `## Thiếu hụt & Rủi ro` (danh sách bullet)

Skill OKR phục vụ persona **solo only** (1 user, 1 objective). Mọi action mặc định `pic: self`. Section `## Solo Profile` chứa duy nhất 1 dòng (chính user). KHÔNG còn các field "Liên lạc", "Vai trò", "Khả dụng %", "Actions" (bảng mapping PIC → action ID): đã bỏ vì không cần với solo. Mode `update-resource` chỉ sửa Solo Profile (capacity, skills) hoặc Công cụ/Tài liệu/Ngân sách.

> **Lưu ý dữ liệu cũ**: Nếu gặp `resources.md` từ schema cũ có section `## Nhân sự (Vai trò & Trách nhiệm)` hoặc cột "Người quản lý" / "Liên lạc", coi là legacy. Mode `update-resource` lần đầu sẽ migrate: tự lấy dòng đầu tiên (hoặc dòng có khả dụng cao nhất) làm Solo Profile, bỏ các cột không còn schema.
```

- [ ] **Step 3: Bỏ section `## Tham chiếu actions/` (PIC mapping không còn relevant)**

Edit `skills/okr-init/references/data-format.md`:

old_string:
```
## Tham chiếu actions/

Khi assign PIC trong mode `update-resource`, sửa frontmatter `.okr/actions/AXXX-*.md`:

```yaml
---
id: AXXX
pic: "Tên người"
---
```

Sync 2 chiều: `resources.md` (cột Actions của người đó) và frontmatter `pic` của action.

## Phát hiện xung đột
```

new_string:
```
## Phát hiện xung đột
```

- [ ] **Step 4: Rewrite section `## Phát hiện xung đột` cho solo**

Edit `skills/okr-init/references/data-format.md`:

old_string:
```
## Phát hiện xung đột

Áp dụng khi update resource hoặc khi `okr-plan` gọi vào để map PIC:

| Tín hiệu | Cảnh báo |
|----------|----------|
| Cùng PIC có ≥3 actions cùng deadline (±2 ngày) | Quá tải, đề xuất tách task hoặc dời |
| PIC khả dụng <50% có >5 actions trong period | Capacity không đủ, đề xuất thêm người |
| Action có deadline trước ngày PIC sẵn sàng | Bất khả thi, đề xuất dời deadline |
| Action không có PIC mà deadline <7 ngày | Cần assign gấp |
| Tool/tài liệu status `missing` mà có action phụ thuộc | Block, đề xuất bổ sung trước |

Mỗi cảnh báo phải kèm đề xuất giải pháp cụ thể: dời deadline, tách task, thêm người, hoặc giảm scope.
```

new_string:
```
## Phát hiện xung đột (solo)

Áp dụng khi `okr-init update-resource` hoàn tất, hoặc khi `okr-plan` đọc resources để check fit:

| Tín hiệu | Cảnh báo |
|----------|----------|
| Tổng giờ ước tính của actions chưa done > capacity còn lại đến end_date | Quá tải, đề xuất giảm scope hoặc dời deadline |
| ≥3 actions cùng deadline (±2 ngày) | Tuần đó dồn việc, đề xuất tách deadline |
| Action cần skill chưa có trong Solo Profile | Đề xuất thêm action học/outsource trước |
| Tool/tài liệu status `missing` mà có action phụ thuộc | Block, đề xuất bổ sung trước |
| Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done | Cảnh báo scope rủi ro, đề xuất xem lại plan |

Mỗi cảnh báo phải kèm đề xuất giải pháp cụ thể: dời deadline, tách task, học/outsource skill, hoặc giảm scope.
```

- [ ] **Step 5: Bỏ bullet "Update PIC PHẢI sync..." khỏi Quy tắc chung**

Edit `skills/okr-init/references/data-format.md`:

old_string:
```
- SOT: ghi đè khi cập nhật, không append. Lịch sử thay đổi đi vào log của `okr-track`.
- Section trống vẫn giữ header để mode `update-*` có chỗ chèn.
- Mọi sửa đổi `resources.md` phải cập nhật `last_updated`.
- Update PIC PHẢI sync cả `resources.md` lẫn frontmatter `actions/*.md`.
```

new_string:
```
- SOT: ghi đè khi cập nhật, không append. Lịch sử thay đổi đi vào log của `okr-track`.
- Section trống vẫn giữ header để mode `update-*` có chỗ chèn.
- Mọi sửa đổi `resources.md` phải cập nhật `last_updated`.
- Field `pic` trong frontmatter `actions/*.md` mặc định `self`. Không cần sync 2 chiều với `resources.md` (Solo Profile chỉ 1 user).
```

- [ ] **Step 6: Verify**

Run:
```bash
grep -nE "Họ Tên, Liên lạc|Zalo/FB/SĐT|Người quản lý|Vai trò & Trách nhiệm|## Tham chiếu actions/|Cùng PIC có ≥3 actions|PIC khả dụng <50%|Update PIC PHẢI sync" skills/okr-init/references/data-format.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "## Solo Profile|Capacity \(giờ/tuần\)|Skills|solo only|Lưu ý dữ liệu cũ|## Phát hiện xung đột \(solo\)|capacity còn lại đến end_date|skill chưa có trong Solo Profile|Field \`pic\` trong frontmatter.*mặc định \`self\`" skills/okr-init/references/data-format.md
```

Expected: ≥7 match.

Run:
```bash
awk '/^## /{print NR, $0}' skills/okr-init/references/data-format.md
```

Expected: section order là `objective.md` → `resources.md (SOT)` → `Phát hiện xung đột (solo)` → `Quy tắc chung`. KHÔNG còn `## Tham chiếu actions/`.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-init/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-init): rewrite resources.md schema for solo persona (K6)

Persona đã chốt là solo (1 user, 1 objective). Schema thay đổi:
- Section "Nhân sự" → "Solo Profile" (1 dòng: Tên, Capacity
  h/tuần, Skills). Bỏ cột "Liên lạc (Zalo/FB/SĐT)", "Vai trò",
  "Khả dụng %", "Actions" mapping.
- Section "Công cụ" bỏ cột "Người quản lý".
- Bỏ section "## Tham chiếu actions/" (PIC mapping team-only).
- Section "## Phát hiện xung đột" rewrite cho solo: dùng
  capacity h/tuần thay vì PIC %, thêm tín hiệu skill thiếu.
- Quy tắc chung: pic mặc định self, không cần sync 2 chiều.
- Note migrate legacy schema cho LLM tương lai.

Refs: deep-insight/okr-skill-review-260510.md K6
EOF
)"
```

---

## Task 5: K6.b — `okr-init/SKILL.md` flow solo only (Phase 5+6 NEW + UPDATE-RESOURCE)

**Files:**
- Modify: `skills/okr-init/SKILL.md:164-217` (Phase 5 NEW + Phase 6 confirm)
- Modify: `skills/okr-init/SKILL.md:265-323` (Mode UPDATE-RESOURCE Phase 1 + Phase 2 + Phase 5)

**Mục tiêu:** Flow Resource collection chỉ còn solo path. Bỏ Bước 0 "Solo hay team?". Phase 5/6 chỉ hỏi Solo Profile (capacity, skills) + Công cụ + Tài liệu + Ngân sách + Rủi ro. Mode UPDATE-RESOURCE menu rút gọn (bỏ "Mapping PIC", "Đổi PIC hàng loạt", "Thêm/xoá người").

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '164,225p' skills/okr-init/SKILL.md
```

Expected: thấy "Phase 5: Thu thập Resource (hỏi tuần tự + cross-check)" với:
- "Bước 0: Solo hay team?" (line 166-173)
- Bullet "1. Nhân sự (team): Ai tham gia? ..." (line 176)
- "Cross-check resource vs. scope" + công thức person-months (line 184-194)
- Phase 6 CONFIRM với bảng có "Người", "Capacity: 1.5 FTE × 3 tháng" (line 196-217)

Run:
```bash
sed -n '265,325p' skills/okr-init/SKILL.md
```

Expected: thấy "Mode UPDATE-RESOURCE" với:
- Phase 1 bảng có "Người: 3, An (PM), Bình (Dev), Chi (Design)" (line 272-281)
- Phase 2 menu 7 mục, gồm "1. Thêm/sửa/xoá người", "5. Mapping PIC vào actions", "6. Đổi PIC hàng loạt"
- Phase 5 áp dụng có "Update field `pic` trong frontmatter `actions/*.md`"

- [ ] **Step 2: Thay Phase 5 mode NEW (bỏ Bước 0, đổi thành solo flow)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 5: Thu thập Resource (hỏi tuần tự + cross-check)

**Bước 0: Solo hay team?**

Hỏi: "Chỉ bạn thực hiện hay có team?"

- **Solo**: skip phần nhân sự chi tiết (Zalo, SĐT, vai trò). Thay vào đó hỏi:
  - **Effort commitment**: Bạn dành bao nhiêu thời gian/tuần cho mục tiêu này? (vd: 10 giờ/tuần, mỗi tối 1 tiếng)
  - Công cụ, tài liệu, ngân sách: hỏi như thường nhưng rút gọn (gợi ý, không bắt buộc).
- **Team**: hỏi đầy đủ phần nhân sự như dưới.

Hỏi tuần tự:
1. **Nhân sự** (team): Ai tham gia? Thu thập: Họ tên, thông tin liên lạc (nick Zalo, Facebook, SĐT, địa chỉ), vai trò, trách nhiệm, ngày tham gia, khả dụng (% thời gian), và ai quản lý công cụ nào.
2. **Công cụ**: Danh sách công cụ sẽ sử dụng, khi nào dùng, dùng để làm gì, resource liên quan (URL, account), và người quản lý.
3. **Tài liệu / Knowledge Base**: Các tài liệu, hệ thống lưu trữ hiện có hoặc cần thiết (Link, folder, file), status (có sẵn/cần tạo/đang thiếu).
4. **Ngân sách** (nếu Project hoặc cần đầu tư): Có ngân sách không? Bao nhiêu?
5. **Thiếu hụt**: Có rủi ro/thiếu hụt nào nhận biết được không?

User skip được (`không có` / `để sau`), nhưng field skip đánh dấu `⚠️ TBD`.

**Cross-check resource vs. scope (bắt buộc sau khi thu thập xong):**

Agent tự tính toán sơ bộ trước khi sang Phase 6:
- Tổng available capacity = Σ (người × % khả dụng × thời gian)
- Ước tính scope từ KR/KI đã define (Phase 3)
- So sánh: capacity có đủ cho scope không?

Nếu lệch rõ rệt → cảnh báo user ngay, hỏi:
- "Scope cần khoảng X person-months, team hiện có Y. Có kế hoạch bổ sung không, hay cần thu hẹp scope?"

Không cần chính xác tuyệt đối. Mục tiêu là phát hiện bất hợp lý lớn (vd: 1 người làm việc của 5 người).
```

new_string:
```
### Phase 5: Thu thập Resource (hỏi tuần tự + cross-check)

Skill phục vụ persona **solo only** (1 user, 1 objective). KHÔNG còn nhánh team. Hỏi tuần tự:

1. **Solo Profile**:
   - **Capacity**: Bạn dành bao nhiêu giờ/tuần cho mục tiêu này? (vd: 10 giờ/tuần, mỗi tối 1 tiếng)
   - **Skills liên quan**: Bạn đã có sẵn kỹ năng nào phục vụ mục tiêu này? (vd: Python, viết content, design Figma). Mục đích: agent biết tự tin đề xuất action loại nào, và cảnh báo khi action cần skill chưa có.
2. **Công cụ**: Danh sách công cụ sẽ sử dụng, khi nào dùng, dùng để làm gì, resource liên quan (URL, account).
3. **Tài liệu / Knowledge Base**: Các tài liệu, hệ thống lưu trữ hiện có hoặc cần thiết (Link, folder, file), status (có sẵn/cần tạo/đang thiếu).
4. **Ngân sách** (nếu Project hoặc cần đầu tư): Có ngân sách không? Bao nhiêu?
5. **Thiếu hụt**: Có rủi ro/thiếu hụt nào nhận biết được không? (vd: skill còn thiếu, tool chưa mua, deadline ngoài tầm kiểm soát)

User skip được (`không có` / `để sau`), nhưng field skip đánh dấu `⚠️ TBD`.

**Cross-check capacity vs. scope (bắt buộc sau khi thu thập xong):**

Agent tự tính toán sơ bộ trước khi sang Phase 6:
- Tổng available capacity = `capacity h/tuần × số tuần đến end_date` (Project) hoặc `capacity h/tuần × 4` (Ongoing, dùng 1 tháng làm horizon đánh giá)
- Ước tính scope từ KR/KI đã define (Phase 3): mỗi KR cần khoảng bao nhiêu giờ?
- So sánh: capacity có đủ cho scope không?

Nếu lệch rõ rệt → cảnh báo user ngay, hỏi:
- "Scope cần khoảng X giờ, capacity hiện có Y giờ. Cần thu hẹp scope, extend deadline, hay tăng capacity?"

Không cần chính xác tuyệt đối. Mục tiêu là phát hiện bất hợp lý lớn (vd: 10 giờ/tuần làm việc của 40 giờ/tuần).
```

- [ ] **Step 3: Thay Phase 6 CONFIRM Resource (đổi bảng cho solo)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 6: CONFIRM Resource (BẮT BUỘC, kèm đánh giá)

```
Tóm tắt Resource
| Mục            | Giá trị                                  |
|----------------|------------------------------------------|
| Người          | An (PM, 100%, từ 01/10), Bình (Dev, 50%) |
| Công cụ        | Notion (Task), GitHub, Figma             |
| Tài liệu/KB    | brief.pdf, thư mục Drive dự án           |
| Ngân sách      | 50M VND                                  |
| Rủi ro         | Bình kiêm 2 dự án                        |

Đánh giá nhanh
  Capacity: 1.5 FTE × 3 tháng = 4.5 person-months
  Scope ước tính: [dựa trên KR/KI] ≈ 5 person-months
  Fit: ⚠️ hơi thiếu, cần buffer hoặc thu hẹp scope
  Rủi ro chính:
  - Bình 50% available, nếu dự án kia tăng tải → bottleneck
  - ⚠️ Ngân sách chưa rõ (TBD)

Xác nhận? (y / sửa / huỷ)
```
```

new_string:
```
### Phase 6: CONFIRM Resource (BẮT BUỘC, kèm đánh giá)

```
Tóm tắt Resource
| Mục            | Giá trị                                  |
|----------------|------------------------------------------|
| Solo Profile   | Tuấn, 10h/tuần, skills: Python, viết     |
| Công cụ        | Notion (Task), GitHub, Figma             |
| Tài liệu/KB    | brief.pdf, thư mục Drive                 |
| Ngân sách      | 5M VND                                   |
| Rủi ro         | Skill design Figma còn yếu               |

Đánh giá nhanh
  Capacity: 10h/tuần × 12 tuần = 120 giờ
  Scope ước tính: [dựa trên KR/KI] ≈ 150 giờ
  Fit: ⚠️ hơi thiếu, cần buffer hoặc thu hẹp scope
  Rủi ro chính:
  - Skill Figma còn yếu, cân nhắc dùng template/outsource phần design
  - ⚠️ Ngân sách chưa rõ (TBD)

Xác nhận? (y / sửa / huỷ)
```
```

- [ ] **Step 4: Thay Mode UPDATE-RESOURCE Phase 1 (bảng hiển thị)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 1: Hiển thị state hiện tại

Đọc `resources.md` + frontmatter `actions/*.md` (nếu có). Hiển thị:

```
Resource hiện tại
| Mục              | Số lượng | Chi tiết                          |
|------------------|----------|-----------------------------------|
| Người            | 3        | An (PM), Bình (Dev), Chi (Design) |
| Công cụ          | 4        | Notion, GitHub, Figma, Slack      |
| Tài liệu/KB      | 2        | brief.pdf, Drive folder           |
| Ngân sách        | 50M VND  | đã chi: 12M                       |
| Actions chưa PIC | 2        | A007, A011                        |
| Cảnh báo         | 1        | Bình khả dụng <50%, có 5 actions  |
```
```

new_string:
```
### Phase 1: Hiển thị state hiện tại

Đọc `resources.md`. Hiển thị:

```
Resource hiện tại
| Mục            | Giá trị                                |
|----------------|----------------------------------------|
| Solo Profile   | Tuấn, 10h/tuần, skills: Python, viết   |
| Công cụ        | 4 (Notion, GitHub, Figma, Slack)       |
| Tài liệu/KB    | 2 (brief.pdf, Drive folder)            |
| Ngân sách      | 5M VND (đã chi: 1.2M)                  |
| Rủi ro         | 1 (Skill Figma còn yếu)                |
```

> **Legacy migrate**: Nếu file vẫn còn section `## Nhân sự (Vai trò & Trách nhiệm)` schema cũ, agent tự convert: lấy dòng đầu (hoặc dòng có khả dụng cao nhất) thành Solo Profile, drop các cột "Liên lạc", "Người quản lý", "Khả dụng %", "Actions". Thông báo user "Đã migrate schema cũ sang Solo Profile."
```

- [ ] **Step 5: Thay Mode UPDATE-RESOURCE Phase 2 (menu rút gọn)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 2: Hỏi user muốn update gì

Menu:
1. Thêm/sửa/xoá người
2. Thêm/sửa/xoá công cụ
3. Thêm/sửa/xoá tài liệu / knowledge base
4. Update ngân sách
5. Mapping PIC vào actions
6. Update rủi ro/thiếu hụt
7. Re-check xung đột

User chọn nhiều cùng lúc được (vd "1, 4").

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn user. Vd nếu chọn 4:
- Liệt kê actions chưa PIC.
- Đề xuất PIC dựa trên skill match + khả dụng.
- User confirm từng cái.
```

new_string:
```
### Phase 2: Hỏi user muốn update gì

Menu:
1. Sửa Solo Profile (capacity h/tuần, skills)
2. Thêm/sửa/xoá công cụ
3. Thêm/sửa/xoá tài liệu / knowledge base
4. Update ngân sách
5. Update rủi ro/thiếu hụt

User chọn nhiều cùng lúc được (vd "1, 4").

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn user. Vd nếu chọn 1:
- Hỏi capacity mới: "Tuần này bạn còn dành được X h/tuần không?"
- Hỏi có thêm skill mới sau quá trình thực thi (vd học được kỹ năng mới)?
```

- [ ] **Step 6: Thay Phase 4 CONFIRM diff (bỏ row PIC) + Phase 5 áp dụng (bỏ sync PIC frontmatter)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng
| Loại          | Trước              | Sau                  |
|---------------|--------------------|----------------------|
| Thêm người    | -                  | Dũng (QA, 80%)       |
| Sửa PIC       | A007: unassigned   | A007: Dũng           |
| Sửa khả dụng  | Bình: 100%         | Bình: 50%            |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Update field `pic` trong frontmatter `actions/*.md` (nếu có mapping mới).
3. Check xung đột (xem section "Phát hiện xung đột" trong `references/data-format.md`):
   - PIC khả dụng <50% nhưng có >N actions cùng deadline → cảnh báo
   - Action có deadline mà PIC chưa khả dụng → cảnh báo
4. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."
```

new_string:
```
### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng
| Loại                  | Trước              | Sau                  |
|-----------------------|--------------------|----------------------|
| Sửa capacity          | 10h/tuần           | 6h/tuần              |
| Thêm skill            | Python, viết       | Python, viết, SQL    |
| Sửa ngân sách         | 5M VND             | 8M VND               |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Cảnh báo cross-check:
   - Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done → cảnh báo "Capacity giảm, scope có thể không kịp deadline. Cân nhắc dời actions sang sau hoặc giảm scope qua `/okr plan update`."
   - Skill mới thêm có thể unlock action TBD trước đó (vd "Skill SQL mới: action A007 trước đây skip vì thiếu skill có thể làm lại").
3. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."
```

- [ ] **Step 7: Verify**

Run:
```bash
grep -nE "Solo hay team\?|Bước 0: Solo|FTE|person-months|Họ tên, thông tin liên lạc|Mapping PIC vào actions|Đổi PIC hàng loạt" skills/okr-init/SKILL.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "solo only|Solo Profile|Capacity: 10h/tuần|capacity h/tuần|Legacy migrate" skills/okr-init/SKILL.md
```

Expected: ≥4 match.

Run:
```bash
awk '/^### Phase /{print NR, $0}' skills/okr-init/SKILL.md
```

Expected: thứ tự phase NEW: Phase 0 → 1 → 2 → 3 → 4 → 5 → 6 → 7. UPDATE-OBJECTIVE: Phase 1 → 2 → 3 → 4 → 5. UPDATE-RESOURCE: Phase 1 → 2 → 3 → 4 → 5.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): drop team flow from resource phases (K6)

Persona đã chốt solo only. Phase 5 NEW bỏ Bước 0 "Solo hay
team?" + đổi nhánh nhân sự thành Solo Profile (capacity h/tuần,
skills). Phase 6 confirm dùng giờ thay vì person-months/FTE.
Mode UPDATE-RESOURCE bảng hiện tại + menu rút gọn (5 mục, bỏ
"Mapping PIC", "Đổi PIC hàng loạt", "Thêm/xoá người"). Phase 5
apply bỏ sync field pic vào actions/*.md (mặc định pic=self).

Refs: deep-insight/okr-skill-review-260510.md K6
EOF
)"
```

---

## Task 6: K5.a — `okr-capture/SKILL.md` Phase 2 bảng 4 type

**Files:**
- Modify: `skills/okr-capture/SKILL.md:32-47` (Phase 2 bảng type 5 hàng)

**Mục tiêu:** Bảng type còn 4 hàng: `action`, `blocker`, `resource`, `thought` (gộp `idea` + `note` → `thought`).

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "| \`action\`\|| \`idea\`\|| \`blocker\`\|| \`resource\`\|| \`note\`\|| \`thought\`" skills/okr-capture/SKILL.md
```

Expected:
- 1 match `| \`action\`` line ~37
- 1 match `| \`idea\`` line ~38
- 1 match `| \`blocker\`` line ~39
- 1 match `| \`resource\`` line ~40
- 1 match `| \`note\`` line ~41
- KHÔNG có `| \`thought\``

- [ ] **Step 2: Thay bảng type**

Edit `skills/okr-capture/SKILL.md`:

old_string:
```
| Type | Khi nào | Ví dụ |
|------|---------|-------|
| `action` | Việc cần làm cụ thể, có output rõ | "Viết unit test cho API auth" |
| `idea` | Ý tưởng chưa rõ, cần suy nghĩ thêm | "Có thể dùng framework X cho frontend?" |
| `blocker` | Vấn đề đang chặn tiến độ | "Server staging bị down, không deploy được" |
| `resource` | Tài nguyên mới phát hiện (người, tool, doc) | "Tìm được thư viện Y hỗ trợ chart, miễn phí" |
| `note` | Ghi chú tự do, context bổ sung | "Khách hàng muốn demo trước 15/6" |
```

new_string:
```
| Type | Khi nào | Ví dụ |
|------|---------|-------|
| `action` | Việc cần làm cụ thể, có output rõ | "Viết unit test cho API auth" |
| `blocker` | Vấn đề đang chặn tiến độ | "Server staging bị down, không deploy được" |
| `resource` | Tài nguyên mới phát hiện (tool, doc) | "Tìm được thư viện Y hỗ trợ chart, miễn phí" |
| `thought` | Ý tưởng, ghi chú, context bổ sung (chưa thành action rõ) | "Có thể dùng framework X cho frontend?", "Khách hàng muốn demo trước 15/6" |

> **Lưu ý dữ liệu cũ**: Schema trước đây có 5 type với `idea` + `note` riêng biệt. Đã gộp thành `thought` để giảm friction phân loại. Nếu gặp inbox file cũ với `type: idea` hoặc `type: note`, coi như `type: thought` khi xử lý.
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "| \`idea\`\|| \`note\`" skills/okr-capture/SKILL.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "\| \`thought\` \||Lưu ý dữ liệu cũ.*idea.*note" skills/okr-capture/SKILL.md
```

Expected: 2 match (1 row bảng + 1 note).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-capture/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-capture): merge idea+note into thought type (K5)

5 type quá phức tạp, idea vs note thường mơ hồ. Gộp thành
`thought` (ý tưởng, ghi chú, context). Còn 4 type: action,
blocker, resource, thought. Thêm note migrate cho LLM gặp file
cũ với type: idea | note.

Refs: deep-insight/okr-skill-review-260510.md K5
EOF
)"
```

---

## Task 7: K5.b — `okr-capture/references/data-format.md` frontmatter + xử lý inbox 4 type

**Files:**
- Modify: `skills/okr-capture/references/data-format.md:14-23` (frontmatter type values)
- Modify: `skills/okr-capture/references/data-format.md:46-56` (bảng "Xử lý inbox")

**Mục tiêu:** Schema YAML frontmatter và bảng "Xử lý inbox" đồng bộ 4 type.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
grep -n "type: action\|type: idea\|type: blocker\|type: resource\|type: note\|type: thought" skills/okr-capture/references/data-format.md
```

Expected: 1 match line ~17 dạng `type: action | idea | blocker | resource | note`.

Run:
```bash
sed -n '46,58p' skills/okr-capture/references/data-format.md
```

Expected: bảng "Xử lý inbox" 5 hàng với rows `action`, `idea`, `blocker`, `resource`, `note`.

- [ ] **Step 2: Sửa frontmatter type values**

Edit `skills/okr-capture/references/data-format.md`:

old_string:
```
type: action | idea | blocker | resource | note
```

new_string:
```
type: action | blocker | resource | thought
```

- [ ] **Step 3: Sửa bảng "Xử lý inbox"**

Edit `skills/okr-capture/references/data-format.md`:

old_string:
```
| Inbox type | Xử lý mặc định | Delegate |
|------------|----------------|----------|
| `action` | Chuyển thành action file trong `actions/` | `okr-plan` mode `update` |
| `idea` | Giữ inbox (chờ rõ hơn) hoặc chuyển thành action | Tuỳ user |
| `blocker` | Đánh dấu action liên quan = blocked | `okr-track` tự xử lý |
| `resource` | Thêm vào resources.md | `okr-init` mode `update-resource` |
| `note` | Append vào log ngày | `okr-track` tự xử lý |
```

new_string:
```
| Inbox type | Xử lý mặc định | Delegate |
|------------|----------------|----------|
| `action` | Chuyển thành action file trong `actions/` | `okr-plan` mode `update` |
| `blocker` | Đánh dấu action liên quan = blocked | `okr-track` tự xử lý |
| `resource` | Thêm vào resources.md (Công cụ / Tài liệu) | `okr-init` mode `update-resource` |
| `thought` | Giữ inbox (chờ rõ hơn) hoặc append vào log ngày, hoặc chuyển thành action nếu user xác nhận | Tuỳ user / `okr-track` tự xử lý |

> **Migrate dữ liệu cũ**: File inbox cũ với `type: idea` hoặc `type: note` → coi như `type: thought` khi xử lý. `okr-track` Phase 5 (inbox processing) phát hiện type cũ → tự update sang `thought` khi đổi `status: processed`.
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -n "type: idea\|type: note\|| \`idea\`\|| \`note\`" skills/okr-capture/references/data-format.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "type: action \| blocker \| resource \| thought|\| \`thought\` \||Migrate dữ liệu cũ" skills/okr-capture/references/data-format.md
```

Expected: 3 match.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-capture/references/data-format.md
git commit -m "$(cat <<'EOF'
docs(okr-capture): sync 4 type schema in frontmatter + processing (K5)

Frontmatter type: action | blocker | resource | thought (bỏ
idea, note). Bảng "Xử lý inbox" đồng bộ 4 hàng. Note migrate
cho file cũ: okr-track Phase 5 tự update khi gặp type: idea
| note → thought.

Refs: deep-insight/okr-skill-review-260510.md K5
EOF
)"
```

---

## Task 8: K1 — `okr-track/SKILL.md` Phase 4a confirm 1 dòng khi ≤2 field

**Files:**
- Modify: `skills/okr-track/SKILL.md:113-119` (Phase 4a Project step 2 CONFIRM)
- Modify: `skills/okr-track/SKILL.md:138` (Phase 4a Ongoing step 5 CONFIRM)

**Mục tiêu:** Mode `light` đếm số field thay đổi. ≤2 field → confirm 1 dòng. ≥3 field → confirm bảng đầy đủ. Giảm friction quick check-in.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '110,142p' skills/okr-track/SKILL.md
```

Expected: thấy:
- line 113: "2. CONFIRM trước khi ghi:" (Project)
- line 114-119: code block bảng "Thay đổi sắp ghi" 3 row
- line 138: "5. CONFIRM trước khi ghi (tương tự Project)." (Ongoing)

- [ ] **Step 2: Thay step 2 Project (CONFIRM nhánh đếm field)**

Edit `skills/okr-track/SKILL.md`:

old_string:
```
2. CONFIRM trước khi ghi:
  ```
   Thay đổi sắp ghi
   - KR1.current: 40 > 50
   - A003.status: doing > done
   - A005.status: doing > blocked, lý do: chờ approve
   Xác nhận? (y/sửa/huỷ)
  ```
```

new_string:
```
2. CONFIRM trước khi ghi (đếm số field thay đổi rồi chọn UI):
   - **Nếu ≤2 field thay đổi** → confirm 1 dòng (giảm friction quick check-in):
     ```
     KR1: 40>50, A003 done. y/n/sửa?
     ```
     hoặc:
     ```
     A005 blocked (chờ approve). y/n/sửa?
     ```
   - **Nếu ≥3 field thay đổi** → confirm bảng đầy đủ:
     ```
     Thay đổi sắp ghi
     - KR1.current: 40 > 50
     - A003.status: doing > done
     - A005.status: doing > blocked, lý do: chờ approve
     Xác nhận? (y/sửa/huỷ)
     ```
   - User trả lời `sửa` → quay lại bước 1 hỏi field nào sửa.
```

- [ ] **Step 3: Sửa step 5 Ongoing (link sang nhánh đếm field)**

Edit `skills/okr-track/SKILL.md`:

old_string:
```
5. CONFIRM trước khi ghi (tương tự Project).
```

new_string:
```
5. CONFIRM trước khi ghi (tương tự Project: ≤2 field → 1 dòng, ≥3 field → bảng).
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -nE "đếm số field thay đổi|≤2 field|≥3 field|y/n/sửa\?" skills/okr-track/SKILL.md
```

Expected: ≥4 match (1 hướng dẫn đếm + 1 ≤2 + 1 ≥3 + 1 ví dụ y/n/sửa, có thể nhiều hơn).

Run:
```bash
grep -nE "tương tự Project: ≤2 field" skills/okr-track/SKILL.md
```

Expected: 1 match (Ongoing step 5).

- [ ] **Step 5: Commit**

```bash
git add skills/okr-track/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-track): light mode confirm 1 dòng khi ≤2 field (K1)

Quick check-in chỉ thay 1-2 field nhưng vẫn phải xem bảng 3+
dòng → friction cao. Phase 4a step 2 (Project) đếm số field, ≤2
→ confirm 1 dòng "KR1: 40>50, A003 done. y/n/sửa?", ≥3 → giữ
bảng đầy đủ như cũ. Ongoing step 5 dùng cùng nhánh.

Refs: deep-insight/okr-skill-review-260510.md K1
EOF
)"
```

---

## Task 9: M3.a — `okr-init/SKILL.md` Quality Gate transparent

**Files:**
- Modify: `skills/okr-init/SKILL.md:27-33` (bảng "Hành vi theo kết quả")

**Mục tiêu:** Khi Quality Gate fail, hiển thị 1 dòng giải thích lý do trước khi follow-up. User không bị "đột ngột bị hỏi sâu" mà không hiểu vì sao.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '25,35p' skills/okr-init/SKILL.md
```

Expected: bảng "Hành vi theo kết quả" với row `| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |`.

- [ ] **Step 2: Sửa row "Bất kỳ fail"**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |
```

new_string:
```
| Bất kỳ fail | Trước follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Vd: `(Mình đào sâu thêm vì "tăng doanh thu" chưa nói kênh nào, sản phẩm nào.)`. Sau đó dùng kỹ thuật phù hợp (xem Deepening Techniques). |
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -nE "Mình đào sâu thêm vì.*lý do cụ thể|Trước follow-up, in 1 dòng" skills/okr-init/SKILL.md
```

Expected: 2 match (cùng row).

Run:
```bash
grep -n "Bất kỳ fail | Follow-up ngay, dùng kỹ thuật" skills/okr-init/SKILL.md
```

Expected: KHÔNG có match (đã thay).

- [ ] **Step 4: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): make Quality Gate transparent on fail (M3)

Quality Gate chạy ngầm, follow-up đột ngột làm user không hiểu
vì sao bị hỏi sâu. Khi fail, in 1 dòng "(Mình đào sâu thêm vì
<lý do cụ thể>)" trước follow-up. Mâu thuẫn "trust user vs ẩn"
được giải bằng cách show lý do chứ không bỏ Quality Gate.

Refs: deep-insight/okr-skill-review-260510.md M3
EOF
)"
```

---

## Task 10: M3.b — `okr-plan/SKILL.md` Quality Gate transparent

**Files:**
- Modify: `skills/okr-plan/SKILL.md:25-30` (bảng "Hành vi theo kết quả")

**Mục tiêu:** Đồng bộ logic Quality Gate transparent với okr-init.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '23,33p' skills/okr-plan/SKILL.md
```

Expected: bảng "Hành vi theo kết quả" với row `| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |`.

- [ ] **Step 2: Sửa row "Bất kỳ fail"**

Edit `skills/okr-plan/SKILL.md`:

old_string:
```
| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |
```

new_string:
```
| Bất kỳ fail | Trước follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Vd: `(Mình đào sâu thêm vì action "Nghiên cứu thêm" chưa nói output là gì.)`. Sau đó dùng kỹ thuật phù hợp (xem Deepening Techniques). |
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -nE "Mình đào sâu thêm vì.*lý do cụ thể|Trước follow-up, in 1 dòng" skills/okr-plan/SKILL.md
```

Expected: 2 match.

Run:
```bash
grep -n "Bất kỳ fail | Follow-up ngay, dùng kỹ thuật" skills/okr-plan/SKILL.md
```

Expected: KHÔNG có match.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-plan/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-plan): make Quality Gate transparent on fail (M3)

Đồng bộ với okr-init: khi Quality Gate fail, in 1 dòng "(Mình
đào sâu thêm vì <lý do>)" trước follow-up. User hiểu vì sao
bị hỏi sâu thay vì bị đột ngột.

Refs: deep-insight/okr-skill-review-260510.md M3
EOF
)"
```

---

## Task 11: K2 — `okr-init/SKILL.md` mode `update-objective` Phase 4 SMART re-validate

**Files:**
- Modify: `skills/okr-init/SKILL.md:246-257` (Phase 4 CONFIRM diff)

**Mục tiêu:** Trước khi confirm diff, agent re-validate SMART cho KR/KI nào có thay đổi (target, baseline, period). Nếu fail → cảnh báo trong bảng confirm, user vẫn quyết định cuối.

- [ ] **Step 1: Verify state hiện tại**

Run:
```bash
sed -n '244,260p' skills/okr-init/SKILL.md
```

Expected: thấy "### Phase 4: CONFIRM diff (BẮT BUỘC)" với code block bảng "Thay đổi sắp áp dụng (objective.md)" 3 row, KHÔNG có section SMART check.

Run:
```bash
grep -n "okr-guide.md\|SMART" skills/okr-init/SKILL.md
```

Expected: ít nhất 1 match line ~330 ref `okr-guide.md` cho SMART check, không có check trong Phase 4 update-objective.

- [ ] **Step 2: Thay Phase 4 (thêm bước re-validate trước bảng confirm)**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------| 
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |

Xác nhận? (y / sửa / huỷ)
```
```

new_string:
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
```

- [ ] **Step 3: Cập nhật Phase 5 cũ (Ghi file) → Phase 6**

Edit `skills/okr-init/SKILL.md`:

old_string:
```
### Phase 5: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr` để check tác động sang plan."
```

new_string:
```
### Phase 6: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr` để check tác động sang plan."
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -nE "^### Phase 4: Re-validate SMART|^### Phase 5: CONFIRM diff|^### Phase 6: Ghi file" skills/okr-init/SKILL.md
```

Expected: 3 match (mỗi phase 1 dòng), nằm trong section "Mode UPDATE-OBJECTIVE" (sau "## Mode UPDATE-OBJECTIVE: sửa objective/KR/KI").

Run:
```bash
awk '/^## Mode UPDATE-OBJECTIVE/,/^## Mode UPDATE-RESOURCE/' skills/okr-init/SKILL.md | grep -E "^### "
```

Expected: thứ tự `Phase 1` → `Phase 2` → `Phase 3` → `Phase 4: Re-validate SMART` → `Phase 5: CONFIRM diff` → `Phase 6: Ghi file`.

Run:
```bash
grep -nE "Cảnh báo SMART \(KR/KI thay đổi\)|Specific.*Measurable.*Achievable" skills/okr-init/SKILL.md
```

Expected: ≥1 match cho mỗi pattern.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/SKILL.md
git commit -m "$(cat <<'EOF'
docs(okr-init): re-validate SMART on update-objective (K2)

SMART check chỉ chạy ở mode `new`. Khi user update KR/period
sau, không có check → KR mới có thể bay khỏi tiêu chí SMART
mà không ai cảnh báo. Thêm Phase 4 "Re-validate SMART" trước
Phase 5 CONFIRM diff trong mode update-objective. Cảnh báo
hiển thị trong bảng confirm, không block.

Refs: deep-insight/okr-skill-review-260510.md K2
EOF
)"
```

---

## Task 12: Final cross-check toàn đợt 2

**Mục tiêu:** Quét lại toàn repo đảm bảo không còn rác legacy `verifier`, `idea`/`note` type, "Solo hay team", "Người quản lý", và 11 commit nằm trên branch hiện tại.

- [ ] **Step 1: Quét legacy `verifier` field/section toàn skills/**

Run:
```bash
grep -rnE "verifier:|## Verifier|Ai verify output|verifier riêng" skills/
```

Expected: KHÔNG có match. (Mention "verify" trong text khác như "user tự verify qua DoD" được giữ.)

- [ ] **Step 2: Quét legacy capture type `idea`/`note` toàn skills/**

Run:
```bash
grep -rnE "type: action \| idea|type: idea \||\| \`idea\` \||\| \`note\` \|" skills/
```

Expected: KHÔNG có match.

Run:
```bash
grep -rnE "type: action \| blocker \| resource \| thought" skills/
```

Expected: ≥1 match (data-format.md).

- [ ] **Step 3: Quét legacy team flow trong okr-init**

Run:
```bash
grep -nE "Solo hay team\?|Bước 0: Solo|FTE|person-months|Họ tên, thông tin liên lạc \(nick Zalo|Mapping PIC vào actions|Đổi PIC hàng loạt|Thêm/sửa/xoá người" skills/okr-init/SKILL.md
```

Expected: KHÔNG có match.

Run:
```bash
grep -nE "Người quản lý|Họ Tên, Liên lạc|Zalo/FB/SĐT|Vai trò & Trách nhiệm|## Tham chiếu actions/|Cùng PIC có ≥3 actions|PIC khả dụng <50%|Update PIC PHẢI sync" skills/okr-init/references/data-format.md
```

Expected: KHÔNG có match.

- [ ] **Step 4: Quét Quality Gate transparent xuất hiện ở 2 nơi**

Run:
```bash
grep -rnE "Mình đào sâu thêm vì.*lý do cụ thể" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md
```

Expected: 2 match (1 ở mỗi file).

- [ ] **Step 5: Quét K1 logic ≤2/≥3 field trong okr-track**

Run:
```bash
grep -nE "≤2 field|≥3 field|y/n/sửa\?" skills/okr-track/SKILL.md
```

Expected: ≥3 match.

- [ ] **Step 6: Quét K2 SMART re-validate trong update-objective**

Run:
```bash
grep -nE "^### Phase 4: Re-validate SMART|Cảnh báo SMART \(KR/KI thay đổi\)" skills/okr-init/SKILL.md
```

Expected: 2 match.

- [ ] **Step 7: Verify lịch sử git**

Run:
```bash
git log --oneline -12
```

Expected: 11 commit gần nhất chứa các tag `(G10/K3)` × 3, `(K6)` × 2, `(K5)` × 2, `(K1)`, `(M3)` × 2, `(K2)` theo thứ tự task 1-11. Commit thứ 12 là commit cuối cùng của đợt 1 (M9.b hoặc xa hơn).

- [ ] **Step 8: Status sạch**

Run:
```bash
git status
```

Expected: `nothing to commit, working tree clean` (ngoại trừ untracked files đã có trước khi bắt đầu plan: `deep-insight/`, `docs/okr-system-review.md`, và file plan này).

---

## Tiêu chí hoàn thành Đợt 2

- [ ] 11 commit nhỏ trên branch hiện tại, mỗi commit reference review item.
- [ ] `grep "verifier"` toàn `skills/` ra rỗng (mention `verify` trong "user tự verify qua DoD" được giữ).
- [ ] `grep -rE "type: idea|type: note"` toàn `skills/` ra rỗng. Frontmatter type values là `action | blocker | resource | thought`.
- [ ] `grep "Solo hay team"`, `"FTE"`, `"person-months"`, `"Người quản lý"`, `"Họ Tên, Liên lạc"`, `"Vai trò & Trách nhiệm"` toàn `skills/` ra rỗng.
- [ ] Quality Gate "Mình đào sâu thêm vì <lý do>" xuất hiện ở `okr-init/SKILL.md` + `okr-plan/SKILL.md`.
- [ ] `okr-track/SKILL.md` Phase 4a Project có nhánh "≤2 field → 1 dòng / ≥3 field → bảng".
- [ ] `okr-init/SKILL.md` mode `update-objective` có Phase 4 "Re-validate SMART" trước Phase 5 "CONFIRM diff".
- [ ] `okr-init/references/data-format.md` schema body sections có `## Solo Profile` (Tên, Capacity giờ/tuần, Skills) thay cho `## Nhân sự`.

## Sau Đợt 2

Mở plan Đợt 3 (Lifecycle gaps, 8 items): G2/M9 Practice schema phụ thuộc Đợt 1 M9 + Đợt 2 K6 (Solo Profile capacity). G11 init quét inbox phụ thuộc Đợt 2 K5 (4 type). M4 impact check khi update-objective phụ thuộc Đợt 2 K2 (SMART re-validate đã làm Phase 4, M4 thêm Impact Check thành Phase 5 hoặc inline trong Phase 4).
