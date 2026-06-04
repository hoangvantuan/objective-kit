---
name: okr-init
description: "Tạo/sửa objective, KR, KI, period, deadline, resources, capacity, tool, ngân sách."
---

# OKR Init: Khởi tạo + cập nhật objective & resource

SOT chính: `objective.md` và `resources.md`. Ngoài ra, mode `update-resource` có thể tạo/sửa `context/<slug>.md` và entry `context/index.md` khi user nâng nội dung cross-cutting thành context, owner của entry đó là `okr-init`.

## Modes

| Mode | Trigger | Flow chi tiết |
|------|---------|---------------|
| `new` | Chưa có `.okr/` hoặc objective.md thiếu | `references/flow-new.md` |
| `update-objective` | Sửa objective/KR/KI/period | `references/flow-update-objective.md` |
| `update-resource` | Sửa capacity/skill/tool/ngân sách | `references/flow-update-resource.md` |

## SOT quyền ghi

| File | Fields |
|------|--------|
| objective.md | Objective text, KR/KI target/baseline/ngưỡng, period, status |
| resources.md | Solo Profile (capacity, skills), tool, tài liệu, ngân sách |
| context/index.md | Chỉ entry do `okr-init` làm owner, khi `update-resource` nâng nội dung cross-cutting thành context |
| context/<slug>.md | Chỉ file context do `okr-init` tạo cho nội dung cross-cutting từ `update-resource` |

> Subset của bảng canonical `../okr-shared/references/sot-ownership.md`. Sửa canonical trước, bảng này theo sau.

## Nguyên tắc

- Hỏi từng câu một, không hỏi hàng loạt.
- BẮT BUỘC confirm bảng trước khi ghi file.
- Quality Gate 3 câu trước mỗi follow-up (xem `okr-shared` skill).
- Preload Contract Tier 1 (`../okr-shared/references/preload.md`): trước khi đề xuất/ghi, đảm bảo nền Tier 1 đã nạp (objective/plan frontmatter, `resources.md` full body, actions/inbox count, `lessons/index.md` toàn bộ, + conditional `context/index.md` khi `context/` tồn tại) + áp dụng lesson liên quan. Idempotent: qua harness đã có, chạy lẻ tự nạp phần thiếu. Quan trọng khi chạy lẻ, không qua `okr-harness`.
- **Reachability khi ghi** (`../okr-shared/references/preload.md`): tài liệu/nguồn DÙNG đăng ký 1 dòng vào `## Tài liệu & Knowledge Base` của `resources.md` (cột `Resource` = path/URL). Nếu tạo file mới ở vị trí lạ (file `context/` hoặc ad-hoc) → bắt buộc dòng confirm gate nhánh 2 (xem flow). Áp dụng "Áp dụng context" khi đề xuất.
- Solo only: 1 user, 1 objective.
- Đề xuất + lý do, user quyết.

## Schema

- `references/data-format.md`: schema objective.md + resources.md
- `references/okr-guide.md`: tiêu chí SMART KR, KI guidelines

## Quy tắc

- KHÔNG ghi file trước confirm.
- KR không SMART → chỉ rõ thiếu gì + gợi ý sửa.
- Không tạo plan.md hay action files (việc okr-plan).
- Không sửa action status (việc okr-track).
