# CLAUDE.md

Hướng dẫn phát triển repo Objective Kit (meta-harness).

## Project là gì

Objective Kit là bộ **harness (skill-only)** quản lý mục tiêu theo OKR. Repo chứa **source code** của harness, KHÔNG phải project sử dụng OKR.

Khi làm việc trong repo này, bạn đang **phát triển/sửa harness**, không phải chạy `/okr-harness`.

## Kiến trúc

 Mỗi skill = 1 thư mục trong `skills/`, gồm `SKILL.md` + `references/` (load on-demand). Toàn bộ "code" là markdown prompt. Deploy = copy `skills/`, dùng được ở bất kỳ project Claude Code nào.

```
skills/
├── okr-harness/                ← Orchestrator: routing, chạy skill inline, tổng hợp
│   └── references/             ← flows.md, skill-contract.md
├── okr-analyze/                ← Phân tích read-only: metrics, issues, priority, dashboard, trace
│   └── references/             ← flow-trace
├── okr-init/                   ← objective, resource, KR/KI
│   └── references/             ← data-format, flow-new, flow-update-*, okr-guide
├── okr-plan/                   ← milestones, actions, dependencies
│   └── references/             ← data-format, action-guide, task-format, flow-*
├── okr-track/                  ← progress, review (deep), inbox, sync, closure
│   └── references/             ← data-format, flow-shared/light/deep/inbox/closure
├── okr-capture/                ← ghi nhanh vào inbox (inline)
│   └── references/             ← data-format
├── okr-retro/                  ← rút bài học từ phiên, ghi .okr/lessons/ (record-only)
│   └── references/             ← data-format
└── okr-shared/                 ← quy tắc chung dùng cho mọi skill
    └── references/             ← schemas, sot-ownership, quality-gate, delegate-protocol, action-priority, metrics
```

### Skill → Vai trò

| Skill       | Vai trò                                                                                                   |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| okr-harness | Entry point: đọc state, route theo intent, chạy skill phù hợp inline, tổng hợp.                           |
| okr-analyze | Đọc `.okr/`, tính metrics, phát hiện issues, xếp priority, dashboard, trace (xem lại history). Read-only. |
| okr-init    | Tạo/sửa objective, resource, KR/KI. Confirm trước ghi.                                                    |
| okr-plan    | Tạo/sửa plan, milestones, actions. Confirm trước ghi.                                                     |
| okr-track   | Cập nhật progress, review sâu, xử lý inbox, sync, archive, log, closure.                                  |
| okr-capture | Ghi nhanh vào inbox (phân loại + ghi).                                                                    |
| okr-retro   | Rút bài học từ phiên, ghi `.okr/lessons/`. Record-only, confirm trước ghi.                                |
| okr-shared  | Quy tắc chung: SOT, schemas, quality gate, delegate, priority. Không chạy độc lập.                        |


### Chế độ thực thi: skill-only inline

Một agent đọc state rồi đọc tiếp SKILL.md phù hợp và thực thi. Chuyển giữa các skill = đọc tiếp SKILL.md kế tiếp, mang theo context. Deep review là chuỗi tuần tự `okr-analyze` → `okr-track` deep → `okr-init`/`okr-plan` (không còn agent team song song).

## Nguyên tắc thiết kế

1. **Solo only**: 1 user, 1 objective. Không team, không multi-objective.
2. **SOT ownership**: Mỗi field chỉ 1 skill được sửa. Canonical tại `.skills/okr-shared/references/sot-ownership.md`.
3. **Confirm trước ghi**: Skill ghi (init/plan) hiển thị bảng tóm tắt, user xác nhận trước khi ghi.
4. **Track đề xuất, init/plan áp dụng**: `okr-track` không sửa cấu trúc. Muốn sửa → chạy tiếp `okr-init`/`okr-plan` (pre-confirmed).
5. **Quality Gate internal**: 3 câu check ngầm. Chi tiết tại `skills/okr-shared/references/quality-gate.md`.
6. **Tiết kiệm token, có chủ đích**: Archive invisible by default. Log chỉ đọc mới nhất. Preload theo **Preload Contract** (`skills/okr-shared/references/preload.md`): nạp đủ nền (gồm `resources.md` full body) trước khi thao tác để tránh đề xuất sai, nhưng giữ body objective/plan + action + log/archive on-demand. Cân bằng "đủ context" vs token, không nạp mù. Đợt 15 bổ sung **Reachability khi ghi** (cùng file): mọi file mới phải neo về gốc preload, `context/` là nhà cho tri thức cross-cutting (conditional preload), audit backstop ở `okr-analyze`.

## Phân vai SOT

| Field                                                             | Skill được sửa                                  |
| ----------------------------------------------------------------- | ----------------------------------------------- |
| Objective text, KR/KI target/baseline/ngưỡng, period, status      | `okr-init` `update-objective`                   |
| Solo Profile (capacity, skills), tool, ngân sách                  | `okr-init` `update-resource`                    |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update`                             |
| Action `## Output/Deliverable` (ghi đè output thực tế)            | `okr-track` `light`/`deep`                      |
| KR.current, KI.current, plan counters                             | `okr-track` `light`/`deep`                      |
| plan.last_track_date, plan.last_review_date                       | `okr-track` `light`/`deep`                      |
| action.status                                                     | `okr-track` `light`/`deep`, `okr-plan` `update` |
| action.completed_date (set khi done)                              | `okr-track` `light`/`deep`                      |
| Inbox items (tạo mới)                                             | `okr-capture`                                   |
| Inbox items (xử lý: status transition)                            | `okr-track`                                     |
| Action notes, external_ids (tạo/sửa)                              | `okr-plan` `new`/`update`                       |
| External sync (pull/push status)                                  | `okr-track` `light`/`deep`                      |
| Bài học (`.okr/lessons/**`)                                       | `okr-retro`                                     |
| `## Tài liệu & Knowledge Base` (resources.md): nguồn DÙNG          | `okr-init` `update-resource`                    |
| `context/<slug>.md` và `context/index.md` entry                   | Owner = skill tạo file (`okr-init`/`okr-plan`/`okr-track`) |
| Reachability audit (read-only)                                    | `okr-analyze`                                   |


> Canonical: `skills/okr-shared/references/sot-ownership.md`. Sửa ở đó, bảng trên giữ đồng bộ.

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
├── context/              # Tri thức/data cross-cutting do dự án tạo (index.md + <slug>.md)
└── lessons/              # Bài học (okr-retro): index.md + skill/ + project/
```

### Reachability khi ghi trong `.okr/`

Canonical: `skills/okr-shared/references/preload.md` "Reachability khi ghi". Bảng dưới chỉ là tóm tắt đối chiếu:

| Loại file/runtime data | Neo vào |
| --- | --- |
| Tài liệu/nguồn DÙNG | `resources.md` `## Tài liệu & Knowledge Base`, cột `Resource` |
| Deliverable file riêng của action | Dòng `Path:` ở đầu `## Output/Deliverable` của action |
| Bài học | `lessons/index.md` |
| Inbox item | `inbox/` |
| Log entry | `log/` |
| Tri thức/data cross-cutting do dự án tạo | `context/<slug>.md` + `context/index.md` |

Schema chi tiết: `references/data-format.md` trong mỗi skill.

## Quy ước phát triển

### Sửa harness

1. Đọc SKILL.md + references liên quan trước khi sửa.
2. Kiểm tra SOT ownership: field bạn sửa có thuộc skill này không?
3. Shared domain tại `skills/okr-shared/`. Các skill khác link sang, không copy.
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

Copy `.skills/` (8 skills + references) vào project muốn dùng OKR. KHÔNG cần copy gì khác

Không cần sửa `CLAUDE.md` của project đích. Skill description của `okr-harness` tự trigger khi user nhắc OKR.

## Lịch sử phát triển

1. Đợt 1-4: Sync tài liệu, solo defaults, lifecycle gaps, dọn thừa
2. Đợt 5: Feedback integration (Roadmap table, external IDs, resources 6 cột)
3. Đợt 6: Hub-and-spoke refactor (shared content tập trung)
4. Đợt 7: Migrate sang harness (agent team + multi-skill)
5. Đợt 8: Đóng gói harness (self-contained, loại bỏ AGENTS.example.md)
6. Đợt 9: Skill-only - loại bỏ `agents/` + agent team, gộp 3 agent vào skill, thêm `okr-analyze`. Chạy inline 1 agent, deploy được mọi project Claude Code.
7. Đợt 10: Gom logic phân tích về `okr-shared` - dời `metrics.md` thành canonical dùng chung, okr-analyze/okr-track trỏ công thức chung thay vì chép, chống drift công thức.
8. Đợt 11: Layer lessons - thêm skill `okr-retro` rút bài học (2 loại: cải tiến skill / project), auto-load `.okr/lessons/index.md` mỗi phiên. Gỡ cơ chế "Tự cải tiến" + ghi CHANGELOG khỏi `okr-harness`.
9. Đợt 12: Review drift tầng sâu (prompt-master). P0 done-count: done rate lấy từ counter `plan.md`, KR% cách 2 chỉ deep/closure, thêm "Tốc độ hoàn thành" (deep/closure). Thêm `action.completed_date`. Guard `paused` ở track. Chuyển mode `trace` từ `okr-track` sang `okr-analyze` (read-only về đúng nhà). Closure bỏ "## Lessons", gợi ý `okr-retro`. Wire "Áp dụng lessons". Bảng phân loại type cho capture. Sửa 4 tham chiếu chết + dọn em-dash.
10. Đợt 13: Tối ưu load file. `okr-analyze` tái dùng SOT đã preload từ orchestrator (không Read trùng `objective.md`/`plan.md`, nhất quán với `okr-track` flow-shared Phase 1). Guard lessons hạ xuống SKILL của `okr-init`/`okr-plan`/`okr-track`: đảm bảo `lessons/index.md` được nạp + áp dụng khi chạy lẻ không qua harness (trước chỉ ở `okr-shared` SKILL mà skill khác không bắt buộc đọc).
11. Đợt 14: Preload Contract. Vá lỗ hổng "entry point lẻ thiếu context": tạo `okr-shared/references/preload.md` canonical (Tier 1 full cho analyze/init/plan/track/harness, Tier 2 minimal cho capture/retro, idempotent). Thêm `resources.md` full body vào preload (trước chỉ on-demand, lại còn bị `okr-analyze` mô tả nhầm là "Frontmatter" dù data ở body). Mục "KHÔNG preload" liệt kê rõ thứ on-demand (body objective/plan, action, log, archive, lesson detail) để flow không giả định nhầm. Gộp guard lessons rời rạc (Đợt 13) vào contract. Drift-fix `flow-shared.md` ("resources KHÔNG preload" → đã preload full).
12. Đợt 15: Reachability khi ghi (chống file mồ côi). Đối xứng Đợt 14 (reachability khi đọc). Nguyên tắc lõi + bản đồ neo canonical ở `preload.md`: mọi file sinh trong `.okr/` phải reachable qua link/đăng ký hoặc vị trí cấu trúc đã biết. Nâng `context/` thành nhà độc lập first-class (index 4 trường, conditional preload, model ghi đa-skill owner-per-entry). Thêm `ls -1 .okr/` + conditional `context/index.md` vào Tier 1. Enforcement 2 lớp: gate dòng confirm lúc tạo (init/plan/track, nhánh 2 theo vị trí) + audit backstop read-only ở okr-analyze (Light ls cấp 1, Deep thuật toán reachable set 6 bước). Giữ dòng `Path:` deliverable bền qua track ghi đè. okr-capture/okr-retro không đổi. Chỉ runtime `.okr/`, tách dev-time.

Chi tiết thay đổi: xem `CHANGELOG.md`.
