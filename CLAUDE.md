# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project là gì

Objective Kit là bộ **Claude Code skills** quản lý mục tiêu theo OKR. Repo này chứa source code của skills, KHÔNG phải project sử dụng skills.

Khi làm việc trong repo này, bạn đang **phát triển/sửa skill**, không phải chạy `/okr` để quản lý mục tiêu.

## Cấu trúc skill

Mỗi skill = 1 thư mục trong `skills/`, gồm:
- `SKILL.md`: prompt chính (YAML frontmatter `name` + `description` + body hướng dẫn)
- `references/`: tài liệu bổ sung được skill load khi cần (schema, flow chi tiết, guide)

Không có code runtime (không có build, test, lint). Toàn bộ "code" là markdown prompt.

## Kiến trúc hub-and-spoke

```
skills/okr/SKILL.md          ← Orchestrator (hub), entry point duy nhất cho user
  ├── skills/okr-init/        ← Khởi tạo/sửa objective + resource
  ├── skills/okr-plan/        ← Tạo/sửa plan + actions
  ├── skills/okr-track/       ← Track progress + review + inbox processing
  └── skills/okr-capture/     ← Thu thập nhanh ý tưởng vào inbox
```

- `okr` đọc state `.okr/`, hiển thị status, route sang skill con phù hợp.
- Skill con KHÔNG được gọi trực tiếp (trừ khi user gõ đúng tên).
- Shared content tập trung tại `skills/okr/references/` (SOT ownership, schemas, quality gate, delegate protocol). Skill con link sang, không duplicate.

## Nguyên tắc thiết kế (phải giữ khi sửa skill)

1. **Solo only**: Persona duy nhất là 1 user cá nhân, 1 objective. Không team, không multi-objective.
2. **SOT ownership**: Mỗi field chỉ 1 skill được sửa. Bảng canonical tại `skills/okr/references/sot-ownership.md`. Bảng readable tại section dưới.
3. **Confirm trước ghi**: `init` và `plan` bắt buộc hiển thị bảng tóm tắt, user xác nhận trước khi ghi file.
4. **Track đề xuất, init/plan áp dụng**: `okr-track` không sửa cấu trúc (KR target, action mới, deadline), chỉ sửa progress fields. Muốn sửa cấu trúc → delegate sang init/plan.
5. **Quality Gate internal**: 3 câu check (đủ cụ thể? giả định ẩn? mâu thuẫn?) chạy ngầm, không hiển thị cho user. Chi tiết tại `skills/okr/references/quality-gate.md`.
6. **Tiết kiệm token**: Archive invisible by default. Log chỉ đọc mới nhất. Orchestrator preload SOT, skill con không đọc lại.

## Phân vai SOT

| Field                                                        | Skill được phép sửa                              |
| ------------------------------------------------------------ | ------------------------------------------------ |
| Objective text, KR/KI target/baseline/ngưỡng, period, status | `okr-init` `update-objective`                    |
| Solo Profile (capacity, skills), tool, ngân sách             | `okr-init` `update-resource`                     |
| Milestones, action structure (title, deadline, deps)         | `okr-plan` `update`                              |
| Action `## Output/Deliverable` (ghi đè output thực tế)      | `okr-track` `light`/`deep`                       |
| KR.current, KI.current, plan counters                        | `okr-track` `light`/`deep`                       |
| plan.last_track_date, plan.last_review_date                  | `okr-track` `light`/`deep`                       |
| action.status                                                | `okr-track` `light`/`deep`, `okr-plan` `update`  |
| Inbox items (tạo mới)                                        | `okr-capture`                                    |
| Inbox items (xử lý: status transition)                       | `okr-track`                                      |
| Action notes, external_ids                                   | `okr-plan` `new`/`update`                        |
| External sync (pull/push status)                             | `okr-track` `light`/`deep`                       |

> Bảng canonical: `skills/okr/references/sot-ownership.md`. Sửa ở đó, bảng trên đây giữ đồng bộ.

## Hai loại mục tiêu (domain knowledge)

- **Project**: có deadline, đo bằng Key Results (baseline → target), kết thúc khi đạt.
- **Ongoing**: duy trì liên tục, đo bằng Key Indicators (ngưỡng tối thiểu), không "xong".

## Cấu trúc dữ liệu runtime (để hiểu schema)

Skills sinh ra thư mục `.okr/` tại project của user (KHÔNG phải trong repo này):

```
.okr/
├── objective.md          # SOT mục tiêu + KR/KI
├── resources.md          # SOT người + tool + ngân sách
├── plan.md               # SOT milestones + counters + Roadmap table
├── actions/              # 1 file/action (AXXX-slug.md)
│   └── archive/          # Actions done, read-only
├── inbox/                # Capture items chờ xử lý
└── log/                  # Append-only, type: [tracking|review|closure]  (okr-track)
```

Schema chi tiết nằm trong `references/data-format.md` của từng skill.

## Quy ước phát triển

### Sửa skill

1. Đọc `SKILL.md` + tất cả `references/` của skill đó trước khi sửa.
2. Kiểm tra SOT ownership: field bạn sửa có thuộc skill này không?
3. Shared content (`skills/okr/references/`) là canonical. Skill con chỉ link, không copy.
4. Sửa xong kiểm tra consistency: cùng concept phải nói giống nhau ở mọi file nhắc đến nó.

### Commit convention

Prefix: `feat`, `fix`, `refactor`, `docs`, `style`. Scope trong ngoặc: `(okr)`, `(okr-init)`, `(okr-plan)`, `(okr-track)`, `(okr-capture)`.

Ví dụ: `feat(okr-track): add period overdue warning to dashboard`

### Tài liệu phát triển

- `docs/okr-system-review.md`: review kiến trúc toàn hệ thống (sơ đồ, trạng thái, flow)
- `deep-insight/`: phân tích sâu, review chất lượng skill
- `docs/superpowers/plans/`: kế hoạch implement từng đợt cải tiến
- `docs/superpowers/specs/`: design spec cho thay đổi lớn

### Lịch sử phát triển (các đợt cải tiến)

Skill được cải tiến qua nhiều đợt, mỗi đợt có plan + spec riêng trong `docs/superpowers/`:

1. Đợt 1: Sync tài liệu (inconsistency giữa các skill)
2. Đợt 2: Solo defaults (gỡ bias team-oriented)
3. Đợt 3: Lifecycle gaps (period overdue, inbox aging, delegate reason)
4. Đợt 4: Dọn thừa (merge routing table, tách Quality Gate, hub-and-spoke)
5. Feedback integration (Roadmap table, external IDs, resources 6 cột)
6. Hub-and-spoke refactor (shared content tập trung)
