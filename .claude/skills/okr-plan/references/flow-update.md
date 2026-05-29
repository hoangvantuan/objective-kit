# Flow: Plan Update (điều chỉnh plan/actions)

> Self-contained. Orchestrator chạy inline, mang theo context đã có.

Trigger phổ biến: `okr-track` deep phát hiện KR at-risk hoặc blocker → đề xuất điều chỉnh cấu trúc → chạy tiếp `okr-plan` mode update.

## Phase 1: Hiển thị state hiện tại

**Bypass detect**: Nếu kích hoạt qua delegate từ `okr-track` inbox (`context.items[]` pre-processed, mỗi item có title, description, related_kr, effort gợi ý, milestone gợi ý) → SKIP Phase 1 + 2, đi thẳng Phase 3 (gom items thành changes) → Phase 4 confirm.

Mode bình thường: đọc `plan.md` + frontmatter `actions/*.md` (không archive) + `objective.md` + `resources.md`. Hiển thị:

```
Plan hiện tại
Milestones (N)
| Milestone    | Deadline   | Actions | Status       |
|-------------|------------|---------|--------------|
| M1: Research | 2026-10-15 | 3/3     | done         |

Cảnh báo
- KR2 at-risk → cần thêm action
```

## Phase 2: Hỏi user/nhận đề xuất

3 nguồn trigger:

1. **Track deep delegate** (`context.changes` + `context.reason`): hiển thị đề xuất, user chọn áp dụng. `pre_confirmed: true` → đi thẳng Phase 4 confirm (xem `delegate-protocol.md`).
2. **Track inbox delegate** (`context.items[]`): gom items thành changes. `action` item → "Thêm action mới". `blocker` → "Sửa action" status=blocked. `resource` → chạy tiếp `okr-init` mode update-resource.
3. **User trực tiếp**: menu:
   1. Thêm action mới
   2. Sửa action (title, due_date, status, deps, deliverable, effort, priority)
   3. Xoá action
   4. Dời deadline milestone
   5. Thêm/xoá milestone

## Phase 3: Thu thập thay đổi

Hỏi chi tiết từng cái user chọn.

## Phase 4: CONFIRM diff

Delegate protocol (pre-confirmed + reason display): xem `delegate-protocol.md`.

Luôn hiển thị bảng diff:

```
Thay đổi sắp áp dụng
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |

Tác động
- A013 mới → gắn KR2
- Dời M2 → actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```

## Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa `actions/AXXX-*.md`. KHÔNG tạo/sửa trong `actions/archive/`.
3. **Re-render Roadmap** trong `plan.md` body `## Roadmap`. Format xem `okr-shared/references/schemas.md`.
4. Đề xuất chạy `okr-track` mode light để confirm state mới.
