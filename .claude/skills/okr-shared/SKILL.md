---
name: okr-shared
description: "Domain knowledge dùng chung cho mọi OKR skill: SOT ownership, schemas (Roadmap, Inbox Aging, Archive), Quality Gate, delegate protocol, action priority algo. Load khi cần tra cứu quy tắc chung."
---

# OKR Shared: Domain Knowledge dùng chung

Chứa quy tắc và schema áp dụng cho mọi skill OKR. Không chạy độc lập. Đọc khi cần.

## Nội dung

| File | Mô tả | Skill dùng |
|------|-------|------------|
| `references/schemas.md` | Roadmap table format, Inbox Aging, Inbox type → Delegate mapping, Archive Rules | `okr-plan` (render Roadmap), `okr-track` (archive, inbox) |
| `references/sot-ownership.md` | Bảng phân vai field: skill nào được sửa gì | Mọi skill (check trước ghi) |
| `references/quality-gate.md` | 3 câu check: đủ cụ thể? giả định ẩn? mâu thuẫn? | `okr-init`, `okr-plan` (trước mỗi follow-up) |
| `references/delegate-protocol.md` | Pre-confirmed flow + reason display khi áp dụng thay đổi cấu trúc | `okr-track` (gửi), `okr-init`/`okr-plan` (nhận) |
| `references/action-priority.md` | Thuật toán chọn action ưu tiên (first-match, top N) | `okr-analyze` (xếp priority), `okr-track` (đề xuất next) |
| `references/metrics.md` | Công thức tính (KR%, KR/KI status, trend, period overdue) + tín hiệu chẩn đoán (action health, capacity signals) | `okr-analyze` (render + issue), `okr-track` (compute trước ghi), `okr-init`/`okr-plan` (capacity fit) |

## Khi nào đọc

- **okr-analyze**: `metrics.md` khi tính metrics + phát hiện issue. `action-priority.md` khi xếp priority. `schemas.md` khi cần hiểu Roadmap format.
- **okr-init / okr-plan**: `quality-gate.md` trước mỗi follow-up. `schemas.md` khi render Roadmap. `sot-ownership.md` khi check quyền ghi. `delegate-protocol.md` khi nhận pre-confirmed payload. `metrics.md` (section Capacity signals) khi check fit.
- **okr-track**: `metrics.md` khi compute lại KR/KI status trước ghi (hoặc khi chạy không qua analyze). `delegate-protocol.md` khi đề xuất thay đổi cấu trúc. `schemas.md` khi archive + xử lý inbox. `action-priority.md` khi đề xuất next action. `sot-ownership.md` khi check quyền ghi.

## Auto-load lessons (an toàn)

Layer bài học (`okr-retro`) được `okr-harness` Phase 1 nạp sẵn `.okr/lessons/index.md` mỗi phiên. An toàn: nếu một skill chạy mà `index.md` chưa có trong context và `.okr/lessons/index.md` tồn tại, đọc nó trước khi thao tác. Chỉ đọc `index.md`, không đọc file detail (load on-demand).
