# Preload Contract: context bảo đảm trước khi chạy skill

Canonical. Định nghĩa context nền PHẢI có trong context trước khi bất kỳ skill OKR nào thao tác.

**Vấn đề giải quyết**: khi gọi skill lẻ (`/okr-track`, `/okr-plan`, `/okr-init`...) KHÔNG qua `okr-harness`, skill thiếu phần preload mà orchestrator đáng lẽ làm, dẫn tới đề xuất sai vì thiếu nguồn lực / bài học. Contract bảo đảm: mọi entry point đều nạp đủ nền trước khi làm.

## Nguyên tắc idempotent (không đọc trùng)

Chỉ nạp cái **chưa có** trong context.

- Chạy QUA `okr-harness`: orchestrator Phase 1 đã preload Tier 1. Skill KHÔNG đọc lại.
- Chạy LẺ (không qua harness): skill tự nạp phần thiếu theo tier của mình, một lần, ở đầu flow (trước mọi thao tác).

Trước khi nạp: kiểm tra `.okr/` tồn tại. Không có → route `okr-init` mode new (không nạp gì thêm).

## Tier 1 — FULL

Áp dụng: `okr-harness`, `okr-analyze`, `okr-init`, `okr-plan`, `okr-track`.

| Nguồn | Độ sâu nạp | Dùng để |
| -------------- | --------------------------------------- | ------------------------------------------------ |
| `objective.md` | Frontmatter: `type`, `status`, `period` | Route, chọn metrics (project/ongoing), guard paused |
| `plan.md`      | Frontmatter + counters                  | Có/không plan, done rate từ counter |
| `resources.md` | **TOÀN BỘ body**                        | Capacity, skills, tool, tài liệu, ngân sách khi đề xuất/đối chiếu fit |
| `actions/*.md` | Scan frontmatter (Bash `ls` + đọc frontmatter từng file) | Count active, overdue (`due_date < today`), blocked |
| `inbox/*.md`   | Scan frontmatter `status: pending`      | Count pending |
| `lessons/index.md` | **TOÀN BỘ**                         | Áp dụng bài học (định hướng đề xuất, xem mục dưới) |

## Tier 2 — MINIMAL

Áp dụng: skill nhẹ, chỉ ghi, không suy luận về tiến độ.

| Skill | Nạp |
| ------------- | ----------------------------------------------------------- |
| `okr-capture` | `lessons/index.md` (toàn bộ) — để phân loại type bám bài học đã có |
| `okr-retro`   | `lessons/index.md` (toàn bộ) + `objective.md` frontmatter — để dedup + gắn `area`/`period` cho bài học |

## KHÔNG preload (load on-demand bởi skill cần)

Liệt kê rõ để skill KHÔNG giả định nhầm các thứ này đã có sẵn. Skill nào cần thì tự đọc đúng lúc:

| Nguồn | Ai đọc, khi nào |
| --------------------------------- | ---------------------------------------------------------------- |
| `objective.md` body (bảng KR/KI)  | `okr-analyze` khi tính metrics / render dashboard |
| `plan.md` body (`## Roadmap`, `## Practices`) | `okr-track` light re-render Roadmap; Ongoing streak; `okr-init` impact check `review_cycle` |
| `actions/*.md` body (`## Checkpoints`, `## Output/Deliverable`) | `okr-track` khi update output / check checkpoint (effort `xl`) |
| `log/**`                          | `okr-track`/`okr-analyze` deep + closure theo Log Reading Rules (`okr-track` `data-format.md`). Light KHÔNG đọc |
| `actions/archive/**`              | `okr-track` closure; `okr-analyze` trace. Default ẩn |
| `lessons/skill/*`, `lessons/project/*` (file detail) | Đọc khi cần detail một bài (index chỉ chứa essence) |

## Áp dụng lessons (không chỉ load cho có)

`lessons/index.md` đã nạp → dùng làm **context định hướng**, không phải lệnh cứng. Trước khi đề xuất KR/action/điều chỉnh hoặc ghi file: đối chiếu lesson có essence liên quan việc đang làm (loại B theo `area`, loại A theo `target`). Lesson cảnh báo điều gì thì bám theo. Cần detail → đọc file lesson tương ứng. User vẫn quyết.
