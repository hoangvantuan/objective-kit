---
name: okr-shared
description: "Quy tắc chung mọi OKR skill: SOT ownership, schemas, Quality Gate, priority."
---

# OKR Shared: Domain Knowledge dùng chung

Chứa quy tắc và schema áp dụng cho mọi skill OKR. Không chạy độc lập. Đọc khi cần.

## Nội dung

| File | Mô tả | Skill dùng |
|------|-------|------------|
| `references/preload.md` | Preload Contract: context nền phải nạp trước khi chạy (Tier 1 full / Tier 2 minimal), `context/index.md` conditional, và Reachability khi ghi | Mọi skill (chạy ở đầu flow) |
| `references/schemas.md` | Roadmap table format, Inbox Aging, Inbox type → Delegate mapping, Archive Rules | `okr-plan` (render Roadmap), `okr-track` (archive, inbox) |
| `references/sot-ownership.md` | Bảng phân vai field: skill nào được sửa gì | Mọi skill (check trước ghi) |
| `references/quality-gate.md` | 3 câu check: đủ cụ thể? giả định ẩn? mâu thuẫn? | `okr-init`, `okr-plan` (trước mỗi follow-up) |
| `references/delegate-protocol.md` | Pre-confirmed flow + reason display khi áp dụng thay đổi cấu trúc | `okr-track` (gửi), `okr-init`/`okr-plan` (nhận) |
| `references/action-priority.md` | Thuật toán chọn action ưu tiên (first-match, top N) | `okr-analyze` (xếp priority), `okr-track` (đề xuất next) |
| `references/metrics.md` | Công thức tính (KR%, KR/KI status, trend, period overdue) + tín hiệu chẩn đoán (action health, capacity signals) | `okr-analyze` (render + issue), `okr-track` (compute trước ghi), `okr-init`/`okr-plan` (capacity fit) |

## Khi nào đọc

- **Mọi skill (đầu flow)**: `preload.md` xác định context nền phải nạp trước khi thao tác. Idempotent: qua harness thì đã nạp, gọi lẻ thì tự nạp phần thiếu. Tier 1 (analyze/init/plan/track/harness) full + conditional `context/index.md`; Tier 2 (capture/retro) minimal.
- **okr-analyze**: `metrics.md` khi tính metrics + phát hiện issue. `action-priority.md` khi xếp priority. `schemas.md` khi cần hiểu Roadmap format.
- **okr-init / okr-plan**: `quality-gate.md` trước mỗi follow-up. `schemas.md` khi render Roadmap. `sot-ownership.md` khi check quyền ghi. `delegate-protocol.md` khi nhận pre-confirmed payload. `metrics.md` (section Capacity signals) khi check fit.
- **okr-track**: `metrics.md` khi compute lại KR/KI status trước ghi (hoặc khi chạy không qua analyze). `delegate-protocol.md` khi đề xuất thay đổi cấu trúc. `schemas.md` khi archive + xử lý inbox. `action-priority.md` khi đề xuất next action. `sot-ownership.md` khi check quyền ghi.
- **okr-retro**: `sot-ownership.md` khi check quyền ghi `.okr/lessons/**`.

## Auto-load + áp dụng lessons

`lessons/index.md` là một phần của **Preload Contract** (`references/preload.md`): Tier 1 và Tier 2 đều nạp toàn bộ index mỗi phiên (idempotent, không đọc trùng). Chỉ index, không đọc file detail (load on-demand khi cần một bài cụ thể).

Cách áp dụng lessons (đối chiếu essence liên quan trước khi đề xuất/ghi) định nghĩa canonical ở `references/preload.md` mục "Áp dụng lessons". Không lặp lại ở đây để tránh drift.

`context/index.md` là phần conditional của Tier 1 khi `context/` tồn tại. Chỉ preload index. Body `context/<slug>.md` đọc on-demand theo `preload.md` "Áp dụng context". Không định nghĩa lại bản đồ neo ở đây.
