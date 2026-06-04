# Đợt 15: Reachability khi ghi (chống file mồ côi)

- Ngày: 2026-06-04
- Trạng thái: Design approved (đã qua 1 vòng phản biện đối kháng), chờ implement plan
- Phạm vi: runtime `.okr/` tại project đích (KHÔNG gồm dev-time repo harness)
- Đối tượng sửa: okr-shared (preload/sot-ownership/schemas), okr-init, okr-plan, okr-track, okr-analyze, okr-harness, CLAUDE.md, CHANGELOG.md

## 1. Bối cảnh và vấn đề

Preload Contract (Đợt 14) giải bài toán *reachability khi ĐỌC*: bảo đảm mọi entry point nạp đủ nền trước khi thao tác. Đợt 15 giải mặt đối xứng: *reachability khi GHI*.

**Vấn đề (đã xác nhận, đã xảy ra thật):** mỗi project đích, lúc dùng harness, phát sinh file mới trong `.okr/`. Loại file không đoán trước, khác nhau theo dự án (glossary, playbook, ghi chú vận hành, bảng tra, file phụ trợ deliverable, data dump...). File đặt ở vị trí ad-hoc (vd `.okr/glossary.md`, `.okr/research/data.md`) không reachable từ gốc preload. Phiên sau orchestrator không nạp, agent không biết file tồn tại. File thành **mồ côi**.

Vì loại file vô hạn theo dự án, không hard-code danh sách được. Cần **cơ chế tổng quát**: cách neo file bất kỳ về gốc preload, cộng một nhà độc lập cho tri thức cross-cutting.

## 2. Nguyên tắc lõi: Reachability khi ghi

Canonical tại `skills/okr-shared/references/preload.md`, cạnh "reachability khi đọc".

> Mọi file/thư mục sinh trong `.okr/` phải **reachable** từ gốc preload, qua **link/đăng ký** hoặc qua **vị trí cấu trúc đã biết**. Không reachable = mồ côi = không hợp lệ.

Hai cơ chế reachable:
- **Qua link/đăng ký:** path file xuất hiện trong một SOT đã preload (Roadmap link, cột Resource của resources, entry trong `context/index.md`, link trong `lessons/index.md`).
- **Qua vị trí cấu trúc đã biết:** file nằm trong thư mục mà preload/skill biết cách quét (xem mục 4.1).

## 3. Quyết định thiết kế

Chọn **PA3 (neo qua link vào SOT đã preload + audit backstop ở okr-analyze)** làm nền, nâng **context/ thành nhà độc lập first-class** theo yêu cầu user.

Loại 2 phương án kia:
- **PA1 (chỉ audit phát hiện):** không cấp nhà cho file lạ, giải nửa vời, dễ lặp orphan.
- **PA2 (full registry làm gốc preload bắt buộc, đăng ký mọi file):** nặng nhất, tốn token mỗi phiên, đăng ký lại cả file đã neo nơi khác gây nhiễu, drift cao, over-engineer cho solo.

PA3 tổng quát mà rẻ và ít drift: phần lớn file lạ quy được về SOT đã preload; reachability-qua-link đã chứng minh chạy trong prompt-only (Roadmap link tới action body).

## 4. Bản đồ neo (anchor map)

| Loại file | Neo vào | Skill ghi link | Trạng thái cơ chế |
| --- | --- | --- | --- |
| Tài liệu/nguồn tham chiếu DÙNG (KB ngoài, link, file deliverable cần tra) | 1 dòng trong `## Tài liệu & Knowledge Base` của `resources.md`, cột `Resource` = path/URL | `okr-init` update-resource | Section + cột path đã có |
| Deliverable file riêng của action | Path ghi vào `## Output/Deliverable` của action; action reachable qua vị trí (`actions/`) + Roadmap link | `okr-plan` (dự kiến lúc tạo), `okr-track` (output thực tế) | Đã có; bổ sung quy tắc giữ path bền (mục 4.3) |
| Bài học | `lessons/index.md` | `okr-retro` | Đã có |
| Inbox item | `inbox/` (thư mục cấu trúc) | `okr-capture` | Đã có |
| Log entry | `log/` (thư mục cấu trúc) | `okr-track` | Đã có |
| **Tri thức/nội dung cross-cutting do dự án TẠO RA** (glossary nội bộ, playbook, ghi chú vận hành, bảng tra, data dump phụ trợ) | `.okr/context/<slug>.md` + entry trong `context/index.md` | `okr-init` / `okr-plan` / `okr-track` (đa-skill, owner-per-entry) | **Mới** |

### 4.1. Vùng "cấu trúc đã biết" (reachable theo vị trí)

Thư mục/file mà preload/skill biết cách quét, audit coi reachable theo vị trí, KHÔNG báo mồ côi:

- File SOT đơn ở `.okr/` root: `objective.md`, `resources.md`, `plan.md`.
- Thư mục: `actions/`, `actions/archive/`, `inbox/`, `log/`, `lessons/` (gồm `index.md`, `skill/`, `project/`), `context/` (gồm `index.md`, các `<slug>.md`).

Mọi file/thư mục NGOÀI tập trên (file lẻ ở root, thư mục con lạ) là ứng viên mồ côi, phải neo qua link/đăng ký.

> "Gốc preload" (Tier 1 Preload Contract) RỘNG hơn "3 file SOT đơn": gồm thêm `lessons/index.md` (nạp toàn bộ) + scan `actions/`/`inbox/`. Khi viết schema, dùng đúng cụm "file SOT đơn ở root" cho audit, không gọi tắt thành "gốc preload".

### 4.2. Ranh giới ngữ nghĩa resources.md vs context/

Điểm dễ lẫn, phải ghi rõ trong schema:

- **`resources.md` `## Tài liệu & Knowledge Base`** = *con trỏ tới nguồn bạn DÙNG/THAM CHIẾU*. Mỗi dòng trỏ tới một resource (thường ngoài `.okr/` hoặc là deliverable cần tra). Không chứa nội dung tri thức.
- **`context/`** = *nhà chứa nội dung cross-cutting do chính dự án tạo ra trong `.okr/`* (tri thức HOẶC data/state phụ trợ), không thuộc objective/plan/resources/lessons/deliverable. Song song `lessons/`, có index riêng auto-load (có điều kiện).

Cây phân loại khi tạo file:
1. Là sản phẩm của một action? → `## Output/Deliverable` của action đó.
2. Là con trỏ tới nguồn/tài liệu để DÙNG? → 1 dòng trong `## Tài liệu & Knowledge Base`.
3. Là bài học? → `lessons/` (qua okr-retro).
4. Còn lại (nội dung/data cross-cutting do dự án sinh) → `.okr/context/` + entry index.

### 4.3. Giữ path deliverable bền qua track ghi đè

`## Output/Deliverable` là SOT của `okr-track` (ghi đè output thực tế). Nếu deliverable là file riêng, `okr-track` khi ghi đè PHẢI giữ/cập nhật dòng path file, không làm đứt link reachable. Quy ước: dòng path đặt ở đầu section dạng `Path: <đường dẫn>`, phần mô tả output free-form bên dưới. Audit dựa vào dòng `Path:` này.

## 5. Tầng context/ (nhà độc lập)

### 5.1. Cấu trúc

```
.okr/context/
├── index.md          # Registry nhẹ. Conditional preload khi context/ tồn tại.
└── <slug>.md         # 1 file/chủ đề
```

### 5.2. Format index.md (4 trường)

```markdown
# Context Index

> Nội dung cross-cutting của project, không thuộc objective/plan/resources/lessons.
> Conditional preload khi thư mục context/ tồn tại. Body file nạp on-demand.

| Path | Vai trò | Khi nào cần đọc | Owner |
| --- | --- | --- | --- |
| context/glossary.md | Định nghĩa thuật ngữ riêng của project | Khi gặp thuật ngữ lạ trong action/objective | okr-track |
| context/playbook-deploy.md | Quy trình deploy chuẩn | Khi làm action liên quan deploy | okr-plan |
```

- `Path`: đường dẫn tương đối từ `.okr/`. Là KEY.
- `Vai trò`: 1 câu, file là gì.
- `Khi nào cần đọc`: mô tả ngôn ngữ tự nhiên để agent tự quyết nạp body (không có executor, xem mục 7).
- `Owner`: skill tạo entry.

### 5.3. Model ghi index.md (cấu trúc MỚI, không có tiền lệ trong repo)

`context/index.md` là file bảng được NHIỀU skill ghi. Đây là cấu trúc ghi-đa-skill ĐẦU TIÊN của hệ; KHÔNG có tiền lệ "owner-per-entry như log/" (thực tế `log/` do duy nhất okr-track ghi). Phải đặc tả tường minh:

- **KEY = cột `Path`**, unique. Idempotent: nếu Path đã có entry → cập nhật entry tại chỗ, KHÔNG thêm dòng trùng.
- **Append**: entry mới thêm cuối bảng.
- **Single-owner per entry**: mỗi entry có 1 `owner` (skill tạo file). Skill KHÔNG sửa entry của owner khác (chỉ sửa entry mình tạo). Vì vậy index tuy đa-skill-ghi nhưng mỗi entry vẫn single-owner, KHÔNG vi phạm nguyên tắc "1 field 1 skill" (index chỉ là tập hợp entry single-owner).
- Cấu trúc index (bảng nhẹ, body on-demand) theo mẫu `lessons/index.md`; `Vai trò` là SOT của dòng index (sửa file thì cập nhật lại dòng).

### 5.4. Conditional preload + chủ thể phát hiện

Preload Contract HIỆN TẠI không `ls` cấp `.okr/` root (chỉ `ls` bên trong `actions/` và `inbox/`). Đợt 15 BỔ SUNG tường minh:

- **Thêm 1 bước đầu Phase 1 (Preload Contract Tier 1):** `Bash ls -1 .okr/` (cấp 1, không đệ quy). Đây là chủ thể DUY NHẤT: (a) phát hiện `context/` tồn tại, (b) phát hiện file lẻ ở root / thư mục con lạ cho Light audit (mục 6).
- Nếu `context/` tồn tại → nạp `context/index.md` toàn bộ (nhẹ, như `lessons/index.md`). Không tồn tại → bỏ qua, tốn 0 token.
- Quy tắc này đặt trong **preload.md Tier 1**, áp dụng cho CẢ 5 entry point Tier 1 (harness/analyze/init/plan/track). Khi chạy lẻ (không qua harness), init/plan/track vẫn nạp index. Tránh lặp lỗ hổng entry-point-lẻ (như guard lessons Đợt 13/14).

## 6. Enforcement 2 lớp

### 6.1. Lớp 1: Gate lúc tạo, định nghĩa theo VỊ TRÍ GHI

Tiêu chí "cần dòng confirm neo" định nghĩa theo VỊ TRÍ, không theo trạng thái "đã neo chưa" (tránh vòng lặp định nghĩa). Cây quyết định 2 nhánh, đặt canonical trong preload.md:

- **Nhánh 1 (KHÔNG cần dòng confirm riêng):** file tạo trong thư mục cấu trúc đã biết qua đúng flow chuẩn của thư mục đó, mà flow đã bao gồm bước neo. Cụ thể: action vào `actions/` (okr-plan ghi Roadmap link), lesson vào `lessons/` (okr-retro ghi index), inbox vào `inbox/`, log vào `log/`. Việc neo là bước bắt buộc sẵn trong flow.
- **Nhánh 2 (BẮT BUỘC dòng confirm + chỉ rõ đích neo):** file tạo ở vị trí KHÁC: deliverable file riêng (cạnh action/ngoài), tài liệu DÙNG, file context/, hoặc bất kỳ file ad-hoc. Skill ghi chèn vào **confirm table đã có** 1 dòng:

  > Sẽ tạo `<path>`, neo vào `<đích: resources / Output-Deliverable / context-index>`, vai trò `<1 câu>`.

File context/ thuộc nhánh 2 (đích neo = `context/index.md` entry); dòng confirm hiển thị việc tạo file + ghi entry.

`okr-init`, `okr-plan`, `okr-track` đã có confirm table → chỉ thêm dòng, không thêm phase. `okr-capture` KHÔNG có confirm table và là Tier 2 (không đủ context phân loại) → **không tham gia tạo file context/** (xem 6.3).

### 6.2. Lớp 2: Audit backstop (okr-analyze, read-only)

**Light (dashboard, mỗi phiên có plan):** tái dùng kết quả `ls -1 .okr/` (mục 5.4). Chỉ soi cấp 1: file lẻ ở root ngoài 3 SOT đơn + index, và thư mục con LẠ (ngoài tập 4.1). KHÔNG đệ quy, KHÔNG đọc link-set, KHÔNG khẳng định mồ côi:

```
ℹ️ Có file/thư mục ở vị trí lạ: <tên>. Chạy review sâu để xác minh + cách neo.
```

Giữ claim token-cheap (1 lệnh ls cấp 1, output ngắn).

**Deep / closure:** đối chiếu đầy đủ reachable set (mục 8). Với mỗi vấn đề: file mồ côi (báo path + gợi ý neo theo bản đồ 4); link chết; context lệch (file context/ thiếu entry, hoặc entry trỏ file đã xóa). Read-only: chỉ báo + đề xuất; việc neo do init/plan/track làm (giữ SOT ownership, đúng "track đề xuất, init/plan áp dụng").

### 6.3. okr-capture giữ nguyên (chỉ inbox)

`okr-capture` là Tier 2 minimal (chỉ nạp `lessons/index.md`), ghi nhanh vào inbox, không có confirm. Bắt nó phân loại tri thức cross-cutting + ghi entry 4 trường vượt vai trò. Quyết định: **capture chỉ ghi inbox như hiện tại.** Việc nâng một inbox item thành file context/ giao cho `okr-track` lúc xử lý inbox (track là Tier 1, đủ context phán đoán). Owner context/ = init/plan/track.

## 7. "Trigger load" trong hệ prompt-only + mệnh lệnh đối chiếu

Hệ không có executor. Ghi rõ trong preload.md để skill không hiểu nhầm.

- Cột "Khi nào cần đọc" KHÔNG tự kích hoạt. Agent đọc index đã preload, thấy mô tả khớp việc hiện tại thì TỰ nạp body file đó, giống nạp action body từ Roadmap link hôm nay.
- Để "tự nạp đúng lúc" không phụ thuộc may rủi, thêm **mệnh lệnh "Áp dụng context"** song song với "Áp dụng lessons" trong preload.md:

  > Nếu `context/index.md` đã nạp: trước khi xử lý action/đề xuất/ghi, quét cột "Khi nào cần đọc"; mục nào khớp chủ đề việc hiện tại thì đọc body file đó trước khi làm.

## 8. Thuật toán audit (reachable set, cho DEEP)

Light chỉ chạy `ls -1 .okr/` (mục 6.2). Deep chạy đầy đủ:

**Bước 1. Liệt kê thực tế:** `ls -R .okr/` (đệ quy). Chỉ lấy TÊN file, KHÔNG đọc body của `archive/`, `log/` (tôn trọng invisible-by-default: audit chỉ kiểm tồn tại path, không mở nội dung).

**Bước 2. Tập reachable theo vị trí:** mục 4.1 (3 file SOT đơn + các thư mục cấu trúc đã biết). File trực tiếp trong các thư mục này = reachable.

**Bước 3. Tập reachable qua link/đăng ký** (bắt file ngoài thư mục đã biết + để check link chết):
- Cột `Resource` của `resources.md` `## Tài liệu & Knowledge Base`.
- Dòng `Path:` trong `## Output/Deliverable` của action ACTIVE **và** action trong `actions/archive/` (deep/closure vốn đọc archive).
- `context/index.md` entries.
- Link trong `lessons/index.md` (mục còn hiệu lực; bỏ qua mục "Lưu trữ ported/obsolete", xem Bước 6).
- Roadmap link trong `plan.md` body (đọc on-demand) — để check Roadmap link chết.

**Bước 4. Mồ côi = file thực tế KHÔNG thuộc Bước 2 và KHÔNG thuộc Bước 3.** Điển hình: file ở `.okr/` root ngoài 3 SOT, hoặc trong thư mục con lạ (vd `.okr/research/`).

**Bước 5. Kiểm tra nội bộ context/:** mọi `context/<slug>.md` (trừ `index.md`) phải có entry trong `index.md`; mọi entry phải trỏ file tồn tại.

**Bước 6. Kiểm tra link chết:** mọi path Bước 3 phải trỏ file tồn tại.
- Roadmap link trỏ `actions/*` không tồn tại, entry context trỏ file đã xóa, cột Resource trỏ file `.okr/` không tồn tại → lỗi cứng.
- File NGOÀI `.okr/` không tồn tại (deliverable ngoài chưa tạo), lesson ported/obsolete đã xóa khỏi `.okr/` (hàng đợi port hợp lệ) → cảnh báo nhẹ, không lỗi cứng.

## 9. Files cần sửa

| File | Thay đổi |
| --- | --- |
| `skills/okr-shared/references/preload.md` | Mục "Reachability khi ghi": nguyên tắc + bản đồ neo + vùng cấu trúc đã biết (4.1) + cây quyết định gate theo vị trí (6.1) + định nghĩa trigger-load + mệnh lệnh "Áp dụng context" (7); thêm bước `ls -1 .okr/` + conditional load `context/index.md` vào Tier 1 (5.4) |
| `skills/okr-shared/references/sot-ownership.md` | Thêm: `## Tài liệu & Knowledge Base` → okr-init; `context/<slug>.md` → owner = skill ghi entry (1 file 1 owner); `context/index.md` → append-only đa-skill, KEY=Path, mỗi entry single-owner, cấm sửa entry owner khác; reachability audit → okr-analyze |
| `skills/okr-shared/references/schemas.md` | Section "Context layer": cấu trúc `.okr/context/`, format entry 4 trường, model ghi index (5.3), ranh giới ngữ nghĩa vs resources (4.2) |
| `skills/okr-init/SKILL.md` + `references/data-format.md` | Quy tắc "tài liệu DÙNG đăng ký dòng `## Tài liệu & Knowledge Base`" + dòng confirm gate (nhánh 2); guard conditional-load context khi chạy lẻ |
| `skills/okr-plan/SKILL.md` + `references/task-format.md` | Deliverable file riêng → dòng `Path:` trong `## Output/Deliverable` + dòng confirm; tạo file context/ + entry; guard conditional-load |
| `skills/okr-track/SKILL.md` + `references/data-format.md` (+ flow liên quan) | Ghi đè `## Output/Deliverable` giữ dòng `Path:` bền (4.3); nâng inbox item → file context/ + entry; dòng confirm gate; guard conditional-load |
| `skills/okr-analyze/SKILL.md` | Check "Reachability audit": Light (ls cấp 1) + Deep (thuật toán mục 8); conditional đọc `context/index.md` |
| `skills/okr-harness/SKILL.md` | Preload Phase 1: thêm bước `ls -1 .okr/` + conditional nạp `context/index.md`; link bản đồ neo về preload.md |
| `CLAUDE.md` | Cấu trúc runtime `.okr/` thêm `context/`; lịch sử Đợt 15 |
| `CHANGELOG.md` | Ghi Đợt 15 |

## 10. Ngoài phạm vi (đợt sau)

- **Dev-time orphan trong repo harness** (quy ước "Sửa harness" CLAUDE.md): phạm vi + enforcement khác, để đợt riêng.
- `okr-retro` không sửa: bài học luôn đi `lessons/`, đã reachable.
- `okr-capture` không sửa (giữ inbox-only, mục 6.3).

## 11. Quyết định đã chốt

1. Hướng: **PA3** (link vào SOT + audit), không PA1/PA2.
2. Audit cadence: **Light = ls cấp 1 ở dashboard; Deep = đối chiếu đầy đủ ở deep/closure.**
3. Scope: **chỉ runtime `.okr/`**, tách dev-time.
4. **context/ là nhà độc lập first-class**, conditional preload, entry 4 trường có `owner`, đa-skill append-only (model mới, đặc tả 5.3).
5. **okr-capture giữ inbox-only**; owner context/ = init/plan/track.

## 12. Tiêu chí Done (acceptance)

**Tài liệu (đo bằng grep/đối chiếu):**
- `preload.md` có mục "Reachability khi ghi" canonical; các skill link về, không chép trùng.
- Bản đồ neo NHẤT QUÁN = cùng danh sách loại file + cùng đích neo xuất hiện ở preload.md / sot-ownership.md / schemas.md / CLAUDE.md (đối chiếu bảng).
- Định nghĩa "trigger load prompt-only" + mệnh lệnh "Áp dụng context" xuất hiện 1 nơi canonical.
- Conditional preload + bước `ls -1 .okr/` mô tả nhất quán ở preload.md + okr-harness.
- 3 skill init/plan/track có dòng confirm gate (nhánh 2) + guard conditional-load; KHÔNG thêm dòng cho file nhánh 1.
- Tên section dùng đúng `## Tài liệu & Knowledge Base` (khớp ký tự để audit grep).
- Không còn em-dash/en-dash trong file sửa.

**Hành vi audit (3 case kiểm thử mong đợi):**
- Case A: tạo `.okr/scratch.md` (file lẻ ở root). Light báo "có file ở vị trí lạ: scratch.md". Deep báo mồ côi + gợi ý (tài liệu→resources / cross-cutting→context/).
- Case B: tạo `.okr/research/data.md` (thư mục con lạ). Light báo "thư mục lạ: research/". Deep báo mồ côi + gợi ý neo.
- Case C: `.okr/context/glossary.md` đã có entry trong `context/index.md`. Light KHÔNG cảnh báo (context/ là thư mục đã biết). Deep: reachable, không báo. Nếu glossary.md THIẾU entry → Deep báo "context lệch".
