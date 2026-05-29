# CLAUDE.md

Hướng dẫn phát triển repo Objective Kit (meta-harness).

## Project là gì

Objective Kit là bộ **harness (skill-only)** quản lý mục tiêu theo OKR. Repo chứa **source code** của harness, KHÔNG phải project sử dụng OKR.

Khi làm việc trong repo này, bạn đang **phát triển/sửa harness**, không phải chạy `/okr-harness`.

## Kiến trúc

8 skills, **skill-only, chạy inline bởi 1 agent**. KHÔNG dùng `.claude/agents/`, KHÔNG dùng agent team. Mỗi skill = 1 thư mục trong `.claude/skills/`, gồm `SKILL.md` + `references/` (load on-demand). Toàn bộ "code" là markdown prompt. Deploy = copy `.claude/skills/`, dùng được ở bất kỳ project Claude Code nào.

```
.claude/skills/
├── okr-harness/                ← Orchestrator: routing, chạy skill inline, tổng hợp
│   └── references/             ← flows.md, skill-contract.md
├── okr-analyze/                ← Phân tích read-only: metrics, issues, priority, dashboard
├── okr-init/                   ← objective, resource, KR/KI
│   └── references/             ← data-format, flow-new, flow-update-*, okr-guide
├── okr-plan/                   ← milestones, actions, dependencies
│   └── references/             ← data-format, action-guide, task-format, flow-*
├── okr-track/                  ← progress, review (deep), inbox, sync, closure, trace
│   └── references/             ← data-format, flow-shared/light/deep/inbox/closure/trace
├── okr-capture/                ← ghi nhanh vào inbox (inline)
│   └── references/             ← data-format
├── okr-retro/                  ← rút bài học từ phiên, ghi .okr/lessons/ (record-only)
│   └── references/             ← data-format
└── okr-shared/                 ← quy tắc chung dùng cho mọi skill
    └── references/             ← schemas, sot-ownership, quality-gate, delegate-protocol, action-priority, metrics
```

### Skill → Vai trò

| Skill | Vai trò |
|-------|---------|
| okr-harness | Entry point: đọc state, route theo intent, chạy skill phù hợp inline, tổng hợp. |
| okr-analyze | Đọc `.okr/`, tính metrics, phát hiện issues, xếp priority, dashboard. Read-only. |
| okr-init | Tạo/sửa objective, resource, KR/KI. Confirm trước ghi. |
| okr-plan | Tạo/sửa plan, milestones, actions. Confirm trước ghi. |
| okr-track | Cập nhật progress, review sâu, xử lý inbox, sync, archive, log, closure, trace. |
| okr-capture | Ghi nhanh vào inbox (phân loại + ghi). |
| okr-retro | Rút bài học từ phiên, ghi `.okr/lessons/`. Record-only, confirm trước ghi. |
| okr-shared | Quy tắc chung: SOT, schemas, quality gate, delegate, priority. Không chạy độc lập. |

### Chế độ thực thi: skill-only inline

Một agent đọc state rồi đọc tiếp SKILL.md phù hợp và thực thi. Chuyển giữa các skill = đọc tiếp SKILL.md kế tiếp, mang theo context. Deep review là chuỗi tuần tự `okr-analyze` → `okr-track` deep → `okr-init`/`okr-plan` (không còn agent team song song).

## Nguyên tắc thiết kế

1. **Solo only**: 1 user, 1 objective. Không team, không multi-objective.
2. **SOT ownership**: Mỗi field chỉ 1 skill được sửa. Canonical tại `.claude/skills/okr-shared/references/sot-ownership.md`.
3. **Confirm trước ghi**: Skill ghi (init/plan) hiển thị bảng tóm tắt, user xác nhận trước khi ghi.
4. **Track đề xuất, init/plan áp dụng**: `okr-track` không sửa cấu trúc. Muốn sửa → chạy tiếp `okr-init`/`okr-plan` (pre-confirmed).
5. **Quality Gate internal**: 3 câu check ngầm. Chi tiết tại `.claude/skills/okr-shared/references/quality-gate.md`.
6. **Tiết kiệm token**: Archive invisible by default. Log chỉ đọc mới nhất. Orchestrator preload SOT.

## Phân vai SOT

| Field | Skill được sửa |
|-------|----------------|
| Objective text, KR/KI target/baseline/ngưỡng, period, status | `okr-init` `update-objective` |
| Solo Profile (capacity, skills), tool, ngân sách | `okr-init` `update-resource` |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update` |
| Action `## Output/Deliverable` (ghi đè output thực tế) | `okr-track` `light`/`deep` |
| KR.current, KI.current, plan counters | `okr-track` `light`/`deep` |
| plan.last_track_date, plan.last_review_date | `okr-track` `light`/`deep` |
| action.status | `okr-track` `light`/`deep`, `okr-plan` `update` |
| Inbox items (tạo mới) | `okr-capture` |
| Inbox items (xử lý: status transition) | `okr-track` |
| Action notes, external_ids (tạo/sửa) | `okr-plan` `new`/`update` |
| External sync (pull/push status) | `okr-track` `light`/`deep` |
| Bài học (`.okr/lessons/**`) | `okr-retro` |

> Canonical: `.claude/skills/okr-shared/references/sot-ownership.md`. Sửa ở đó, bảng trên giữ đồng bộ.

## Hai loại mục tiêu

- **Project**: có deadline, đo bằng Key Results (baseline → target), kết thúc khi đạt.
- **Ongoing**: duy trì liên tục, đo bằng Key Indicators (ngưỡng tối thiểu), không "xong".

## Cấu trúc dữ liệu runtime

Harness sinh ra `.okr/` tại **project đích** (không phải repo này):

```
.okr/
├── objective.md          # SOT mục tiêu + KR/KI
├── resources.md          # SOT người + tool + ngân sách
├── plan.md               # SOT milestones + counters + Roadmap table
├── actions/              # 1 file/action (AXXX-slug.md)
│   └── archive/          # Actions done, read-only
├── inbox/                # Capture items chờ xử lý
├── log/                  # Append-only, type: [tracking|review|closure]
└── lessons/              # Bài học (okr-retro): index.md + skill/ + project/
```

Schema chi tiết: `references/data-format.md` trong mỗi skill.

## Quy ước phát triển

### Sửa harness

1. Đọc SKILL.md + references liên quan trước khi sửa.
2. Kiểm tra SOT ownership: field bạn sửa có thuộc skill này không?
3. Shared domain tại `.claude/skills/okr-shared/`. Các skill khác link sang, không copy.
4. Sửa xong kiểm tra consistency: cùng concept phải nói giống nhau ở mọi file.

### Commit convention

Prefix: `feat`, `fix`, `refactor`, `docs`, `style`.

Scope: `(okr-harness)`, `(okr-analyze)`, `(okr-init)`, `(okr-plan)`, `(okr-track)`, `(okr-capture)`, `(okr-retro)`, `(okr-shared)`.

Ví dụ: `feat(okr-track): add period overdue warning to dashboard`

### Tài liệu

- `docs/okr-system-review.md`: review kiến trúc toàn hệ thống
- `docs/`: phân tích sâu, review chất lượng
- `docs/superpowers/plans/`: kế hoạch implement từng đợt
- `docs/superpowers/specs/`: design spec cho thay đổi lớn
- `_archive/skills/`: kiến trúc hub-and-spoke gốc (reference lịch sử, không dùng)

### Deploy sang project đích

Copy `.claude/skills/` (8 skills + references) vào project muốn dùng OKR. KHÔNG cần copy gì khác: không có `.claude/agents/`, không lệ thuộc agent team.

Không cần sửa `CLAUDE.md` của project đích. Skill description của `okr-harness` tự trigger khi user nhắc OKR.

## Lịch sử phát triển

1. Đợt 1-4: Sync tài liệu, solo defaults, lifecycle gaps, dọn thừa
2. Đợt 5: Feedback integration (Roadmap table, external IDs, resources 6 cột)
3. Đợt 6: Hub-and-spoke refactor (shared content tập trung)
4. Đợt 7: Migrate sang harness (agent team + multi-skill)
5. Đợt 8: Đóng gói harness (self-contained, loại bỏ AGENTS.example.md)
6. Đợt 9: Skill-only - loại bỏ `.claude/agents/` + agent team, gộp 3 agent vào skill, thêm `okr-analyze`. Chạy inline 1 agent, deploy được mọi project Claude Code.
7. Đợt 10: Gom logic phân tích về `okr-shared` - dời `metrics.md` thành canonical dùng chung, okr-analyze/okr-track trỏ công thức chung thay vì chép, chống drift công thức.
8. Đợt 11: Layer lessons - thêm skill `okr-retro` rút bài học (2 loại: cải tiến skill / project), auto-load `.okr/lessons/index.md` mỗi phiên. Gỡ cơ chế "Tự cải tiến" + ghi CHANGELOG khỏi `okr-harness`.

Chi tiết thay đổi: xem `CHANGELOG.md`.