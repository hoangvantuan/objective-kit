# Objective Kit: Bộ skill quản lý mục tiêu & dự án

## Tổng quan

**Objective Kit** là bộ skill quản lý mục tiêu và dự án theo framework OKR. Gồm nhiều skill độc lập, mỗi skill phụ trách 1 phase trong vòng đời dự án.

**Đối tượng**: Cá nhân hoặc nhóm (nhiều người).
**Quy mô**: Không giới hạn, từ dự án nhỏ đến dài hạn.
**Lưu trữ**: Markdown + YAML frontmatter trong thư mục `.ok/` của dự án.

## WHY: Vấn đề cần giải quyết

Claude Code hoạt động theo session. Mỗi session mới mất context về mục tiêu, tiến độ, kế hoạch. Cần persistence layer giúp:

1. Xác định rõ mục tiêu (không trôi hướng giữa chừng)
2. Phân tách mục tiêu thành hành động cụ thể, đo được
3. Theo dõi tiến độ qua nhiều session
4. Quản lý resource, vai trò, trách nhiệm
5. Review và điều chỉnh khi thực tế thay đổi

## HOW: Kiến trúc

### Bộ skill (mỗi phase = 1 skill riêng)

```
objective-kit/
├── skills/
│   ├── ok/
│   │   └── SKILL.md            # Orchestrator: router tự chọn phase
│   ├── ok-init/
│   │   ├── SKILL.md            # Khởi tạo mục tiêu
│   │   └── references/
│   │       └── okr-guide.md
│   ├── ok-plan/
│   │   ├── SKILL.md            # Lập kế hoạch hành động
│   │   └── references/
│   │       └── task-format.md
│   ├── ok-resource/
│   │   ├── SKILL.md            # Quản lý resource & vai trò
│   │   └── references/
│   │       └── role-matrix.md
│   ├── ok-track/
│   │   ├── SKILL.md            # Tracking tiến độ
│   │   └── references/
│   │       └── metrics.md
│   ├── ok-review/
│   │   └── SKILL.md            # Lookback & điều chỉnh
│   └── shared/
│       └── data-format.md      # Schema chung cho YAML frontmatter
```

User gọi skill trực tiếp (`/ok-plan`, `/ok-track`) hoặc qua orchestrator (`/ok`).

### Hai loại mục tiêu

| | Project (có kỳ hạn) | Habit (thói quen) |
|---|---|---|
| Tính chất | Có điểm bắt đầu và kết thúc | Liên tục, không kết thúc |
| Đo lường | Key Results với target cụ thể | Tần suất thực hiện, streak |
| Ví dụ | "Ra mắt MVP trước 30/7" | "Viết blog mỗi tuần" |
| Khi nào xong | Đạt target hoặc hết deadline | Không bao giờ "xong", chỉ duy trì |
| Actions | Tasks có deadline, output, DoD | Recurring tasks, checklist lặp lại |

### Hai tầng dữ liệu

**Tầng 1: Source of Truth (SOT)**

Luôn phản ánh trạng thái mới nhất. Không chứa lịch sử. Khi thay đổi, file ghi đè giá trị mới.

```
<project>/.ok/
├── objective.md            # Objective + Key Results (giá trị hiện tại)
├── plan.md                 # Roadmap, milestones (trạng thái hiện tại)
├── actions/                # Mỗi action = trạng thái hiện tại
│   ├── A001-ten-task.md
│   └── A002-ten-task.md
└── resources.md            # Vai trò, resource allocation hiện tại
```

**Tầng 2: Log (lịch sử thay đổi)**

Ghi theo thời gian. Append-only. Dùng khi cần xem lại quá trình.

```
<project>/.ok/
└── log/
    ├── 2026-05-09.md       # Thay đổi ngày này
    ├── 2026-05-16.md
    └── reviews/
        └── 2026-06-01.md   # Phiên review chính thức
```

**Nguyên tắc**: SOT = đọc để biết hiện tại. Log = đọc để hiểu quá khứ.

### Workflow cập nhật dữ liệu

Khi user cung cấp thông tin mới (tiến độ, thay đổi, blocker):

1. Skill đọc SOT files liên quan
2. Cập nhật SOT files với giá trị mới (ghi đè)
3. Ghi 1 entry vào log/YYYY-MM-DD.md ghi nhận thay đổi

Khi user muốn xem lịch sử:

1. Skill đọc log/*.md theo khoảng thời gian
2. Tổng hợp progression qua thời gian

## WHAT: Chi tiết từng Skill

### ok-init: Khởi tạo mục tiêu

**Trigger**: Chưa có `.ok/` trong dự án, hoặc `/ok-init`

**Workflow**:
1. Hỏi loại mục tiêu: Project hay Habit
2. Hỏi Why/How/What của dự án
3. Hỗ trợ viết Objective (định tính, truyền cảm hứng)
4. Hỗ trợ viết Key Results (định lượng, đo được, có baseline + target)
5. Kiểm tra SMART cho mỗi Key Result
6. Tạo `.ok/objective.md`

**Format `objective.md` (Project)**:

```yaml
---
type: project
objective: "Mô tả Objective ngắn gọn"
quarter: Q3-2026
start_date: 2026-05-09
end_date: 2026-07-31
status: active
---
```

```markdown
## Objective

[Mô tả chi tiết WHY, tại sao mục tiêu này quan trọng]

## Key Results

| # | Key Result | Baseline | Target | Current | Status |
|---|-----------|----------|--------|---------|--------|
| KR1 | [Mô tả] | 0 | 100 | 0 | pending |
| KR2 | [Mô tả] | 0 | 50 | 0 | pending |
| KR3 | [Mô tả] | 0 | 10 | 0 | pending |

## Context

[Bối cảnh dự án, HOW/WHAT tổng quan]
```

**Format `objective.md` (Habit)**:

```yaml
---
type: habit
objective: "Mô tả thói quen muốn xây dựng"
start_date: 2026-05-09
frequency: weekly
status: active
---
```

```markdown
## Objective

[Mô tả WHY, tại sao thói quen này quan trọng]

## Key Indicators

| # | Chỉ số | Tần suất | Streak hiện tại | Streak dài nhất |
|---|--------|----------|-----------------|-----------------|
| KI1 | [Mô tả] | weekly | 0 | 0 |
| KI2 | [Mô tả] | daily | 0 | 0 |

## Checklist lặp lại

- [ ] [Hành động 1]
- [ ] [Hành động 2]
```

### ok-plan: Lập kế hoạch hành động

**Trigger**: Có objective.md nhưng chưa có plan.md, hoặc `/ok-plan`

**Workflow**:
1. Đọc objective.md
2. Phân tách mỗi Key Result thành Initiatives
3. Mỗi Initiative cụ thể hoá thành Tasks (Actions)
4. Xác định dependencies, thứ tự
5. Tạo plan.md (roadmap) + files trong actions/

**Format `plan.md`**:

```yaml
---
total_actions: 0
completed: 0
in_progress: 0
blocked: 0
milestones:
  - name: "Milestone 1"
    target_date: 2026-06-01
    key_results: [KR1, KR2]
    status: pending
---
```

```markdown
## Roadmap

### Milestone 1: [Tên] (target: 2026-06-01)

Liên quan: KR1, KR2

Actions:
- A001: [Tên action]
- A002: [Tên action]
```

**Format `actions/A001-ten-task.md`**:

```yaml
---
id: A001
title: "Tên task"
description: "Mô tả chi tiết task cần làm"
key_result: KR1
milestone: "Milestone 1"
status: pending
priority: high
effort: medium
pic: ""
due_date: 2026-05-20
depends_on: []
---
```

```markdown
## Definition of Done

- [ ] [Tiêu chí hoàn thành 1]
- [ ] [Tiêu chí hoàn thành 2]
- [ ] [Tiêu chí hoàn thành 3]

## Output/Deliverable

[Mô tả cụ thể sản phẩm đầu ra khi task hoàn thành]

## Ghi chú

[Thông tin bổ sung, context, ràng buộc]
```

**Status values**: `pending` | `doing` | `done` | `blocked`

### ok-resource: Quản lý tài nguyên & vai trò

**Trigger**: `/ok-resource`

**Workflow**:
1. Đọc plan.md và actions/ để hiểu cần gì
2. Hỏi user về nhân sự, vai trò, trách nhiệm
3. Mapping resource vào action cụ thể
4. Phát hiện thiếu hụt hoặc xung đột
5. Tạo/cập nhật resources.md

**Format `resources.md`**:

```yaml
---
last_updated: 2026-05-09
---
```

```markdown
## Vai trò & Trách nhiệm

| Tên | Vai trò | Trách nhiệm | Actions phụ trách | Khả dụng |
|-----|---------|-------------|-------------------|----------|
| Tuấn | Lead | Ra quyết định, review output, unblock team | A001, A003 | 50% |
| Minh | Dev | Implement, test, deploy | A002, A004 | 100% |

## Công cụ & Tài liệu

| Tài nguyên | Mục đích | Status | Người quản lý |
|------------|----------|--------|---------------|
| [Tool X] | Dùng cho A002 | Có sẵn | Minh |

## Ngân sách (nếu có)

| Khoản mục | Dự kiến | Thực tế | Ghi chú |
|-----------|---------|---------|---------|

## Thiếu hụt & Rủi ro

- [Liệt kê resource chưa có nhưng cần, rủi ro tiềm ẩn]
```

### ok-track: Theo dõi tiến độ

**Trigger**: `/ok-track` hoặc `/ok` khi đang phase thực thi

**Workflow**:
1. Đọc objective.md + frontmatter tất cả actions/
2. Tính tiến độ mỗi Key Result
3. So sánh tiến độ thực vs. kế hoạch (theo timeline)
4. Cập nhật SOT files (objective.md Current, plan.md counts)
5. Ghi entry vào log/YYYY-MM-DD.md
6. Highlight: actions chậm, blocked, sắp deadline

**Cách tính tiến độ Key Result**:

Mỗi KR có Baseline, Target, Current. Tiến độ = (Current - Baseline) / (Target - Baseline) * 100%.

Cập nhật Current bằng 2 cách:
1. User tự nhập giá trị (ưu tiên, vì KR đo outcome)
2. Tính từ actions (nếu KR đo số lượng hoàn thành)

**Format `log/YYYY-MM-DD.md`**:

```yaml
---
date: 2026-05-09
type: tracking
---
```

```markdown
## Thay đổi

- KR1: 30% -> 45%
- A003: doing -> done (PIC: Minh)
- A005: doing -> blocked (lý do: chờ phản hồi từ X)

## Ghi chú

[Nhận xét, context bổ sung]
```

### ok-review: Lookback & điều chỉnh

**Trigger**: `/ok-review`

**Workflow**:
1. Đọc objective.md + plan.md (SOT) để biết trạng thái hiện tại
2. Đọc log/*.md để thấy progression theo thời gian
3. Phân tích:
   - Cái gì đạt, cái gì trượt
   - Root cause (không dừng ở triệu chứng)
   - Tốc độ thực tế vs. kế hoạch
4. Đề xuất điều chỉnh:
   - Thay đổi target Key Result
   - Thêm/bớt/sửa actions
   - Dời deadline milestones
   - Thay đổi phân công (PIC)
5. Nếu user đồng ý, cập nhật SOT files
6. Ghi log vào log/reviews/YYYY-MM-DD.md

**Format `log/reviews/YYYY-MM-DD.md`**:

```yaml
---
date: 2026-06-01
type: review
period: "2026-05-09 to 2026-06-01"
---
```

```markdown
## Tổng kết

| Key Result | Target | Current | % | Trend |
|-----------|--------|---------|---|-------|
| KR1 | 100 | 45 | 45% | on-track |
| KR2 | 50 | 10 | 20% | at-risk |

## Phân tích

### Đạt tốt
- [Liệt kê]

### Cần cải thiện
- [Liệt kê + root cause]

## Điều chỉnh đã áp dụng

- [Liệt kê thay đổi đã đồng ý]
```

### ok: Orchestrator (router)

**Trigger**: `/ok` không kèm tham số

**Logic**:

```
1. .ok/ tồn tại?
   Không → gọi ok-init

2. .ok/objective.md tồn tại?
   Không → gọi ok-init

3. .ok/plan.md tồn tại?
   Không → gọi ok-plan

4. Đọc trạng thái actions:
   Tất cả done → gọi ok-review
   Có blocked → hiển thị blockers, hỏi user
   Bình thường → gọi ok-track

5. User override: /ok init|plan|resource|track|review
```

## Tham khảo

- [OKR Best Practices (Synergita)](https://www.synergita.com/blog/okr-best-practices/)
- [OKR Guide (Devokr)](https://devokr.com/en/blog/okr-guide)
- [CCPM: Project Management for AI Agents](https://github.com/automazeio/ccpm)
- [Planning with Files](https://github.com/OthmanAdi/planning-with-files)
- [Single Source of Truth (Wikipedia)](https://en.wikipedia.org/wiki/Single_source_of_truth)
