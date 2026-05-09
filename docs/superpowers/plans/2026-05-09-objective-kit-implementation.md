# Objective Kit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Xây dựng bộ 6 skill Claude Code quản lý mục tiêu & dự án theo OKR, lưu trữ bằng markdown + YAML frontmatter trong thư mục `.ok/`.

**Architecture:** Orchestrator (`/ok`) router tới 5 skill con (`ok-init`, `ok-plan`, `ok-resource`, `ok-track`, `ok-review`). Dữ liệu 2 tầng: SOT (trạng thái hiện tại) + Log (lịch sử). Shared reference file định nghĩa schema chung.

**Tech Stack:** Claude Code Skills (SKILL.md markdown), YAML frontmatter, Bash (cho verification)

**Spec:** `docs/superpowers/specs/2026-05-09-objective-kit-design.md`

## File Structure

```
objective-kit/
├── CLAUDE.md                           # Plugin-level instructions (ngắn)
├── skills/
│   ├── ok/
│   │   └── SKILL.md                    # Orchestrator router
│   ├── ok-init/
│   │   ├── SKILL.md                    # Khởi tạo mục tiêu
│   │   └── references/
│   │       └── okr-guide.md            # Hướng dẫn viết OKR + SMART
│   ├── ok-plan/
│   │   ├── SKILL.md                    # Lập kế hoạch hành động
│   │   └── references/
│   │       └── task-format.md          # Template action file
│   ├── ok-resource/
│   │   ├── SKILL.md                    # Quản lý resource & vai trò
│   │   └── references/
│   │       └── role-matrix.md          # Template resource file
│   ├── ok-track/
│   │   ├── SKILL.md                    # Tracking tiến độ
│   │   └── references/
│   │       └── metrics.md              # Cách tính metrics + log format
│   ├── ok-review/
│   │   └── SKILL.md                    # Lookback & điều chỉnh
│   └── shared/
│       └── data-format.md              # Schema chung YAML frontmatter
```

---

### Task 1: Khởi tạo repo + shared data format

**Files:**
- Create: `CLAUDE.md`
- Create: `skills/shared/data-format.md`

- [ ] **Step 1: Khởi tạo git repo**

```bash
cd /Users/tuanhv/Desktop/git_projects/objective-kit
git init
```

- [ ] **Step 2: Tạo CLAUDE.md**

```markdown
# Objective Kit

Bộ skill quản lý mục tiêu & dự án theo OKR.

## Cấu trúc

- `/ok` : Orchestrator, tự phát hiện phase phù hợp
- `/ok-init` : Khởi tạo mục tiêu (Project hoặc Habit)
- `/ok-plan` : Lập kế hoạch hành động
- `/ok-resource` : Quản lý tài nguyên & vai trò
- `/ok-track` : Theo dõi tiến độ
- `/ok-review` : Lookback & điều chỉnh

## Quy ước dữ liệu

Mọi skill đọc/ghi trong thư mục `.ok/` của dự án hiện tại. Xem `skills/shared/data-format.md` cho schema chi tiết.
```

- [ ] **Step 3: Tạo `skills/shared/data-format.md`**

File này định nghĩa schema YAML frontmatter cho mọi file trong `.ok/`. Mọi skill reference file này để đảm bảo format thống nhất.

```markdown
# Data Format: Schema YAML Frontmatter

Mọi file trong `.ok/` dùng markdown + YAML frontmatter. Skill đọc frontmatter để hiểu trạng thái nhanh, đọc body khi cần chi tiết.

## Hai tầng dữ liệu

| Tầng | Vị trí | Hành vi |
|------|--------|---------|
| **SOT** (Source of Truth) | `.ok/objective.md`, `.ok/plan.md`, `.ok/actions/`, `.ok/resources.md` | Ghi đè khi cập nhật. Luôn phản ánh trạng thái mới nhất |
| **Log** | `.ok/log/YYYY-MM-DD.md`, `.ok/log/reviews/` | Append-only. Ghi lịch sử thay đổi |

Nguyên tắc: SOT = hiện tại. Log = quá khứ.

## objective.md

### Project

~~~yaml
---
type: project
objective: "string"
quarter: "Q1-2026"
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
status: active | completed | paused | cancelled
---
~~~

Body chứa: `## Objective` (WHY), `## Key Results` (bảng: #, Key Result, Baseline, Target, Current, Status), `## Context`.

KR Status: `pending` | `in-progress` | `achieved` | `missed`

### Habit

~~~yaml
---
type: habit
objective: "string"
start_date: YYYY-MM-DD
frequency: daily | weekly | monthly
status: active | paused
---
~~~

Body chứa: `## Objective` (WHY), `## Key Indicators` (bảng: #, Chỉ số, Tần suất, Streak hiện tại, Streak dài nhất), `## Checklist lặp lại`.

## plan.md

~~~yaml
---
total_actions: int
completed: int
in_progress: int
blocked: int
milestones:
  - name: "string"
    target_date: YYYY-MM-DD
    key_results: [KR1, KR2]
    status: pending | in-progress | done
---
~~~

Body: `## Roadmap` với các heading milestone, mỗi milestone liệt kê actions.

## actions/AXXX-slug.md

~~~yaml
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
~~~

Body: `## Definition of Done` (checklist), `## Output/Deliverable`, `## Ghi chú`.

## resources.md

~~~yaml
---
last_updated: YYYY-MM-DD
---
~~~

Body: `## Vai trò & Trách nhiệm` (bảng: Tên, Vai trò, Trách nhiệm, Actions, Khả dụng), `## Công cụ & Tài liệu`, `## Ngân sách`, `## Thiếu hụt & Rủi ro`.

## log/YYYY-MM-DD.md

~~~yaml
---
date: YYYY-MM-DD
type: tracking
---
~~~

Body: `## Thay đổi` (danh sách thay đổi), `## Ghi chú`.

## log/reviews/YYYY-MM-DD.md

~~~yaml
---
date: YYYY-MM-DD
type: review
period: "YYYY-MM-DD to YYYY-MM-DD"
---
~~~

Body: `## Tổng kết` (bảng KR), `## Phân tích` (Đạt tốt, Cần cải thiện), `## Điều chỉnh đã áp dụng`.
```

- [ ] **Step 4: Tạo .gitignore**

```
.DS_Store
```

- [ ] **Step 5: Verify cấu trúc**

```bash
ls -la CLAUDE.md skills/shared/data-format.md
```

Expected: cả 2 file tồn tại.

- [ ] **Step 6: Commit**

```bash
git add CLAUDE.md skills/shared/data-format.md .gitignore
git commit -m "feat: init repo + shared data format schema"
```

---

### Task 2: Skill ok-init (khởi tạo mục tiêu)

**Files:**
- Create: `skills/ok-init/SKILL.md`
- Create: `skills/ok-init/references/okr-guide.md`

- [ ] **Step 1: Tạo `skills/ok-init/references/okr-guide.md`**

Reference file chứa hướng dẫn viết OKR tốt + checklist SMART.

```markdown
# Hướng dẫn viết OKR

## Objective (mục tiêu)

Objective là câu trả lời cho "Chúng ta muốn đạt được điều gì?"

Tiêu chí:
- Định tính (không có số)
- Truyền cảm hứng, tạo động lực
- Rõ ràng, ai đọc cũng hiểu
- Có thời hạn (quarter hoặc custom)

Ví dụ tốt: "Trở thành nguồn tài liệu AI hàng đầu cho cộng đồng Việt Nam"
Ví dụ xấu: "Tăng 50% lượt truy cập website" (đây là Key Result, không phải Objective)

## Key Results (kết quả then chốt)

Key Result trả lời "Làm sao biết đã đạt Objective?"

Tiêu chí SMART:
- **S**pecific: Cụ thể, không mơ hồ
- **M**easurable: Đo được bằng số (Baseline → Target)
- **A**chievable: Khả thi với resource hiện có (stretch nhưng không ảo)
- **R**elevant: Liên quan trực tiếp đến Objective
- **T**ime-bound: Có deadline rõ

Số lượng: 3-5 KR mỗi Objective. Ít hơn 3 thì thiếu góc nhìn. Nhiều hơn 5 thì mất focus.

Ví dụ:
- KR1: Xuất bản 12 bài deep-dive (baseline: 0, target: 12)
- KR2: Đạt 5000 subscriber newsletter (baseline: 200, target: 5000)
- KR3: 3 bài được trích dẫn bởi nguồn uy tín (baseline: 0, target: 3)

## Phân biệt Project vs Habit

| Câu hỏi | Project | Habit |
|----------|---------|-------|
| Có điểm kết thúc? | Có | Không |
| Đo bằng gì? | Key Results (target số) | Key Indicators (streak, tần suất) |
| Khi nào "xong"? | Đạt target hoặc hết deadline | Không bao giờ "xong" |

Nếu user chưa rõ, hỏi: "Mục tiêu này có ngày kết thúc không?"
```

- [ ] **Step 2: Tạo `skills/ok-init/SKILL.md`**

```markdown
---
name: ok-init
description: "Khởi tạo mục tiêu dự án theo OKR. Dùng khi bắt đầu dự án mới hoặc khi chưa có thư mục .ok/ trong folder hiện tại. Trigger: /ok-init, hoặc tự động khi /ok phát hiện chưa có .ok/"
---

# ok-init: Khởi tạo mục tiêu

Tạo `.ok/objective.md` cho dự án hiện tại. Hỗ trợ 2 loại: Project (có kỳ hạn) và Habit (thói quen).

## Checklist

1. Kiểm tra `.ok/` đã tồn tại chưa. Nếu rồi, hỏi user muốn tạo mới hay cập nhật
2. Hỏi loại mục tiêu: Project hay Habit
3. Thu thập thông tin:
   - **WHY**: Tại sao mục tiêu này quan trọng?
   - **HOW**: Cách tiếp cận tổng quan
   - **WHAT**: Kết quả cụ thể mong đợi
4. Viết Objective (hỏi 1 câu tại 1 thời điểm, không hỏi nhiều câu cùng lúc)
5. Viết Key Results (Project) hoặc Key Indicators (Habit). Đọc `references/okr-guide.md` để kiểm tra chất lượng
6. Kiểm tra SMART cho mỗi KR/KI
7. Tạo thư mục `.ok/` và file `objective.md`

## Format output

Đọc `../shared/data-format.md` phần `objective.md` để lấy schema chính xác.

### Project

~~~yaml
---
type: project
objective: "[tóm tắt ngắn]"
quarter: "[Q?-YYYY]"
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
status: active
---
~~~

Body: `## Objective` (mô tả WHY chi tiết), `## Key Results` (bảng), `## Context` (bối cảnh).

### Habit

~~~yaml
---
type: habit
objective: "[tóm tắt ngắn]"
start_date: YYYY-MM-DD
frequency: daily|weekly|monthly
status: active
---
~~~

Body: `## Objective` (mô tả WHY), `## Key Indicators` (bảng), `## Checklist lặp lại`.

## Quy tắc

- Hỏi từng câu một, không hỏi hàng loạt
- Đề xuất Objective và KR, nhưng luôn để user quyết
- Nếu KR không đạt SMART, chỉ ra cụ thể thiếu tiêu chí nào và gợi ý sửa
- Tạo `.ok/` folder nếu chưa có
- Chỉ tạo `objective.md`. Không tạo plan.md hay file khác
```

- [ ] **Step 3: Verify file tồn tại**

```bash
ls -la skills/ok-init/SKILL.md skills/ok-init/references/okr-guide.md
```

Expected: cả 2 file tồn tại.

- [ ] **Step 4: Commit**

```bash
git add skills/ok-init/
git commit -m "feat: add ok-init skill (khởi tạo mục tiêu OKR)"
```

---

### Task 3: Skill ok-plan (lập kế hoạch hành động)

**Files:**
- Create: `skills/ok-plan/SKILL.md`
- Create: `skills/ok-plan/references/task-format.md`

- [ ] **Step 1: Tạo `skills/ok-plan/references/task-format.md`**

```markdown
# Task Format: Template action file

## Tạo file action

Mỗi action = 1 file trong `.ok/actions/`. Tên file: `AXXX-slug.md` (XXX = số thứ tự 3 chữ số, slug = tên ngắn dùng dấu gạch ngang).

Ví dụ: `A001-thiet-ke-database.md`, `A002-viet-api-auth.md`

## Template

~~~yaml
---
id: AXXX
title: "Tên task rõ ràng"
description: "1-2 câu mô tả task cần làm"
key_result: KR1
milestone: "Tên milestone"
status: pending
priority: high
effort: m
pic: ""
due_date: YYYY-MM-DD
depends_on: []
---
~~~

~~~markdown
## Definition of Done

- [ ] [Tiêu chí 1: cụ thể, verify được]
- [ ] [Tiêu chí 2]

## Output/Deliverable

[Mô tả sản phẩm đầu ra khi task hoàn thành. Phải cụ thể: file gì, ở đâu, chứa gì]

## Ghi chú

[Context bổ sung, ràng buộc, link tham khảo]
~~~

## Quy tắc đặt tên

- ID bắt đầu từ A001, tăng dần
- Slug dùng tiếng Việt không dấu hoặc tiếng Anh, dấu gạch ngang
- Mỗi task phải map vào đúng 1 Key Result
- Mỗi task phải thuộc 1 milestone

## Priority values

| Value | Khi nào dùng |
|-------|-------------|
| critical | Chặn toàn bộ dự án nếu không xong |
| high | Cần xong sớm, ảnh hưởng nhiều task khác |
| medium | Quan trọng nhưng không chặn ai |
| low | Làm khi rảnh, nice-to-have |

## Effort values

| Value | Thời gian ước lượng |
|-------|-------------------|
| xs | < 1 giờ |
| s | 1-4 giờ |
| m | 1-2 ngày |
| l | 3-5 ngày |
| xl | > 1 tuần |
```

- [ ] **Step 2: Tạo `skills/ok-plan/SKILL.md`**

```markdown
---
name: ok-plan
description: "Lập kế hoạch hành động cho dự án. Phân tách Key Results thành milestones và actions cụ thể. Dùng khi có objective nhưng chưa có plan. Trigger: /ok-plan, hoặc tự động khi /ok phát hiện thiếu plan.md"
---

# ok-plan: Lập kế hoạch hành động

Đọc `.ok/objective.md`, phân tách Key Results thành milestones và actions, tạo `.ok/plan.md` + `.ok/actions/`.

## Điều kiện tiên quyết

- `.ok/objective.md` phải tồn tại. Nếu chưa có, hướng user chạy `/ok-init` trước.

## Checklist

1. Đọc `.ok/objective.md` để hiểu Objective và Key Results
2. Với mỗi Key Result, đề xuất Initiatives (sáng kiến lớn)
3. Với mỗi Initiative, phân tách thành Actions cụ thể
4. Nhóm actions thành Milestones (theo thời gian)
5. Xác định dependencies giữa actions
6. Hỏi user về deadline milestones
7. Hỏi user về PIC (Person In Charge) cho từng action nếu có nhiều người
8. Tạo `.ok/plan.md` với roadmap tổng
9. Tạo file cho mỗi action trong `.ok/actions/`

## Format output

Đọc `../shared/data-format.md` phần `plan.md` và `actions/` để lấy schema.
Đọc `references/task-format.md` để lấy template action file.

## Quy tắc

- Mỗi action phải có Definition of Done rõ ràng
- Mỗi action phải có Output/Deliverable cụ thể
- Không tạo action mơ hồ kiểu "Nghiên cứu thêm" mà không có output
- Dependencies phải hợp lệ (ID tồn tại, không vòng tròn)
- Cập nhật `total_actions` trong frontmatter plan.md
- Nếu objective.md type=habit, plan.md chứa recurring tasks thay vì milestones
```

- [ ] **Step 3: Verify**

```bash
ls -la skills/ok-plan/SKILL.md skills/ok-plan/references/task-format.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/ok-plan/
git commit -m "feat: add ok-plan skill (lập kế hoạch hành động)"
```

---

### Task 4: Skill ok-resource (quản lý tài nguyên & vai trò)

**Files:**
- Create: `skills/ok-resource/SKILL.md`
- Create: `skills/ok-resource/references/role-matrix.md`

- [ ] **Step 1: Tạo `skills/ok-resource/references/role-matrix.md`**

```markdown
# Role Matrix: Template resource file

## Bảng vai trò & trách nhiệm

| Cột | Mô tả |
|-----|-------|
| Tên | Tên người/nhóm |
| Vai trò | Chức danh hoặc vai trò trong dự án |
| Trách nhiệm | Cụ thể: làm gì, quyết định gì, chịu trách nhiệm gì |
| Actions | Danh sách ID action phụ trách (A001, A003...) |
| Khả dụng | % thời gian dành cho dự án này |

## RACI (khi cần chi tiết hơn)

Nếu dự án có nhiều người, dùng RACI cho các quyết định quan trọng:

| | Người A | Người B | Người C |
|---|---|---|---|
| Quyết định X | R (Responsible) | A (Accountable) | I (Informed) |
| Quyết định Y | C (Consulted) | R | A |

- **R**: Người thực hiện
- **A**: Người chịu trách nhiệm cuối cùng (chỉ 1 người)
- **C**: Người cần hỏi ý kiến trước khi làm
- **I**: Người cần thông báo sau khi xong

## Phát hiện xung đột

Kiểm tra:
- Cùng 1 người phụ trách quá nhiều action cùng lúc?
- Action không có PIC?
- Milestone cần resource chưa sẵn sàng?
- Effort tổng vượt khả dụng?
```

- [ ] **Step 2: Tạo `skills/ok-resource/SKILL.md`**

```markdown
---
name: ok-resource
description: "Quản lý tài nguyên, vai trò và trách nhiệm trong dự án. Mapping resource vào actions, phát hiện thiếu hụt và xung đột. Trigger: /ok-resource"
---

# ok-resource: Quản lý tài nguyên & vai trò

Tạo/cập nhật `.ok/resources.md`. Mapping nhân sự, công cụ, ngân sách vào actions cụ thể.

## Điều kiện tiên quyết

- `.ok/plan.md` và `.ok/actions/` phải tồn tại. Nếu chưa, hướng user chạy `/ok-plan`.

## Checklist

1. Đọc `.ok/plan.md` và frontmatter tất cả `.ok/actions/*.md`
2. Liệt kê actions chưa có PIC
3. Hỏi user về nhân sự: tên, vai trò, trách nhiệm, khả dụng
4. Mapping PIC vào actions (cập nhật frontmatter `pic` trong action files)
5. Hỏi user về công cụ, tài liệu, ngân sách
6. Kiểm tra xung đột (đọc `references/role-matrix.md` phần phát hiện xung đột)
7. Tạo/cập nhật `.ok/resources.md`

## Format output

Đọc `../shared/data-format.md` phần `resources.md` để lấy schema.

## Quy tắc

- Khi cập nhật PIC, sửa cả frontmatter trong `actions/*.md` (SOT)
- Mỗi action có deadline mà PIC khả dụng < 50% cần cảnh báo
- Nếu phát hiện xung đột, đề xuất giải pháp cụ thể (dời deadline, tách task, thêm người)
- File resources.md là SOT: luôn phản ánh trạng thái mới nhất, không chứa lịch sử
```

- [ ] **Step 3: Verify**

```bash
ls -la skills/ok-resource/SKILL.md skills/ok-resource/references/role-matrix.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/ok-resource/
git commit -m "feat: add ok-resource skill (quản lý tài nguyên & vai trò)"
```

---

### Task 5: Skill ok-track (theo dõi tiến độ)

**Files:**
- Create: `skills/ok-track/SKILL.md`
- Create: `skills/ok-track/references/metrics.md`

- [ ] **Step 1: Tạo `skills/ok-track/references/metrics.md`**

```markdown
# Metrics: Cách tính và log format

## Tiến độ Key Result

Công thức: `% = (Current - Baseline) / (Target - Baseline) * 100`

Cập nhật Current bằng 2 cách:
1. **User tự nhập** (ưu tiên): user cung cấp giá trị mới cho KR
2. **Tính từ actions**: đếm actions `done` thuộc KR đó / tổng actions thuộc KR

Ưu tiên cách 1 vì KR đo outcome (kết quả), không phải output (sản lượng).

## Trend

| Trend | Điều kiện |
|-------|----------|
| on-track | Tiến độ thực >= tiến độ kỳ vọng theo timeline |
| at-risk | Tiến độ thực < kỳ vọng nhưng chênh < 20% |
| off-track | Chênh >= 20% hoặc có actions blocked |

Tiến độ kỳ vọng = % thời gian đã trôi trong period.

Ví dụ: period 3 tháng, đã qua 1.5 tháng = kỳ vọng 50%. KR đạt 35% → at-risk.

## Cập nhật SOT

Khi track, cập nhật:
1. `objective.md`: cột Current và Status trong bảng KR
2. `plan.md`: frontmatter `completed`, `in_progress`, `blocked`
3. Mỗi action có thay đổi: frontmatter `status`

## Log format

Mỗi lần track = 1 entry trong `log/YYYY-MM-DD.md`. Nếu file ngày đó đã có, append thêm section mới.

Log ghi:
- Giá trị thay đổi (cũ → mới)
- Actions thay đổi status
- Blockers mới phát hiện
- Ghi chú của user
```

- [ ] **Step 2: Tạo `skills/ok-track/SKILL.md`**

```markdown
---
name: ok-track
description: "Theo dõi tiến độ dự án. Cập nhật Key Results, actions, phát hiện trễ và blockers. Trigger: /ok-track, hoặc tự động khi /ok phát hiện dự án đang thực thi"
---

# ok-track: Theo dõi tiến độ

Đọc SOT files, nhận update từ user, cập nhật SOT + ghi log.

## Điều kiện tiên quyết

- `.ok/objective.md` và `.ok/plan.md` phải tồn tại.

## Checklist

1. Đọc `.ok/objective.md` (KR hiện tại)
2. Đọc frontmatter tất cả `.ok/actions/*.md` (status)
3. Hiển thị dashboard ngắn:
   - Mỗi KR: Current/Target, %, trend
   - Actions: tổng, done, doing, blocked
   - Highlight: actions chậm deadline, blocked
4. Hỏi user có update gì mới (tiến độ, hoàn thành task, blockers)
5. Cập nhật SOT files (đọc `references/metrics.md` cho cách tính)
6. Tạo thư mục `.ok/log/` nếu chưa có
7. Ghi entry vào `.ok/log/YYYY-MM-DD.md`

## Dashboard format

Hiển thị khi bắt đầu track:

```
📊 Tiến độ dự án: [Tên Objective]

Key Results:
  KR1: ████░░░░░░ 40/100 (40%) ▶ on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ⚠ at-risk

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending

⚠ Cần chú ý:
  - A005 blocked: [lý do]
  - A007 quá hạn 3 ngày
```

## Quy tắc

- Luôn hiển thị dashboard trước khi hỏi update
- Sau khi nhận update, cập nhật SOT files ngay (ghi đè giá trị mới)
- Ghi log mọi thay đổi vào `.ok/log/YYYY-MM-DD.md`
- Nếu log ngày hôm nay đã có, append section mới (không ghi đè log)
- Với habit type: hiển thị streak thay vì %, checklist thay vì actions
```

- [ ] **Step 3: Verify**

```bash
ls -la skills/ok-track/SKILL.md skills/ok-track/references/metrics.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/ok-track/
git commit -m "feat: add ok-track skill (theo dõi tiến độ)"
```

---

### Task 6: Skill ok-review (lookback & điều chỉnh)

**Files:**
- Create: `skills/ok-review/SKILL.md`

- [ ] **Step 1: Tạo `skills/ok-review/SKILL.md`**

```markdown
---
name: ok-review
description: "Lookback & điều chỉnh dự án. Phân tích tiến độ tổng thể, tìm root cause trễ, đề xuất điều chỉnh plan. Trigger: /ok-review"
---

# ok-review: Lookback & điều chỉnh

Đọc SOT + log, phân tích toàn diện, đề xuất và áp dụng điều chỉnh.

## Điều kiện tiên quyết

- `.ok/objective.md`, `.ok/plan.md`, `.ok/actions/` phải tồn tại.

## Checklist

1. Đọc `.ok/objective.md` + `.ok/plan.md` (SOT: trạng thái hiện tại)
2. Đọc `.ok/log/*.md` (Log: lịch sử thay đổi theo thời gian)
3. Tính tổng kết:
   - Mỗi KR: Target vs Current, %, trend
   - Timeline: % thời gian đã dùng vs % tiến độ thực
4. Phân tích (đọc `shared/data-format.md` phần `log/reviews/`):
   - Cái gì đạt tốt, tại sao
   - Cái gì trượt, root cause (hỏi "tại sao?" ít nhất 3 lần, không dừng ở triệu chứng)
   - Actions blocked lâu nhất, nguyên nhân
   - Tốc độ hoàn thành actions: nhanh hơn hay chậm hơn kế hoạch
5. Đề xuất điều chỉnh cụ thể:
   - Thay đổi target KR (tăng/giảm)
   - Thêm/bớt/sửa actions
   - Dời deadline milestones
   - Thay đổi PIC, thêm resource
6. Hỏi user từng đề xuất (đồng ý/từ chối/sửa)
7. Áp dụng điều chỉnh: cập nhật SOT files
8. Tạo thư mục `.ok/log/reviews/` nếu chưa có
9. Ghi log vào `.ok/log/reviews/YYYY-MM-DD.md`

## Format review

Hiển thị trước khi đề xuất:

```
📋 Review dự án: [Tên Objective]
Period: YYYY-MM-DD → YYYY-MM-DD

Tổng kết KR:
| KR | Target | Current | % | Trend |
|----|--------|---------|---|-------|
| KR1 | 100 | 45 | 45% | on-track |
| KR2 | 50 | 10 | 20% | at-risk |

Timeline: 60% thời gian đã dùng | 35% tiến độ trung bình

Đạt tốt:
  - [Phân tích]

Cần cải thiện:
  - [Phân tích + root cause]
```

## Quy tắc

- Phân tích root cause, không dừng ở triệu chứng
- Mỗi đề xuất điều chỉnh phải kèm lý do
- Hỏi user confirm từng điều chỉnh trước khi sửa SOT
- Log review ghi đầy đủ: tổng kết, phân tích, điều chỉnh đã áp dụng
- Nếu objective type=habit: review streak, adherence rate, đề xuất điều chỉnh frequency hoặc checklist
```

- [ ] **Step 2: Verify**

```bash
ls -la skills/ok-review/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add skills/ok-review/
git commit -m "feat: add ok-review skill (lookback & điều chỉnh)"
```

---

### Task 7: Skill ok (orchestrator router)

**Files:**
- Create: `skills/ok/SKILL.md`

- [ ] **Step 1: Tạo `skills/ok/SKILL.md`**

```markdown
---
name: ok
description: "Quản lý mục tiêu & dự án theo OKR. Entry point chính, tự phát hiện phase phù hợp. Gọi /ok để bắt đầu hoặc tiếp tục dự án. Gọi /ok init|plan|resource|track|review để vào phase cụ thể."
---

# ok: Orchestrator quản lý dự án

Entry point chính. Phân tích trạng thái `.ok/` rồi route đến skill phù hợp.

## Logic routing

Khi user gọi `/ok` không kèm tham số:

1. `.ok/` chưa tồn tại → thông báo "Chưa có mục tiêu. Dùng `/ok-init` để khởi tạo."
2. `.ok/objective.md` chưa có → thông báo tương tự
3. `.ok/plan.md` chưa có → thông báo "Có mục tiêu nhưng chưa có kế hoạch. Dùng `/ok-plan` để lập plan."
4. Đọc frontmatter tất cả `.ok/actions/*.md`:
   - Tất cả status=done → thông báo "Mọi action hoàn thành. Dùng `/ok-review` để review tổng kết."
   - Có status=blocked → hiển thị blockers + hỏi user muốn `/ok-track` hay `/ok-review`
   - Có actions đang thực thi → thông báo "Dự án đang thực thi. Dùng `/ok-track` để cập nhật tiến độ."

Khi user gọi `/ok <phase>`:

- `/ok init` → hướng dẫn dùng `/ok-init`
- `/ok plan` → hướng dẫn dùng `/ok-plan`
- `/ok resource` → hướng dẫn dùng `/ok-resource`
- `/ok track` → hướng dẫn dùng `/ok-track`
- `/ok review` → hướng dẫn dùng `/ok-review`

## Hiển thị khi routing

Trước khi route, hiển thị trạng thái ngắn gọn:

```
🎯 Dự án: [Objective]
📅 Period: [start] → [end]
📊 KR: X/Y đạt | Actions: X done, Y doing, Z blocked
➡️ Đề xuất: [phase phù hợp]
```

## Quy tắc

- Skill này CHỈ phân tích và route. Không tạo/sửa file nào
- Đọc frontmatter tối thiểu (objective.md + glob actions/*.md)
- Luôn hiển thị trạng thái ngắn trước khi đề xuất phase
```

- [ ] **Step 2: Verify toàn bộ cấu trúc**

```bash
find skills/ -name "SKILL.md" -o -name "*.md" | sort
```

Expected output:
```
skills/ok/SKILL.md
skills/ok-init/SKILL.md
skills/ok-init/references/okr-guide.md
skills/ok-plan/SKILL.md
skills/ok-plan/references/task-format.md
skills/ok-resource/SKILL.md
skills/ok-resource/references/role-matrix.md
skills/ok-review/SKILL.md
skills/ok-track/SKILL.md
skills/ok-track/references/metrics.md
skills/shared/data-format.md
```

- [ ] **Step 3: Commit**

```bash
git add skills/ok/
git commit -m "feat: add ok orchestrator skill (router)"
```

---

### Task 8: Verification tổng thể

**Files:** Không tạo file mới. Verify toàn bộ.

- [ ] **Step 1: Verify cấu trúc thư mục**

```bash
cd /Users/tuanhv/Desktop/git_projects/objective-kit
find . -type f -not -path './.git/*' | sort
```

Expected: tất cả files từ file structure ở đầu plan.

- [ ] **Step 2: Verify mỗi SKILL.md có frontmatter hợp lệ**

```bash
for f in $(find skills/ -name "SKILL.md"); do
  echo "=== $f ==="
  head -4 "$f"
  echo ""
done
```

Expected: mỗi SKILL.md bắt đầu bằng `---`, có `name:` và `description:`.

- [ ] **Step 3: Verify shared data-format được reference**

```bash
grep -r "data-format" skills/ --include="*.md"
```

Expected: nhiều skill reference đến `shared/data-format.md`.

- [ ] **Step 4: Verify cross-references nhất quán**

```bash
grep -r "references/" skills/ --include="*.md"
```

Expected: mỗi skill reference đúng file trong thư mục `references/` của nó.

- [ ] **Step 5: Verify git log**

```bash
git log --oneline
```

Expected: 7 commits (1 init + 6 skill tasks).

- [ ] **Step 6: Commit verification**

Không commit thêm. Task này chỉ verify.
