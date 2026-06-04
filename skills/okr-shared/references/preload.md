# Preload Contract: context bảo đảm trước khi chạy skill

Canonical. Định nghĩa context nền PHẢI có trong context trước khi bất kỳ skill OKR nào thao tác.

**Vấn đề giải quyết**: khi gọi skill lẻ (`/okr-track`, `/okr-plan`, `/okr-init`...) KHÔNG qua `okr-harness`, skill thiếu phần preload mà orchestrator đáng lẽ làm, dẫn tới đề xuất sai vì thiếu nguồn lực / bài học. Contract bảo đảm: mọi entry point đều nạp đủ nền trước khi làm.

## Nguyên tắc idempotent (không đọc trùng)

Chỉ nạp cái **chưa có** trong context.

- Chạy QUA `okr-harness`: orchestrator Phase 1 đã preload Tier 1. Skill KHÔNG đọc lại.
- Chạy LẺ (không qua harness): skill tự nạp phần thiếu theo tier của mình, một lần, ở đầu flow (trước mọi thao tác).

Trước khi nạp: kiểm tra `.okr/` tồn tại. Không có → route `okr-init` mode new (không nạp gì thêm).

## Tier 1: FULL

Áp dụng: `okr-harness`, `okr-analyze`, `okr-init`, `okr-plan`, `okr-track`.

**Bước 0 (bắt buộc, trước bảng):** `Bash ls -1 .okr/` (cấp 1, KHÔNG đệ quy). Chụp cây thư mục gốc một lần. Đây là chủ thể DUY NHẤT để: (a) phát hiện `context/` tồn tại nhằm conditional load `context/index.md`, (b) phát hiện file lẻ ở root / thư mục con lạ cho Light audit của `okr-analyze` (xem "Reachability khi ghi"). Output ngắn, token-cheap.

| Nguồn | Độ sâu nạp | Dùng để |
| -------------- | --------------------------------------- | ------------------------------------------------ |
| `objective.md` | Frontmatter: `type`, `status`, `period` | Route, chọn metrics (project/ongoing), guard paused |
| `plan.md`      | Frontmatter + counters                  | Có/không plan, done rate từ counter |
| `resources.md` | **TOÀN BỘ body**                        | Capacity, skills, tool, tài liệu, ngân sách khi đề xuất/đối chiếu fit |
| `actions/*.md` | Scan frontmatter (Bash `ls` + đọc frontmatter từng file) | Count active, overdue (`due_date < today`), blocked |
| `inbox/*.md`   | Scan frontmatter `status: pending`      | Count pending |
| `lessons/index.md` | **TOÀN BỘ**                         | Áp dụng bài học (định hướng đề xuất, xem mục dưới) |
| `context/index.md` | **TOÀN BỘ**, conditional: chỉ khi `context/` tồn tại theo `ls` Bước 0 | Reachability khi ghi: biết file context cross-cutting nào tồn tại + khi nào nạp body (xem "Reachability khi ghi") |

## Tier 2: MINIMAL

Áp dụng: skill nhẹ, chỉ ghi, không suy luận về tiến độ.

| Skill | Nạp |
| ------------- | ----------------------------------------------------------- |
| `okr-capture` | `lessons/index.md` (toàn bộ), để phân loại type bám bài học đã có |
| `okr-retro`   | `lessons/index.md` (toàn bộ) + `objective.md` frontmatter, để dedup + gắn `area`/`period` cho bài học |

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
| `context/<slug>.md` body | Skill đọc khi cột "Khi nào cần đọc" trong `context/index.md` khớp việc hiện tại (xem "Áp dụng context"). Index chỉ chứa 4 trường, body on-demand |

## Áp dụng lessons (không chỉ load cho có)

`lessons/index.md` đã nạp → dùng làm **context định hướng**, không phải lệnh cứng. Trước khi đề xuất KR/action/điều chỉnh hoặc ghi file: đối chiếu lesson có essence liên quan việc đang làm (loại B theo `area`, loại A theo `target`). Lesson cảnh báo điều gì thì bám theo. Cần detail → đọc file lesson tương ứng. User vẫn quyết.

## Reachability khi ghi (chống file mồ côi)

Canonical. Đối xứng với reachability khi ĐỌC (phần trên bảo đảm nạp đủ nền). Phần này bảo đảm file MỚI sinh ra trong `.okr/` không thành mồ côi. `sot-ownership.md` / `schemas.md` / `CLAUDE.md` đối chiếu bản đồ neo dưới đây, không định nghĩa khác.

> Mọi file/thư mục sinh trong `.okr/` phải **reachable** từ gốc preload, qua **link/đăng ký** hoặc qua **vị trí cấu trúc đã biết**. Không reachable = mồ côi = không hợp lệ.

Hai cơ chế reachable:
- **Qua link/đăng ký:** path xuất hiện trong SOT đã preload hoặc SOT được audit/skill đọc on-demand: cột `Resource` của `resources.md`, entry trong `context/index.md`, link trong `lessons/index.md`, Roadmap link trong `plan.md`, dòng `Path:` trong action body.
- **Qua vị trí cấu trúc đã biết:** file nằm trong thư mục mà preload/skill biết cách quét (xem "Vùng cấu trúc đã biết").

### Bản đồ neo (anchor map)

| Loại file | Neo vào | Skill ghi link |
| --- | --- | --- |
| Tài liệu/nguồn tham chiếu DÙNG (KB ngoài, link, deliverable cần tra) | 1 dòng trong `## Tài liệu & Knowledge Base` của `resources.md`, cột `Resource` = path/URL | `okr-init` update-resource |
| Deliverable file riêng của action | Dòng `Path:` ở đầu `## Output/Deliverable` của action; action reachable qua `actions/` + Roadmap link | `okr-plan` (dự kiến), `okr-track` (output thực tế) |
| Bài học | `lessons/index.md` | `okr-retro` |
| Inbox item | `inbox/` (thư mục cấu trúc) | `okr-capture` |
| Log entry | `log/` (thư mục cấu trúc) | `okr-track` |
| Tri thức/nội dung cross-cutting do dự án TẠO RA (glossary nội bộ, playbook, ghi chú vận hành, bảng tra, data dump phụ trợ) | `.okr/context/<slug>.md` + entry trong `context/index.md` | `okr-init` / `okr-plan` / `okr-track` (owner per entry) |

Format chi tiết `context/` + ranh giới ngữ nghĩa vs `resources.md`: xem `schemas.md` section "Context layer".

### Vùng cấu trúc đã biết (reachable theo vị trí)

Thư mục/file mà preload/skill biết cách quét. Audit coi reachable theo vị trí, KHÔNG báo mồ côi:

- File SOT đơn ở `.okr/` root: `objective.md`, `resources.md`, `plan.md`.
- Thư mục: `actions/`, `actions/archive/`, `inbox/`, `log/`, `lessons/` (gồm `index.md`, `skill/`, `project/`), `context/` (gồm `index.md`, các `<slug>.md`).

Riêng `context/<slug>.md` còn phải có entry tương ứng trong `context/index.md`. Light audit chỉ thấy `context/` ở cấp 1 nên không cảnh báo. Đối chiếu file thật trong `context/` với `context/index.md` là việc của Deep audit trong `okr-analyze`. Thiếu entry thì là `context lệch`, không phải reachable đủ.

Mọi file/thư mục NGOÀI tập trên là ứng viên mồ côi, phải neo qua link/đăng ký. Quy tắc này không thay thế yêu cầu entry index của `context/`.

> Thuật ngữ: "gốc preload" (Tier 1) RỘNG hơn "3 file SOT đơn" (gồm thêm `lessons/index.md`, scan `actions/`/`inbox/`, conditional `context/index.md`). Khi nói audit theo vị trí, dùng đúng cụm "file SOT đơn ở root", KHÔNG gọi tắt thành "gốc preload".

### Cây quyết định gate khi tạo file (theo VỊ TRÍ GHI)

Tiêu chí "cần dòng confirm neo" định nghĩa theo VỊ TRÍ, KHÔNG theo trạng thái "đã neo chưa" (tránh vòng lặp định nghĩa). Hai nhánh:

- **Nhánh 1 (KHÔNG cần dòng confirm riêng):** file tạo trong thư mục cấu trúc đã biết qua đúng flow chuẩn của thư mục đó, mà flow đã bao gồm bước neo. Cụ thể: action vào `actions/` (`okr-plan` ghi Roadmap link), lesson vào `lessons/` (`okr-retro` ghi index), inbox vào `inbox/`, log vào `log/`. Neo là bước bắt buộc sẵn trong flow.
- **Nhánh 2 (BẮT BUỘC dòng confirm + chỉ rõ đích neo):** file hoặc nguồn ở vị trí KHÁC: deliverable file riêng, tài liệu DÙNG, file `context/`, hoặc bất kỳ file ad-hoc. Skill ghi chèn vào **confirm table đã có** (không thêm phase mới) 1 dòng:

  > Sẽ tạo/đăng ký `<path hoặc URL>`, neo vào `<đích: resources / Output-Deliverable / context-index>`, vai trò `<1 câu>`.

File `context/` thuộc nhánh 2 (đích neo = `context/index.md` entry); dòng confirm hiển thị việc tạo file + ghi entry. Light audit chỉ ghi nhận thư mục này ở cấp 1, không đánh `context lệch`. Deep audit trong `okr-analyze` mới đối chiếu file thật với index. Tài liệu DÙNG dạng URL thuộc nhánh 2 nhưng là đăng ký resource, không tạo file.

`okr-init`/`okr-plan`/`okr-track` đã có confirm table nên chỉ THÊM dòng. `okr-capture` (Tier 2, không có confirm table) KHÔNG tham gia tạo file context (chỉ ghi inbox); việc nâng inbox item thành file context giao `okr-track` lúc xử lý inbox.

### Trigger-load trong hệ prompt-only + mệnh lệnh "Áp dụng context"

Hệ KHÔNG có executor. Cột "Khi nào cần đọc" trong `context/index.md` KHÔNG tự kích hoạt. Agent đọc index đã preload, thấy mô tả khớp việc hiện tại thì TỰ nạp body file đó, giống nạp action body từ Roadmap link hôm nay.

> **Áp dụng context** (song song với "Áp dụng lessons"): nếu `context/index.md` đã nạp, trước khi xử lý action / đề xuất / ghi, quét cột "Khi nào cần đọc"; mục nào khớp chủ đề việc hiện tại thì đọc body file đó TRƯỚC khi làm.

### Giữ path deliverable bền qua track ghi đè

`## Output/Deliverable` là SOT của `okr-track` (ghi đè output thực tế). Nếu deliverable là file riêng, `okr-track` khi ghi đè PHẢI giữ/cập nhật dòng path file, không làm đứt link reachable. Quy ước: dòng `Path: <đường dẫn>` đặt ở ĐẦU section, phần mô tả output free-form bên dưới. Audit dựa vào dòng `Path:` này.
