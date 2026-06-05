---
name: okr-track
description: "Cập nhật progress, review sâu, inbox, closure, external sync."
---

# OKR Track: Track + Review + Inbox Processing

Ghi progress fields, log, archive. KHÔNG sửa cấu trúc. Khi xử lý inbox `thought` cross-cutting, có thể tạo `context/<slug>.md` và entry `context/index.md` với owner = `okr-track`.

## Modes

| Mode         | Trigger                        | Flow chi tiết                |
| ------------ | ------------------------------ | ---------------------------- |
| `light`      | Daily tracking                 | `references/flow-light.md`   |
| `deep`       | Review sâu, milestone, at-risk | `references/flow-deep.md`    |
| `inbox-only` | Xử lý inbox, skip progress     | `references/flow-inbox.md`   |
| `closure`    | Tổng kết khi hết period        | `references/flow-closure.md` |


Shared logic (dashboard, mode detection, rules): `references/flow-shared.md`

## SOT quyền ghi

| File             | Fields                                            |
| ---------------- | ------------------------------------------------- |
| objective.md     | KR.current, KI.current                            |
| plan.md          | counters, last_track_date, last_review_date       |
| actions/*.md     | status, `## Output/Deliverable` section           |
| log/*.md         | Append entries                                    |
| inbox/*.md       | Status transition (pending → processed/discarded) |
| context/*.md     | Tạo file context từ inbox `thought` cross-cutting |
| context/index.md | Thêm/cập nhật entry owner = okr-track             |
| actions/archive/ | Move done actions                                 |

> Subset của bảng canonical `../okr-shared/references/sot-ownership.md`. Sửa canonical trước, bảng này theo sau.

**KHÔNG được sửa**: Objective text, KR target, action title/deadline/deps. Đề xuất → áp dụng qua `okr-init`/`okr-plan` (pre-confirmed).

## Nguyên tắc

- Dashboard TRƯỚC khi hỏi update.
- Confirm trước ghi. Light ≤2 field → 1 dòng, ≥3 → bảng.
- Preload Contract Tier 1 (`../okr-shared/references/preload.md`): trước khi thao tác, đảm bảo nền Tier 1 đã nạp (objective/plan frontmatter, `resources.md` full body, actions/inbox count, `lessons/index.md` toàn bộ, + conditional `context/index.md` khi `context/` tồn tại) + áp dụng lesson liên quan khi đề xuất điều chỉnh (deep). Idempotent: qua harness đã có, chạy lẻ tự nạp phần thiếu. Quan trọng khi chạy lẻ, không qua `okr-harness`.
- **Reachability khi ghi** (`../okr-shared/references/preload.md`): ghi đè `## Output/Deliverable` PHẢI giữ/cập nhật dòng `Path:` (không làm đứt link). Nâng inbox item `thought` (tri thức cross-cutting) thành file `context/` + entry `context/index.md` (owner = okr-track) kèm dòng confirm gate. Áp dụng "Áp dụng context" khi xử lý action.
- Deep: trình bày + tinh chỉnh root cause (analyze deep tính ≥3 lần "tại sao?"). Đề xuất, không tự sửa cấu trúc.
- Append-only log. Không ghi đè.
- External sync: pull trước, confirm, push sau. Fail → skip, không block.
- Cuối flow đề xuất next action (xem `okr-shared` action-priority).

## Schema

- `references/data-format.md`: schema log, progress fields, inbox processing rules
- Metrics (KR%, KI status, trend, period overdue, action health): `../okr-shared/references/metrics.md` (canonical, dùng chung với okr-analyze)
