# Đợt 15: Reachability khi ghi (chống file mồ côi) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Bảo đảm mọi file/thư mục sinh trong `.okr/` runtime đều reachable từ gốc preload (qua link/đăng ký hoặc vị trí cấu trúc đã biết), cộng một nhà độc lập `context/` cho tri thức cross-cutting, để phiên sau không bỏ sót file (chống "file mồ côi").

**Architecture:** Harness skill-only, toàn bộ "code" là markdown prompt. Giải pháp PA3: neo file qua link vào SOT đã preload + audit backstop read-only ở `okr-analyze` (Light/Deep). Canonical nguyên tắc + bản đồ neo đặt ở `okr-shared/references/preload.md`; các skill khác link/đối chiếu, không chép trùng. Thêm tầng `context/` first-class với index conditional-preload.

**Tech Stack:** Markdown prompt files trong `skills/`, Bash (`ls`, `grep`) để verify. KHÔNG có test runner tự động.

---

## Bối cảnh thực thi (đọc trước khi bắt đầu)

Đây KHÔNG phải codebase thông thường. Mỗi "skill" là một thư mục markdown trong `skills/`. Không có biên dịch, không có unit test. Vì vậy:

- **"Test" trong plan này = grep + Read đối chiếu.** Mỗi task có bước verify bằng lệnh `grep` (chuỗi canonical phải xuất hiện đúng) và đọc lại để xác nhận nội dung khớp ngữ cảnh xung quanh.
- **Nguyên tắc thứ tự (theo CLAUDE.md repo):** "Sửa canonical trước, bảng/skill link sau". Nên Task 1 (`preload.md` canonical) chạy đầu tiên, các skill (Task 4 trở đi) tham chiếu nó.
- **Cấm em-dash và en-dash** trong mọi file sửa. Đây là một tiêu chí Done của spec (mục 12). Dùng dấu phẩy, dấu hai chấm, hoặc tách câu. Plan này cũng tuân thủ.
- **Bản đồ neo lặp có kiểm soát ở 4 nơi** (preload.md / sot-ownership.md / schemas.md / CLAUDE.md). Mỗi nơi nhìn từ một góc (neo / ai-ghi / format / tổng quan) nhưng danh sách loại file + đích neo PHẢI khớp nhau. `preload.md` là canonical; 3 nơi kia đối chiếu. Task 10 verify nhất quán.
- **Phạm vi: CHỈ runtime `.okr/`.** Không đụng tới quy ước dev-time orphan trong repo harness (để đợt sau, spec mục 10).
- **Không sửa `okr-retro` và `okr-capture`** (spec mục 10 + 6.3): retro luôn ghi `lessons/` (đã reachable), capture giữ inbox-only.

Commit convention (CLAUDE.md): prefix `feat|fix|refactor|docs|style`, scope `(okr-shared)|(okr-init)|(okr-plan)|(okr-track)|(okr-analyze)|(okr-harness)`. Mỗi task kết thúc bằng 1 commit.

## File Structure (file đụng tới + trách nhiệm)

| File | Vai trò sau thay đổi | Task |
| --- | --- | --- |
| `skills/okr-shared/SKILL.md` + `references/preload.md` | Canonical: thêm mục "Reachability khi ghi" (nguyên tắc + bản đồ neo + vùng cấu trúc đã biết + cây quyết định gate + trigger-load + "Áp dụng context") và `ls -1 .okr/` + conditional load `context/index.md` vào Tier 1. SKILL overview trỏ đúng contract mới, không định nghĩa lại | 1 |
| `skills/okr-shared/references/sot-ownership.md` | Thêm dòng ownership: Tài liệu/KB, `context/<slug>.md`, `context/index.md`, reachability audit | 2 |
| `skills/okr-shared/references/schemas.md` | Thêm section "Context layer" (cấu trúc, format 4 trường, model ghi index, ranh giới vs resources) + bổ sung mapping `thought → context` | 3 |
| `skills/okr-harness/SKILL.md` + `references/flows.md` | Phase 1 preload: thêm `ls -1 .okr/` + conditional nạp `context/index.md`; link bản đồ neo; cập nhật flow inbox có nhánh `thought → context` | 4 |
| `skills/okr-analyze/SKILL.md` | Conditional đọc `context/index.md` + check "Reachability audit" (Light ls cấp 1 / Deep thuật toán 6 bước) | 5 |
| `skills/okr-init/SKILL.md` + `references/flow-update-resource.md` + `references/data-format.md` | Guard conditional-load context; đăng ký tài liệu DÙNG; dòng confirm gate (nhánh 2); cross-ref ranh giới resources vs context | 6 |
| `skills/okr-plan/SKILL.md` + `references/task-format.md` + `references/flow-new.md` + `references/flow-update.md` | Guard; quy ước dòng `Path:` trong `## Output/Deliverable`; dòng confirm gate; tạo file context/ + entry | 7 |
| `skills/okr-track/SKILL.md` + `references/data-format.md` + `references/flow-shared.md` + `references/flow-inbox.md` + `references/flow-light.md` | Guard; giữ dòng `Path:` bền khi ghi đè output; Tier 1 shared nhắc conditional context; nâng inbox item `thought` thành file context/; dòng confirm gate | 8 |
| `CLAUDE.md` | Cây runtime `.okr/` thêm `context/`; mục lịch sử Đợt 15 | 9 |
| `CHANGELOG.md` | Một dòng Đợt 15 | 9 |

---

## Task 1: preload.md - canonical "Reachability khi ghi" + Tier 1 mở rộng

**Files:**
- Modify: `skills/okr-shared/references/preload.md` (hiện 53 dòng; thêm bước ls vào Tier 1, thêm dòng bảng `context/index.md`, thêm 1 section lớn cuối file)
- Modify: `skills/okr-shared/SKILL.md` (overview trỏ đúng Preload Contract mới, không chép canonical)

Đây là nền của cả đợt. Mọi skill khác link về đây.

- [ ] **Step 1: Dọn em-dash cũ rồi thêm bước `ls -1 .okr/` vào Tier 1**

Trước hết dọn 4 em-dash cũ trong `preload.md` (acceptance mục 12 áp cho cả file): hai heading `## Tier 1`/`## Tier 2` đổi ký tự nối sau số thành dấu hai chấm (kết quả `## Tier 1: FULL`, `## Tier 2: MINIMAL`); hai dòng bảng Tier 2 (`okr-capture`, `okr-retro`) đổi ký tự nối mệnh đề thành dấu phẩy (vd `(toàn bộ), để phân loại type...`). Các SKILL khác trỏ "Preload Contract Tier 1" bằng chữ, không trỏ anchor có chữ FULL, nên đổi tên heading KHÔNG phá link.

Sau đó, ngay dưới heading `## Tier 1: FULL` và dòng "Áp dụng: ...", chèn đoạn sau TRƯỚC bảng nguồn (bảng bắt đầu bằng `| Nguồn | Độ sâu nạp | Dùng để |`):

````markdown
**Bước 0 (bắt buộc, trước bảng):** `Bash ls -1 .okr/` (cấp 1, KHÔNG đệ quy). Chụp cây thư mục gốc một lần. Đây là chủ thể DUY NHẤT để: (a) phát hiện `context/` tồn tại nhằm conditional load `context/index.md`, (b) phát hiện file lẻ ở root / thư mục con lạ cho Light audit của `okr-analyze` (xem "Reachability khi ghi"). Output ngắn, token-cheap.
````

- [ ] **Step 2: Thêm dòng `context/index.md` vào bảng Tier 1**

Trong bảng Tier 1, ngay sau dòng `| lessons/index.md | **TOÀN BỘ** | Áp dụng bài học ... |`, thêm dòng:

````markdown
| `context/index.md` | **TOÀN BỘ**, conditional: chỉ khi `context/` tồn tại theo `ls` Bước 0 | Reachability khi ghi: biết file context cross-cutting nào tồn tại + khi nào nạp body (xem "Reachability khi ghi") |
````

- [ ] **Step 3: Thêm dòng `context/<slug>.md` body vào mục "KHÔNG preload"**

Trong bảng dưới heading `## KHÔNG preload (load on-demand bởi skill cần)`, sau dòng `lessons/skill/*, lessons/project/*`, thêm:

````markdown
| `context/<slug>.md` body | Skill đọc khi cột "Khi nào cần đọc" trong `context/index.md` khớp việc hiện tại (xem "Áp dụng context"). Index chỉ chứa 4 trường, body on-demand |
````

- [ ] **Step 4: Thêm section lớn "Reachability khi ghi" vào cuối file**

Thêm vào CUỐI `preload.md` (sau section "Áp dụng lessons") nguyên văn:

`````markdown
## Reachability khi ghi (chống file mồ côi)

Canonical. Đối xứng với reachability khi ĐỌC (phần trên bảo đảm nạp đủ nền). Phần này bảo đảm file MỚI sinh ra trong `.okr/` không thành mồ côi. `sot-ownership.md` / `schemas.md` / `CLAUDE.md` đối chiếu bản đồ neo dưới đây, không định nghĩa khác.

> Mọi file/thư mục sinh trong `.okr/` phải **reachable** từ gốc preload, qua **link/đăng ký** hoặc qua **vị trí cấu trúc đã biết**. Không reachable = mồ côi = không hợp lệ.

Hai cơ chế reachable:
- **Qua link/đăng ký:** path xuất hiện trong một SOT đã preload (Roadmap link trong `plan.md`, cột `Resource` của `resources.md`, entry trong `context/index.md`, link trong `lessons/index.md`).
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

Mọi file/thư mục NGOÀI tập trên (file lẻ ở root, thư mục con lạ) là ứng viên mồ côi, phải neo qua link/đăng ký.

> Thuật ngữ: "gốc preload" (Tier 1) RỘNG hơn "3 file SOT đơn" (gồm thêm `lessons/index.md`, scan `actions/`/`inbox/`, conditional `context/index.md`). Khi nói audit theo vị trí, dùng đúng cụm "file SOT đơn ở root", KHÔNG gọi tắt thành "gốc preload".

### Cây quyết định gate khi tạo file (theo VỊ TRÍ GHI)

Tiêu chí "cần dòng confirm neo" định nghĩa theo VỊ TRÍ, KHÔNG theo trạng thái "đã neo chưa" (tránh vòng lặp định nghĩa). Hai nhánh:

- **Nhánh 1 (KHÔNG cần dòng confirm riêng):** file tạo trong thư mục cấu trúc đã biết qua đúng flow chuẩn của thư mục đó, mà flow đã bao gồm bước neo. Cụ thể: action vào `actions/` (`okr-plan` ghi Roadmap link), lesson vào `lessons/` (`okr-retro` ghi index), inbox vào `inbox/`, log vào `log/`. Neo là bước bắt buộc sẵn trong flow.
- **Nhánh 2 (BẮT BUỘC dòng confirm + chỉ rõ đích neo):** file hoặc nguồn ở vị trí KHÁC: deliverable file riêng, tài liệu DÙNG, file `context/`, hoặc bất kỳ file ad-hoc. Skill ghi chèn vào **confirm table đã có** (không thêm phase mới) 1 dòng:

  > Sẽ tạo/đăng ký `<path hoặc URL>`, neo vào `<đích: resources / Output-Deliverable / context-index>`, vai trò `<1 câu>`.

File `context/` thuộc nhánh 2 (đích neo = `context/index.md` entry); dòng confirm hiển thị việc tạo file + ghi entry. Tài liệu DÙNG dạng URL thuộc nhánh 2 nhưng là đăng ký resource, không tạo file.

`okr-init`/`okr-plan`/`okr-track` đã có confirm table nên chỉ THÊM dòng. `okr-capture` (Tier 2, không có confirm table) KHÔNG tham gia tạo file context (chỉ ghi inbox); việc nâng inbox item thành file context giao `okr-track` lúc xử lý inbox.

### Trigger-load trong hệ prompt-only + mệnh lệnh "Áp dụng context"

Hệ KHÔNG có executor. Cột "Khi nào cần đọc" trong `context/index.md` KHÔNG tự kích hoạt. Agent đọc index đã preload, thấy mô tả khớp việc hiện tại thì TỰ nạp body file đó, giống nạp action body từ Roadmap link hôm nay.

> **Áp dụng context** (song song với "Áp dụng lessons"): nếu `context/index.md` đã nạp, trước khi xử lý action / đề xuất / ghi, quét cột "Khi nào cần đọc"; mục nào khớp chủ đề việc hiện tại thì đọc body file đó TRƯỚC khi làm.

### Giữ path deliverable bền qua track ghi đè

`## Output/Deliverable` là SOT của `okr-track` (ghi đè output thực tế). Nếu deliverable là file riêng, `okr-track` khi ghi đè PHẢI giữ/cập nhật dòng path file, không làm đứt link reachable. Quy ước: dòng `Path: <đường dẫn>` đặt ở ĐẦU section, phần mô tả output free-form bên dưới. Audit dựa vào dòng `Path:` này.
`````

- [ ] **Step 5: Cập nhật `okr-shared/SKILL.md` để trỏ đúng contract mới**

Trong `skills/okr-shared/SKILL.md`, bảng `References`, thay dòng `references/preload.md` bằng:

````markdown
| `references/preload.md` | Preload Contract: context nền phải nạp trước khi chạy (Tier 1 full / Tier 2 minimal), `context/index.md` conditional, và Reachability khi ghi | Mọi skill (chạy ở đầu flow) |
````

Trong section `Khi nào đọc gì`, thay bullet `Mọi skill (đầu flow)` bằng:

````markdown
- **Mọi skill (đầu flow)**: `preload.md` xác định context nền phải nạp trước khi thao tác. Idempotent: qua harness thì đã nạp, gọi lẻ thì tự nạp phần thiếu. Tier 1 (analyze/init/plan/track/harness) full + conditional `context/index.md`; Tier 2 (capture/retro) minimal.
````

Ngay sau đoạn `lessons/index.md là một phần...`, thêm đoạn ngắn:

````markdown
`context/index.md` là phần conditional của Tier 1 khi `context/` tồn tại. Chỉ preload index. Body `context/<slug>.md` đọc on-demand theo `preload.md` "Áp dụng context". Không định nghĩa lại bản đồ neo ở đây.
````

- [ ] **Step 6: Verify nội dung + không em-dash**

Run:
```bash
grep -n "Reachability khi ghi\|Bản đồ neo\|Vùng cấu trúc đã biết\|Áp dụng context\|ls -1 .okr/\|context/index.md" skills/okr-shared/references/preload.md
rg -n "[\u{2013}\u{2014}]" skills/okr-shared/references/preload.md skills/okr-shared/SKILL.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: dòng đầu in ra ≥7 match (gồm cả Tier 1 và section mới). Dòng sau in `OK: khong co em-dash/en-dash`.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-shared/SKILL.md skills/okr-shared/references/preload.md
git commit -m "feat(okr-shared): them Reachability khi ghi + ls + conditional context vao preload (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 2: sot-ownership.md - ownership cho tài liệu, context, audit

**Files:**
- Modify: `skills/okr-shared/references/sot-ownership.md` (hiện 21 dòng; thêm 4 dòng vào bảng + 1 ghi chú)

- [ ] **Step 1: Thêm 4 dòng ownership vào bảng**

Trong bảng (header `| Field | Skill được phép sửa |`), sau dòng `| Bài học (.okr/lessons/**...) | okr-retro |`, thêm:

````markdown
| `## Tài liệu & Knowledge Base` (resources.md): đăng ký nguồn DÙNG | `okr-init` `update-resource`                     |
| `context/<slug>.md` (nội dung cross-cutting)                     | Owner = skill tạo file (`okr-init`/`okr-plan`/`okr-track`), 1 file 1 owner |
| `context/index.md` entry (đăng ký file context)                 | Append-only đa-skill. KEY = cột `Path`. Mỗi entry single-owner = skill tạo. Cấm sửa entry owner khác |
| Reachability audit (phát hiện mồ côi / link chết)               | `okr-analyze` (read-only, Light + Deep)          |
````

- [ ] **Step 2: Thêm ghi chú model đa-skill dưới bảng**

Ngay trên dòng cuối `> Bảng này là bản canonical. Load cùng skill okr khi chạy.`, chèn:

````markdown
> **`context/index.md` đa-skill nhưng không vi phạm "1 field 1 skill":** index chỉ là TẬP HỢP entry single-owner. Mỗi entry có 1 owner (skill tạo file), chỉ owner đó được sửa entry của mình. Nhiều skill ghi cùng bảng, nhưng không skill nào sửa entry của skill khác. Bản đồ neo canonical: `preload.md` "Reachability khi ghi".
````

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "Tài liệu & Knowledge Base\|context/<slug>.md\|context/index.md\|Reachability audit" skills/okr-shared/references/sot-ownership.md
rg -n "[\u{2013}\u{2014}]" skills/okr-shared/references/sot-ownership.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥4 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-shared/references/sot-ownership.md
git commit -m "feat(okr-shared): them ownership tai lieu/context/audit vao sot-ownership (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: schemas.md - section "Context layer" + mapping `thought → context`

**Files:**
- Modify: `skills/okr-shared/references/schemas.md` (hiện 151 dòng; thêm 1 dòng vào bảng "Inbox type → Delegate mapping" + 1 section mới cuối file)

- [ ] **Step 1: Thêm dòng `thought → context` vào "Inbox type → Delegate mapping"**

Trong bảng dưới heading `## Inbox type → Delegate mapping`, sau dòng `| `thought` → log | Có (append log) | - |`, thêm:

````markdown
| `thought` → context | Có (`okr-track` tạo `context/<slug>.md` + entry `context/index.md`) | - |
````

- [ ] **Step 2: Thêm section "Context layer" vào cuối file**

Thêm vào CUỐI `schemas.md` nguyên văn:

`````markdown
---

## Context layer (`.okr/context/`)

> Canonical FORMAT + ranh giới ngữ nghĩa. Cách NEO file về preload: xem `preload.md` "Reachability khi ghi". Các skill link về đây, không chép lại.

Nhà độc lập first-class cho **nội dung cross-cutting do chính dự án tạo ra trong `.okr/`** (tri thức HOẶC data/state phụ trợ), không thuộc objective/plan/resources/lessons/deliverable. Song song `lessons/`, có index riêng auto-load (conditional preload khi `context/` tồn tại).

### Cấu trúc

````
.okr/context/
├── index.md          # Registry nhẹ. Conditional preload khi context/ tồn tại.
└── <slug>.md         # 1 file/chủ đề
````

### Format index.md (4 trường)

````markdown
# Context Index

> Nội dung cross-cutting của project, không thuộc objective/plan/resources/lessons.
> Conditional preload khi thư mục context/ tồn tại. Body file nạp on-demand.

| Path | Vai trò | Khi nào cần đọc | Owner |
| --- | --- | --- | --- |
| context/glossary.md | Định nghĩa thuật ngữ riêng của project | Khi gặp thuật ngữ lạ trong action/objective | okr-track |
| context/playbook-deploy.md | Quy trình deploy chuẩn | Khi làm action liên quan deploy | okr-plan |
````

- `Path`: đường dẫn tương đối từ `.okr/`. Là KEY.
- `Vai trò`: 1 câu, file là gì. SOT của dòng index (sửa file thì cập nhật lại dòng).
- `Khi nào cần đọc`: mô tả ngôn ngữ tự nhiên để agent tự quyết nạp body (prompt-only, không executor; xem `preload.md` "Trigger-load" + "Áp dụng context").
- `Owner`: skill tạo entry.

### Model ghi index.md (cấu trúc ghi-đa-skill ĐẦU TIÊN của hệ)

KHÔNG có tiền lệ "owner-per-entry như log/" (thực tế `log/` do duy nhất `okr-track` ghi). Đặc tả tường minh:

- **KEY = cột `Path`**, unique. Idempotent: Path đã có entry thì cập nhật tại chỗ, KHÔNG thêm dòng trùng.
- **Append:** entry mới thêm cuối bảng.
- **Single-owner per entry:** mỗi entry có 1 `owner` (skill tạo file). Skill KHÔNG sửa entry của owner khác (chỉ sửa entry mình tạo). Index đa-skill-ghi nhưng mỗi entry vẫn single-owner, KHÔNG vi phạm "1 field 1 skill".
- Bảng nhẹ, body on-demand (theo mẫu `lessons/index.md`).

### Ranh giới ngữ nghĩa: resources.md vs context/

- **`resources.md` `## Tài liệu & Knowledge Base`** = *con trỏ tới nguồn bạn DÙNG/THAM CHIẾU*. Mỗi dòng trỏ tới một resource (thường ngoài `.okr/` hoặc deliverable cần tra). KHÔNG chứa nội dung tri thức.
- **`context/`** = *nhà chứa nội dung cross-cutting do chính dự án tạo ra trong `.okr/`* (tri thức HOẶC data/state phụ trợ).

Cây phân loại khi tạo file:
1. Là sản phẩm của một action? → `## Output/Deliverable` của action đó.
2. Là con trỏ tới nguồn/tài liệu để DÙNG? → 1 dòng trong `## Tài liệu & Knowledge Base`.
3. Là bài học? → `lessons/` (qua `okr-retro`).
4. Còn lại (nội dung/data cross-cutting do dự án sinh) → `.okr/context/` + entry index.
`````

> Lưu ý fence cho engineer: section trên dùng fence 5-backtick bao ngoài vì bên trong có block 4-backtick (cấu trúc thư mục) và 3-backtick lồng. Khi dán vào `schemas.md`, giữ nguyên các cấp fence để hiển thị đúng.

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "Context layer\|Format index.md\|Model ghi index.md\|Ranh giới ngữ nghĩa\|thought` → context" skills/okr-shared/references/schemas.md
rg -n "[\u{2013}\u{2014}]" skills/okr-shared/references/schemas.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥5 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-shared/references/schemas.md
git commit -m "feat(okr-shared): them Context layer schema + mapping thought->context (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: okr-harness - Phase 1 preload thêm ls + conditional context

**Files:**
- Modify: `skills/okr-harness/SKILL.md` (sửa block "Preload" trong Phase 1, dòng 39-50)
- Modify: `skills/okr-harness/references/flows.md` (Inbox flow có nhánh `thought → context`)

- [ ] **Step 1: Thêm `ls` Bước 0 và `context/index.md` vào block preload code**

Trong `## Phase 1` → `### Preload`, block code (bắt đầu `objective.md → frontmatter...`), thêm dòng `ls` ở ĐẦU block:

````
ls -1 .okr/       → cấp 1, KHÔNG đệ quy: phát hiện context/ + file lẻ/thư mục lạ (cho audit)
````

Sau dòng `lessons/index.md  → đọc TOÀN BỘ (bài học, auto-load mỗi phiên)`, thêm:

````
context/index.md  → conditional: nạp TOÀN BỘ nếu context/ tồn tại (như lessons/index.md)
````

- [ ] **Step 2: Cập nhật câu mô tả ngay dưới block**

Tìm câu (dòng 48): `Chỉ frontmatter (trừ `resources.md` + `lessons/index.md` đọc toàn bộ). Không đọc objective/plan body...`. Thay bằng:

````markdown
Chỉ frontmatter (trừ `resources.md`, `lessons/index.md`, và `context/index.md` khi tồn tại đọc toàn bộ). Không đọc objective/plan body, action body, log, archive, file detail bài học, hay `context/<slug>.md` body (on-demand, xem contract mục "KHÔNG preload").
````

- [ ] **Step 3: Thêm câu link bản đồ neo + "Áp dụng context"**

Ngay sau đoạn "Lessons đã nạp → dùng làm context định hướng..." (dòng 50), thêm 1 đoạn mới:

````markdown
`context/index.md` (nếu nạp) → áp dụng quy tắc `okr-shared/references/preload.md` "Áp dụng context": trước khi skill ghi/đề xuất, quét cột "Khi nào cần đọc", nạp body file khớp. Mọi file mới sinh trong phiên phải reachable theo **bản đồ neo** (canonical: `preload.md` "Reachability khi ghi").
````

- [ ] **Step 4: Cập nhật inbox flow trong `references/flows.md`**

Trong `skills/okr-harness/references/flows.md`, section `## 6. Inbox flow`, thay dòng sequence hiện tại:

````markdown
    H->>H: Xử lý: blocker → tự ghi, thought → log/giữ
````

bằng:

````markdown
    H->>H: Xử lý: blocker → tự ghi, thought → action/log/context/giữ
````

Ngay dưới sơ đồ sequence của Inbox flow, thêm ghi chú:

````markdown
`thought` có 4 nhánh xử lý canonical ở `okr-track/references/flow-inbox.md`: tạo action, append log, nâng thành `context/<slug>.md` + `context/index.md`, hoặc giữ inbox.
````

- [ ] **Step 5: Verify**

Run:
```bash
grep -n "ls -1 .okr/\|context/index.md\|bản đồ neo\|Áp dụng context\|thought.*context" skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md
rg -n "[\u{2013}\u{2014}]" skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥5 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 6: Commit**

```bash
git add skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md
git commit -m "feat(okr-harness): preload Phase 1 them ls + conditional context/index (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 5: okr-analyze - conditional đọc context + Reachability audit (Light/Deep)

**Files:**
- Modify: `skills/okr-analyze/SKILL.md` (thêm dòng vào bảng "Đọc song song"; thêm mục vào "Phát hiện issues"; thêm 1 section "Reachability audit")

- [ ] **Step 1: Thêm `context/` vào bảng "Đọc song song"**

Trong section `## Đọc state`, dòng `Preload Contract Tier 1` đang liệt kê các nguồn trong ngoặc. Thay cụm:

````markdown
`objective.md`/`plan.md` frontmatter, `resources.md` full body, actions/inbox count, `lessons/index.md`
````

bằng:

````markdown
`objective.md`/`plan.md` frontmatter, `resources.md` full body, actions/inbox count, `lessons/index.md`, conditional `context/index.md` khi `context/` tồn tại
````

Sau đó, trong bảng `| File | Đọc gì | Ghi chú |`, sau dòng `| log/ | ... |`, thêm:

````markdown
| `context/index.md` | TOÀN BỘ (conditional: chỉ khi `context/` tồn tại) | Reachability + "Áp dụng context": nạp body `<slug>.md` khi việc hiện tại khớp cột "Khi nào cần đọc" |
````

- [ ] **Step 2: Thêm mục Reachability vào "Phát hiện issues"**

Trong section `## Phát hiện issues`, sau mục `7. Capacity / xung đột tài nguyên...`, thêm:

````markdown
8. **Reachability (chống file mồ côi)**: chạy "Reachability audit" (xem section dưới). Light = ls cấp 1 (cảnh báo nhẹ). Deep/closure = thuật toán đầy đủ (báo mồ côi + link chết + context lệch).
````

- [ ] **Step 3: Thêm section "Reachability audit" sau "Mode trace"**

Sau section `## Mode trace (xem lại lịch sử, read-only)` (kết thúc trước `## Error handling`), chèn section mới nguyên văn:

`````markdown
## Reachability audit (read-only, chống file mồ côi)

Backstop cho "Reachability khi ghi" (canonical `../okr-shared/references/preload.md`). Read-only: CHỈ báo + đề xuất; việc neo do `okr-init`/`okr-plan`/`okr-track` làm (giữ SOT ownership, đúng "track đề xuất, init/plan áp dụng").

### Light (dashboard, mỗi phiên có plan)

Tái dùng kết quả `ls -1 .okr/` (đã chạy ở Preload Contract Bước 0). Chỉ soi CẤP 1, KHÔNG đệ quy, KHÔNG đọc link-set, KHÔNG khẳng định mồ côi. Whitelist đã biết ở root:

- File: `objective.md`, `resources.md`, `plan.md`.
- Thư mục: `actions`, `inbox`, `log`, `lessons`, `context`.

Bất kỳ file lẻ / thư mục con NGOÀI whitelist trên:

````
ℹ️ Có file/thư mục ở vị trí lạ: <tên>. Chạy review sâu để xác minh + cách neo.
````

Giữ claim token-cheap (1 lệnh ls cấp 1, output ngắn).

### Deep / closure (thuật toán reachable set đầy đủ)

**Bước 1. Liệt kê thực tế:** `ls -R .okr/` (đệ quy). Chỉ lấy TÊN file, KHÔNG đọc body của `archive/`, `log/` (tôn trọng invisible-by-default: audit chỉ kiểm tồn tại path).

**Bước 2. Tập reachable theo vị trí:** 3 file SOT đơn ở root + các thư mục cấu trúc đã biết (`actions/`, `actions/archive/`, `inbox/`, `log/`, `lessons/`, `context/`). File theo độ sâu chuẩn của từng thư mục = reachable theo vị trí. Riêng `context/` vẫn phải qua Bước 5 để kiểm entry index, dù không bị gọi là mồ côi ở Bước 4.

**Bước 3. Tập reachable qua link/đăng ký** (bắt file ngoài thư mục đã biết + để check link chết):
- Cột `Resource` của `resources.md` `## Tài liệu & Knowledge Base`.
- Dòng `Path:` trong `## Output/Deliverable` của action ACTIVE **và** action trong `actions/archive/` (deep/closure vốn đọc archive).
- `context/index.md` entries (cột `Path`).
- Link trong `lessons/index.md` (mục còn hiệu lực; bỏ qua mục "ported/obsolete", xem Bước 6).
- Roadmap link trong `plan.md` body (đọc on-demand) để check Roadmap link chết.

**Bước 4. Mồ côi = file thực tế KHÔNG thuộc Bước 2 và KHÔNG thuộc Bước 3.** Điển hình: file ở `.okr/` root ngoài 3 SOT, hoặc trong thư mục con lạ (vd `.okr/research/`). Báo path + gợi ý neo theo bản đồ neo (tài liệu DÙNG → resources; cross-cutting → context/).

**Bước 5. Kiểm tra nội bộ context/:** nếu `context/` tồn tại mà thiếu `context/index.md` → báo "context lệch". Mọi `context/<slug>.md` (trừ `index.md`) phải có entry trong `index.md`; mọi entry phải trỏ file tồn tại. Lệch → báo "context lệch".

**Bước 6. Kiểm tra link chết:** mọi path Bước 3 phải trỏ file tồn tại.
- Roadmap link trỏ `actions/*` không tồn tại, entry context trỏ file đã xóa, cột Resource trỏ file `.okr/` không tồn tại → **lỗi cứng**.
- File NGOÀI `.okr/` không tồn tại (deliverable ngoài chưa tạo), lesson ported/obsolete đã xóa khỏi `.okr/` (hàng đợi port hợp lệ) → **cảnh báo nhẹ**, không lỗi cứng.

Output mỗi vấn đề: path + loại (mồ côi / link chết / context lệch) + gợi ý neo + skill áp dụng. KHÔNG tự sửa.
`````

- [ ] **Step 4: Verify**

Run:
```bash
grep -n "Reachability audit\|vị trí lạ\|reachable set\|context lệch\|Mồ côi" skills/okr-analyze/SKILL.md
rg -n "[\u{2013}\u{2014}]" skills/okr-analyze/SKILL.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥5 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-analyze/SKILL.md
git commit -m "feat(okr-analyze): them Reachability audit Light/Deep + conditional context (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 6: okr-init - guard context + đăng ký tài liệu DÙNG + confirm gate

**Files:**
- Modify: `skills/okr-init/SKILL.md` (sửa dòng Nguyên tắc Preload Contract)
- Modify: `skills/okr-init/references/flow-update-resource.md` (Phase 4 confirm + ghi chú tài liệu DÙNG)
- Modify: `skills/okr-init/references/data-format.md` (cross-ref ranh giới `resources.md` vs `context/`)

- [ ] **Step 1: Thêm guard conditional-load context vào SKILL Nguyên tắc**

Trong `skills/okr-init/SKILL.md` section `## Nguyên tắc`, dòng bắt đầu `- Preload Contract Tier 1 (...): trước khi đề xuất/ghi, đảm bảo nền Tier 1 đã nạp (...)`. Trong dấu ngoặc liệt kê, thay `lessons/index.md` toàn bộ)` thành:

````markdown
`lessons/index.md` toàn bộ, + conditional `context/index.md` khi `context/` tồn tại)
````

Sau đó, thêm 1 bullet mới ngay dưới bullet Preload đó:

````markdown
- **Reachability khi ghi** (`../okr-shared/references/preload.md`): tài liệu/nguồn DÙNG đăng ký 1 dòng vào `## Tài liệu & Knowledge Base` của `resources.md` (cột `Resource` = path/URL). Nếu tạo file mới ở vị trí lạ (file `context/` hoặc ad-hoc) → bắt buộc dòng confirm gate nhánh 2 (xem flow). Áp dụng "Áp dụng context" khi đề xuất.
````

- [ ] **Step 2: Thêm dòng confirm gate vào flow-update-resource Phase 4**

Trong `skills/okr-init/references/flow-update-resource.md`, section `### Phase 4: CONFIRM diff`, ngay TRƯỚC dòng `Xác nhận? (y / sửa / huỷ)` bên trong block confirm, thêm dòng gate (chỉ hiển thị khi có tạo file mới ở vị trí lạ):

````markdown
[Nếu thay đổi gồm tạo file mới ở vị trí lạ, vd nâng nội dung thành context/ hoặc thêm deliverable file riêng, in thêm dòng neo TRƯỚC "Xác nhận?":]
> Sẽ tạo `.okr/context/<slug>.md`, neo vào `context/index.md` entry, vai trò `<1 câu>`.
````

- [ ] **Step 3: Làm rõ ranh giới tài liệu DÙNG vs context trong Phase 5 (cảnh báo)**

Trong `flow-update-resource.md` section `### Phase 5: Áp dụng + cảnh báo`, sau mục 2 (cross-check), thêm mục:

````markdown
3. **Phân loại đúng nhà** (ranh giới `schemas.md` "Context layer"): nếu user mô tả một *con trỏ tới nguồn DÙNG* (link, file cần tra) → vào `## Tài liệu & Knowledge Base`. Nếu là *nội dung cross-cutting do dự án tạo* (glossary, playbook) → KHÔNG nhét vào resources; tạo `context/<slug>.md` + entry `context/index.md` (owner = okr-init) kèm dòng confirm gate ở Phase 4.
````

(Lưu ý: mục cảnh báo cũ đánh số 3 trong Phase 5 hiện tại là dòng "Hiển thị: Đã update resource...". Giữ nguyên dòng đó; mục mới chèn TRƯỚC nó và đánh số lại nếu cần, hoặc dùng bullet không số để tránh xung đột đánh số. Ưu tiên bullet không số.)

Đồng thời, trong `skills/okr-init/references/data-format.md`, mục "Body sections" của `resources.md` (khoảng dòng 62-68) liệt kê section `## Tài liệu & Knowledge Base`. Ngay dưới khối liệt kê đó, thêm 1 dòng cross-ref ranh giới:

````markdown
> `## Tài liệu & Knowledge Base` chỉ chứa *con trỏ tới nguồn DÙNG/THAM CHIẾU* (thường ngoài `.okr/` hoặc deliverable cần tra). Nội dung cross-cutting do dự án TẠO RA (glossary, playbook, bảng tra) thuộc `.okr/context/`, KHÔNG nhét vào đây. Ranh giới canonical: `../../okr-shared/references/schemas.md` "Context layer".
````

- [ ] **Step 4: Verify**

Run:
```bash
grep -n "context/index.md\|Reachability khi ghi\|neo vào\|Phân loại đúng nhà\|Context layer" skills/okr-init/SKILL.md skills/okr-init/references/flow-update-resource.md skills/okr-init/references/data-format.md
rg -n "[\u{2013}\u{2014}]" skills/okr-init/SKILL.md skills/okr-init/references/flow-update-resource.md skills/okr-init/references/data-format.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥4 match; `OK: khong co em-dash/en-dash`.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-init/SKILL.md skills/okr-init/references/flow-update-resource.md skills/okr-init/references/data-format.md
git commit -m "feat(okr-init): guard context + dang ky tai lieu DUNG + confirm gate (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 7: okr-plan - guard + dòng Path deliverable + confirm gate + tạo context

**Files:**
- Modify: `skills/okr-plan/SKILL.md` (Nguyên tắc Preload Contract)
- Modify: `skills/okr-plan/references/task-format.md` (quy ước dòng `Path:` trong `## Output/Deliverable`)
- Modify: `skills/okr-plan/references/flow-new.md` (Phase 3 confirm gate)
- Modify: `skills/okr-plan/references/flow-update.md` (Phase 4 confirm gate)

- [ ] **Step 1: Guard conditional-load context vào SKILL**

Trong `skills/okr-plan/SKILL.md` section `## Nguyên tắc`, dòng Preload Contract, trong ngoặc liệt kê thay `lessons/index.md` toàn bộ)` thành:

````markdown
`lessons/index.md` toàn bộ, + conditional `context/index.md` khi `context/` tồn tại)
````

Thêm 1 bullet mới dưới đó:

````markdown
- **Reachability khi ghi** (`../okr-shared/references/preload.md`): deliverable là file riêng → ghi dòng `Path:` ở đầu `## Output/Deliverable` (xem `task-format.md`). Tạo file `context/` (playbook, bảng tra cho plan) → entry `context/index.md` (owner = okr-plan) + dòng confirm gate nhánh 2. Action vào `actions/` là nhánh 1 (KHÔNG cần dòng confirm riêng, Roadmap link đã neo).
````

- [ ] **Step 2: Thêm quy ước dòng `Path:` vào task-format.md**

Trong `skills/okr-plan/references/task-format.md`, trong block template markdown, thay đoạn `## Output/Deliverable` hiện tại:

````markdown
## Output/Deliverable

[Mô tả sản phẩm đầu ra khi task hoàn thành. Phải cụ thể: file gì, ở đâu, chứa gì]
````

thành:

````markdown
## Output/Deliverable

Path: [chỉ khi deliverable là FILE riêng: đường dẫn file, vd `docs/report.md` hoặc `.okr/context/...`. Không phải file riêng thì bỏ dòng Path]
[Mô tả sản phẩm đầu ra khi task hoàn thành. Phải cụ thể: file gì, ở đâu, chứa gì]
````

Ngay sau block template (sau dòng đóng ```` ``` ````), thêm ghi chú:

````markdown
> **Dòng `Path:` để reachability** (`../../okr-shared/references/preload.md` "Giữ path deliverable bền"): nếu deliverable là file riêng, dòng `Path: <đường dẫn>` đặt ở ĐẦU section. `okr-track` khi ghi đè output thực tế PHẢI giữ/cập nhật dòng này, không làm đứt link. Audit Deep dựa vào dòng `Path:`. Deliverable không phải file (vd "KR1 đạt 5k DAU") thì không có dòng `Path:`.
````

- [ ] **Step 3: Confirm gate vào flow-new Phase 3**

Trong `skills/okr-plan/references/flow-new.md`, section `## Phase 3: CONFIRM Plan`, trong block confirm, TRƯỚC dòng `Xác nhận? (y / sửa <ID>: ...)`, thêm:

````markdown
[Nếu plan tạo deliverable file riêng (action có dòng Path:) hoặc file context/, in dòng neo cho mỗi file TRƯỚC "Xác nhận?":]
Neo file mới
  - Sẽ tạo/đăng ký `<path>`, neo vào `<Output-Deliverable của AXXX / context-index>`, vai trò `<1 câu>`.
````

- [ ] **Step 4: Confirm gate vào flow-update Phase 4**

Trong `skills/okr-plan/references/flow-update.md`, section `## Phase 4: CONFIRM diff`, trong block confirm, TRƯỚC dòng `Xác nhận? (y / sửa / huỷ)`, thêm:

````markdown
[Nếu thay đổi tạo deliverable file riêng hoặc file context/, in dòng neo TRƯỚC "Xác nhận?":]
Neo file mới
  - Sẽ tạo/đăng ký `<path>`, neo vào `<Output-Deliverable của AXXX / context-index>`, vai trò `<1 câu>`.
````

- [ ] **Step 5: Thêm bước tạo context vào flow-new Phase 4 (ghi file)**

Trong `flow-new.md` section `## Phase 4: Ghi file`, sau mục 5 (Render bảng Roadmap), thêm:

````markdown
6. **Nếu tạo file context/**: ghi `.okr/context/<slug>.md` (nội dung) + thêm/cập nhật entry trong `.okr/context/index.md` (KEY = `Path`, idempotent, owner = okr-plan). Format index: `okr-shared/references/schemas.md` "Context layer". Tạo `context/index.md` nếu chưa có.
````

- [ ] **Step 6: Thêm bước tạo context vào flow-update Phase 5 (áp dụng)**

Trong `flow-update.md` section `## Phase 5: Áp dụng`, sau mục 3 (Re-render Roadmap), thêm:

````markdown
4. **Nếu tạo file context/**: ghi `.okr/context/<slug>.md` (nội dung) + thêm/cập nhật entry trong `.okr/context/index.md` (KEY = `Path`, idempotent, owner = okr-plan). Format index: `okr-shared/references/schemas.md` "Context layer". Tạo `context/index.md` nếu chưa có.
````

(Đánh số lại: mục "Đề xuất chạy okr-track..." hiện là 4 → thành 5.)

- [ ] **Step 7: Verify**

Run:
```bash
grep -n "Path:\|context/index.md\|Neo file mới\|Reachability khi ghi" skills/okr-plan/SKILL.md skills/okr-plan/references/task-format.md skills/okr-plan/references/flow-new.md skills/okr-plan/references/flow-update.md
rg -n "[\u{2013}\u{2014}]" skills/okr-plan/SKILL.md skills/okr-plan/references/task-format.md skills/okr-plan/references/flow-new.md skills/okr-plan/references/flow-update.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥6 match; `OK: khong co em-dash/en-dash`.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-plan/SKILL.md skills/okr-plan/references/task-format.md skills/okr-plan/references/flow-new.md skills/okr-plan/references/flow-update.md
git commit -m "feat(okr-plan): dong Path deliverable + confirm gate + tao context (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 8: okr-track - guard + giữ Path bền + nâng inbox thành context + confirm gate

**Files:**
- Modify: `skills/okr-track/SKILL.md` (Nguyên tắc Preload Contract)
- Modify: `skills/okr-track/references/data-format.md` (Output/Deliverable giữ dòng Path)
- Modify: `skills/okr-track/references/flow-shared.md` (Tier 1 shared nhắc conditional `context/index.md`)
- Modify: `skills/okr-track/references/flow-inbox.md` (nâng `thought` thành context)
- Modify: `skills/okr-track/references/flow-light.md` (confirm gate khi tạo context)

- [ ] **Step 1: Guard conditional-load context vào SKILL**

Trong `skills/okr-track/SKILL.md` section `## Nguyên tắc`, dòng Preload Contract, trong ngoặc liệt kê thay `lessons/index.md` toàn bộ)` thành:

````markdown
`lessons/index.md` toàn bộ, + conditional `context/index.md` khi `context/` tồn tại)
````

Thêm 1 bullet mới dưới đó:

````markdown
- **Reachability khi ghi** (`../okr-shared/references/preload.md`): ghi đè `## Output/Deliverable` PHẢI giữ/cập nhật dòng `Path:` (không làm đứt link). Nâng inbox item `thought` (tri thức cross-cutting) thành file `context/` + entry `context/index.md` (owner = okr-track) kèm dòng confirm gate. Áp dụng "Áp dụng context" khi xử lý action.
````

Trong bảng `## SOT quyền ghi`, thêm 2 dòng sau dòng `inbox/*.md`:

````markdown
| context/*.md     | Tạo file context từ inbox `thought` cross-cutting |
| context/index.md | Thêm/cập nhật entry owner = okr-track             |
````

- [ ] **Step 2: Cập nhật flow-shared Tier 1**

Trong `skills/okr-track/references/flow-shared.md`, dòng bắt đầu `SOT data Tier 1 (...) đã có từ orchestrator`, thay cụm trong ngoặc:

````markdown
objective/plan frontmatter, actions frontmatter, `resources.md` full body, inbox count, `lessons/index.md`
````

bằng:

````markdown
objective/plan frontmatter, actions frontmatter, `resources.md` full body, inbox count, `lessons/index.md`, conditional `context/index.md` khi `context/` tồn tại
````

Trong phần `Đọc thêm (on-demand, KHÔNG nằm trong preload):`, thêm bullet:

````markdown
- `.okr/context/<slug>.md` body: đọc khi `context/index.md` đã nạp và cột "Khi nào cần đọc" khớp việc hiện tại (xem `../../okr-shared/references/preload.md` "Áp dụng context").
````

- [ ] **Step 3: Giữ dòng Path bền trong data-format.md**

Trong `skills/okr-track/references/data-format.md`, section `### Progress fields`, mục `.okr/actions/AXXX-*.md`, dòng `- Body section `## Output/Deliverable`: ghi đè nội dung dự kiến...`. Thay bằng:

````markdown
- Body section `## Output/Deliverable`: ghi đè nội dung dự kiến bằng output thực tế (free-form markdown). Chỉ ghi khi user cung cấp output. **Giữ reachability**: nếu deliverable là file riêng, GIỮ/cập nhật dòng `Path: <đường dẫn>` ở ĐẦU section (xem `../../okr-shared/references/preload.md` "Giữ path deliverable bền"). Output thực tế là file mới → cập nhật `Path:` trỏ đúng; không có file riêng → không cần dòng `Path:`.
````

Trong cùng file, section `## Hai tầng dữ liệu`, thêm row:

````markdown
| **Context** | `.okr/context/` | Tạo từ inbox `thought` cross-cutting; body on-demand, index conditional preload |
````

- [ ] **Step 4: Nâng inbox `thought` thành context trong flow-inbox.md**

Trong `skills/okr-track/references/flow-inbox.md`, section `## Bước 2`, mục gợi ý cho `thought` (dòng `- `thought`: đủ rõ để thành action chưa?...`). Thay bằng:

````markdown
- `thought`: phân 4 nhánh. (a) Đủ rõ thành action → tạo action. (b) Là tri thức/nội dung cross-cutting dùng nhiều lần (glossary, playbook, bảng tra) → nâng thành `context/<slug>.md` + entry `context/index.md`. (c) Chỉ là ghi chú thời điểm → append log ngày. (d) Chưa rõ → giữ inbox.
````

Trong cùng file, section `## Bước 3`, sau đoạn "Xử lý từng item theo bảng Inbox type → Delegate mapping...", thêm:

````markdown
**Nâng `thought` thành context (nhánh b):** nếu item là tri thức cross-cutting, `okr-track` ghi `.okr/context/<slug>.md` (nội dung từ item) + thêm entry vào `.okr/context/index.md` (KEY = `Path`, idempotent, owner = okr-track, 4 trường theo `schemas.md` "Context layer"). Tạo `context/index.md` nếu chưa có. Trước khi ghi, hiển thị dòng confirm gate:
> Sẽ tạo `.okr/context/<slug>.md`, neo vào `context/index.md` entry, vai trò `<1 câu>`.
Xong → đổi `status: processed` cho inbox item.
````

- [ ] **Step 5: Confirm gate khi tạo context trong flow-light.md**

Trong `skills/okr-track/references/flow-light.md`, section bước `2. CONFIRM trước khi ghi`, sau block confirm bảng đầy đủ (nhánh ≥3 field), thêm ghi chú:

````markdown
> **Gate neo file mới**: nếu lần track này tạo hoặc đăng ký file ở vị trí lạ (vd nâng inbox thành `context/<slug>.md`, hoặc output thực tế là deliverable file riêng mới), chèn vào confirm 1 dòng TRƯỚC "Xác nhận?": `Sẽ tạo/đăng ký <path>, neo vào <context-index / Output-Deliverable AXXX>, vai trò <1 câu>.` (Xem `../../okr-shared/references/preload.md` "Cây quyết định gate". Action/log không cần vì là nhánh 1.)
````

- [ ] **Step 6: Verify**

Run:
```bash
grep -n "context/index.md\|Path:\|Reachability khi ghi\|Gate neo\|nâng thành\|Áp dụng context" skills/okr-track/SKILL.md skills/okr-track/references/data-format.md skills/okr-track/references/flow-shared.md skills/okr-track/references/flow-inbox.md skills/okr-track/references/flow-light.md
rg -n "[\u{2013}\u{2014}]" skills/okr-track/SKILL.md skills/okr-track/references/data-format.md skills/okr-track/references/flow-shared.md skills/okr-track/references/flow-inbox.md skills/okr-track/references/flow-light.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥8 match; `OK: khong co em-dash/en-dash`.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-track/SKILL.md skills/okr-track/references/data-format.md skills/okr-track/references/flow-shared.md skills/okr-track/references/flow-inbox.md skills/okr-track/references/flow-light.md
git commit -m "feat(okr-track): giu Path ben + nang inbox thanh context + confirm gate (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 9: CLAUDE.md + CHANGELOG.md - cây runtime + lịch sử

**Files:**
- Modify: `CLAUDE.md` (cây `.okr/` thêm `context/`; mục lịch sử Đợt 15)
- Modify: `CHANGELOG.md` (1 dòng Đợt 15)

- [ ] **Step 1: Thêm `context/` vào cây runtime `.okr/` trong CLAUDE.md**

Trong `CLAUDE.md` section `## Cấu trúc dữ liệu runtime`, trong block cây `.okr/`, sau dòng `├── log/                   # Append-only, type: [tracking|review|closure]`, thêm:

````
├── context/              # Tri thức/data cross-cutting do dự án tạo (index.md + <slug>.md)
````

(Giữ dòng `└── lessons/` là dòng cuối. Đảm bảo ký tự cây: dòng context dùng `├──`, lessons giữ `└──`.)

- [ ] **Step 2: Thêm tóm tắt bản đồ neo runtime vào CLAUDE.md**

Ngay dưới block cây `.okr/`, thêm section ngắn:

````markdown
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
````

- [ ] **Step 3: Thêm mục Đợt 15 vào "Lịch sử phát triển" CLAUDE.md**

Trong section `## Lịch sử phát triển`, sau mục `11. Đợt 14: Preload Contract...`, thêm:

````markdown
12. Đợt 15: Reachability khi ghi (chống file mồ côi). Đối xứng Đợt 14 (reachability khi đọc). Nguyên tắc lõi + bản đồ neo canonical ở `preload.md`: mọi file sinh trong `.okr/` phải reachable qua link/đăng ký hoặc vị trí cấu trúc đã biết. Nâng `context/` thành nhà độc lập first-class (index 4 trường, conditional preload, model ghi đa-skill owner-per-entry). Thêm `ls -1 .okr/` + conditional `context/index.md` vào Tier 1. Enforcement 2 lớp: gate dòng confirm lúc tạo (init/plan/track, nhánh 2 theo vị trí) + audit backstop read-only ở okr-analyze (Light ls cấp 1, Deep thuật toán reachable set 6 bước). Giữ dòng `Path:` deliverable bền qua track ghi đè. okr-capture/okr-retro không đổi. Chỉ runtime `.okr/`, tách dev-time.
````

- [ ] **Step 4: Cập nhật phần mô tả Preload trong CLAUDE.md (mục "Tiết kiệm token")**

Trong section `## Nguyên tắc thiết kế`, mục 6 "Tiết kiệm token, có chủ đích", câu cuối nhắc Preload Contract. Sau cụm `(skills/okr-shared/references/preload.md)`, thêm 1 câu:

````markdown
Đợt 15 bổ sung **Reachability khi ghi** (cùng file): mọi file mới phải neo về gốc preload, `context/` là nhà cho tri thức cross-cutting (conditional preload), audit backstop ở `okr-analyze`.
````

- [ ] **Step 5: Thêm dòng Đợt 15 vào CHANGELOG.md**

Trong `CHANGELOG.md`, trong bảng, thêm 1 hàng MỚI ngay dưới hàng header/separator (trên hàng `2026-06-04 | Đợt 14`), nội dung:

````markdown
| 2026-06-04 | Đợt 15: Reachability khi ghi (chống file mồ côi). (1) `preload.md` canonical: nguyên tắc reachability khi ghi + bản đồ neo + vùng cấu trúc đã biết + cây quyết định gate theo vị trí + trigger-load prompt-only + mệnh lệnh "Áp dụng context"; thêm `ls -1 .okr/` Bước 0 + conditional `context/index.md` vào Tier 1. (2) Tầng `context/` first-class: schema "Context layer" (`schemas.md`), index 4 trường, model ghi đa-skill owner-per-entry, mapping `thought → context`. (3) sot-ownership thêm tài liệu/context/audit. (4) Gate dòng confirm nhánh 2 ở init/plan/track + guard conditional-load. (5) okr-analyze: Reachability audit Light (ls cấp 1) + Deep (reachable set 6 bước). (6) Giữ dòng `Path:` deliverable bền. (7) okr-harness Phase 1 + flow inbox + CLAUDE.md cây runtime và tóm tắt bản đồ neo. | okr-shared (SKILL/preload/sot-ownership/schemas), okr-harness (SKILL/flows), okr-analyze, okr-init (+flow-update-resource/data-format), okr-plan (+task-format/flow-new/flow-update), okr-track (+data-format/flow-shared/flow-inbox/flow-light), CLAUDE.md | Vá lỗ hổng file mồ côi: file mới sinh trong `.okr/` không reachable từ preload → phiên sau bỏ sót. Cơ chế tổng quát (neo + audit) thay vì hard-code loại file |
````

- [ ] **Step 6: Verify**

Run:
```bash
grep -n "context/\|Đợt 15\|Reachability khi ghi\|Tài liệu & Knowledge Base\|Output/Deliverable" CLAUDE.md
grep -n "Đợt 15" CHANGELOG.md
rg -n "[\u{2013}\u{2014}]" CLAUDE.md CHANGELOG.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: CLAUDE.md ≥6 match, CHANGELOG.md 1 match; `OK: khong co em-dash/en-dash`.

- [ ] **Step 7: Commit**

```bash
git add CLAUDE.md CHANGELOG.md
git commit -m "docs: cay runtime context/ + lich su Dot 15 (CLAUDE.md, CHANGELOG.md)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 10: Acceptance verification (3 case + nhất quán bản đồ neo + em-dash)

Không sửa file ở task này; chỉ verify toàn cục theo tiêu chí Done spec mục 12. Nếu phát hiện lệch, quay lại task tương ứng sửa.

**Files:**
- Read-only verify trên toàn bộ file đã sửa.

- [ ] **Step 1: Verify "Reachability khi ghi" canonical 1 nơi + các skill link về**

Run:
```bash
grep -rn "Reachability khi ghi" skills/ CLAUDE.md
```
Expected: định nghĩa canonical (nguyên tắc + bản đồ neo + cây quyết định + trigger-load) CHỈ ở `skills/okr-shared/references/preload.md`. Các file khác (`okr-shared/SKILL.md`, okr-init/plan/track SKILL, okr-analyze, okr-harness, CLAUDE.md) chỉ **trỏ về** preload.md, KHÔNG chép lại bản đồ neo đầy đủ. Nếu thấy bản đồ neo bị copy nguyên bảng ra chỗ khác (ngoài 4 nơi cho phép: preload/sot-ownership/schemas/CLAUDE) → xoá, thay bằng link.

- [ ] **Step 2: Verify bản đồ neo nhất quán ở 4 nơi**

Run:
```bash
grep -n "Tài liệu & Knowledge Base\|context/<slug>.md\|context/index.md\|Output/Deliverable" skills/okr-shared/references/preload.md skills/okr-shared/references/sot-ownership.md skills/okr-shared/references/schemas.md CLAUDE.md
```
Expected: cùng danh sách loại file + cùng đích neo (tài liệu DÙNG → resources `## Tài liệu & Knowledge Base`; deliverable → `## Output/Deliverable` dòng `Path:`; cross-cutting → `context/<slug>.md` + `context/index.md`) xuất hiện nhất quán. Đọc 4 file, đối chiếu bằng mắt: không nơi nào mâu thuẫn đích neo.

- [ ] **Step 3: Verify tên section dùng đúng ký tự để grep**

Run:
```bash
grep -rn "## Tài liệu & Knowledge Base" skills/
```
Expected: chuỗi xuất hiện khớp ký tự (dấu `&`, có dấu tiếng Việt) ở `okr-init/references/data-format.md` (định nghĩa cũ) + preload.md + schemas.md + sot-ownership.md. Đây là chuỗi audit Deep grep, phải đồng nhất.
Các SKILL/flow khác có thể trỏ tới cùng chuỗi này, nhưng không được viết biến thể như `KnowledgeBase`, `KB`, hoặc thiếu dấu `&`.

- [ ] **Step 4: Verify trigger-load + "Áp dụng context" canonical 1 nơi**

Run:
```bash
grep -rn "Áp dụng context\|trigger-load\|Trigger-load" skills/
```
Expected: định nghĩa "Áp dụng context" (mệnh lệnh đầy đủ) canonical ở `preload.md`. `okr-shared/SKILL.md`, okr-harness/okr-analyze/okr-track chỉ trỏ về, không định nghĩa lại.

- [ ] **Step 5: Verify confirm gate (nhánh 2) ở 3 skill + KHÔNG có ở nhánh 1**

Run:
```bash
grep -rn "neo vào\|Neo file mới\|confirm gate\|Gate neo" skills/okr-init skills/okr-plan skills/okr-track
```
Expected: dòng confirm gate xuất hiện ở init (flow-update-resource), plan (flow-new + flow-update), track (flow-inbox + flow-light). Đọc kỹ: KHÔNG có dòng gate nào áp cho action vào `actions/` hay log vào `log/` (nhánh 1). Nếu có → xoá.

- [ ] **Step 6: Verify guard conditional-load context ở init/plan/track**

Run:
```bash
rg -n "conditional .*context/index.md" skills/okr-init/SKILL.md skills/okr-plan/SKILL.md skills/okr-track/SKILL.md
```
Expected: cả 3 SKILL có cụm conditional `context/index.md` trong dòng Preload Contract Tier 1 (đảm bảo chạy lẻ không qua harness vẫn nạp index, tránh lặp lỗ hổng entry-point-lẻ của Đợt 13/14).

- [ ] **Step 7: Walkthrough 3 case audit (đọc okr-analyze, mô phỏng bằng đầu)**

Đọc lại `skills/okr-analyze/SKILL.md` section "Reachability audit". Xác nhận 3 case spec mục 12 cho ra hành vi đúng:

- **Case A** (`.okr/scratch.md`, file lẻ ở root): Light tái dùng `ls -1 .okr/`, `scratch.md` ngoài whitelist 3 SOT → in `ℹ️ Có file/thư mục ở vị trí lạ: scratch.md`. Deep Bước 4: scratch.md không thuộc reachable set → báo mồ côi + gợi ý (tài liệu DÙNG → resources / cross-cutting → context/). ĐẠT.
- **Case B** (`.okr/research/data.md`, thư mục con lạ): Light thấy `research` ngoài whitelist thư mục → in `vị trí lạ: research`. Deep Bước 2: `research/` không trong tập thư mục đã biết → mọi file trong đó mồ côi → báo + gợi ý neo. ĐẠT.
- **Case C** (`.okr/context/glossary.md` đã có entry trong `context/index.md`): Light: `context` trong whitelist → KHÔNG cảnh báo. Deep Bước 2: file trong `context/` reachable theo vị trí; Bước 5: có entry trong index → không báo. Nếu glossary.md THIẾU entry → Bước 5 báo "context lệch". ĐẠT.

Nếu bất kỳ case nào không khớp văn bản trong SKILL → quay lại Task 5 sửa cho khớp.

- [ ] **Step 8: Verify toàn bộ không còn em-dash trong file sửa**

Run:
```bash
if rg -n "[\u{2013}\u{2014}]" \
  skills/okr-shared/SKILL.md \
  skills/okr-shared/references/preload.md \
  skills/okr-shared/references/sot-ownership.md \
  skills/okr-shared/references/schemas.md \
  skills/okr-harness/SKILL.md \
  skills/okr-harness/references/flows.md \
  skills/okr-analyze/SKILL.md \
  skills/okr-init/SKILL.md skills/okr-init/references/flow-update-resource.md skills/okr-init/references/data-format.md \
  skills/okr-plan/SKILL.md skills/okr-plan/references/task-format.md skills/okr-plan/references/flow-new.md skills/okr-plan/references/flow-update.md \
  skills/okr-track/SKILL.md skills/okr-track/references/data-format.md skills/okr-track/references/flow-shared.md skills/okr-track/references/flow-inbox.md skills/okr-track/references/flow-light.md \
  CLAUDE.md CHANGELOG.md; then
  echo "FAIL: con em-dash/en-dash, sua truoc khi xong"
else
  echo "PASS: khong con em-dash/en-dash"
fi
```
Expected: `PASS: khong con em-dash/en-dash`.

- [ ] **Step 9: Verify tham chiếu chết (link nội bộ)**

Run:
```bash
grep -rn "schemas.md\|preload.md\|sot-ownership.md" skills/okr-analyze skills/okr-init skills/okr-plan skills/okr-track skills/okr-harness | grep -i context
```
Đọc các dòng trỏ tới `preload.md` "Reachability khi ghi" / `schemas.md` "Context layer". Xác nhận section đích tồn tại đúng tên (Task 1 tạo "Reachability khi ghi", Task 3 tạo "Context layer"). Không có link trỏ tên sai.

- [ ] **Step 10: Commit (nếu Step nào ở trên phải sửa)**

Nếu các verify trên không cần sửa gì, bỏ qua. Nếu có sửa:
```bash
git add -A
git commit -m "fix(okr-shared): dong bo ban do neo + sua tham chieu sau verify (Dot 15)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Tự kiểm tra plan (đã chạy khi viết)

**1. Spec coverage** (mục 9 spec, 10 nhóm file):
- okr-shared SKILL + preload.md → Task 1. sot-ownership.md → Task 2. schemas.md → Task 3. okr-harness SKILL + flows → Task 4. okr-analyze → Task 5. okr-init (+data-format/flow) → Task 6. okr-plan (+task-format/flow-new/flow-update) → Task 7. okr-track (+data-format/flow-shared/flow-inbox/flow-light) → Task 8. CLAUDE.md + CHANGELOG.md → Task 9. Tiêu chí Done mục 12 → Task 10. Đủ.
- Lưu ý: spec mục 9 ghi okr-init sửa cả `references/data-format.md`. Section `## Tài liệu & Knowledge Base` đã có sẵn trong data-format.md đó (đọc xác nhận dòng 65, 83-88), nên Task 6 KHÔNG thêm schema trùng vào data-format.md, chỉ thêm 1 dòng cross-ref ranh giới (trỏ schemas "Context layer") để người sửa resources biết phân biệt với context/. Quy tắc đăng ký + confirm gate đặt ở SKILL + flow. Ranh giới canonical đặt ở schemas.md (Task 3) để tránh trùng.

**2. Placeholder scan:** Mọi step có nội dung markdown cụ thể để dán, lệnh grep cụ thể, expected cụ thể. Không có "TBD"/"tương tự task N"/"thêm xử lý phù hợp".

**3. Type/tên nhất quán:** "Reachability khi ghi" (preload canonical), "Context layer" (schemas), "Bản đồ neo", dòng `Path:`, "Áp dụng context", whitelist root (objective/resources/plan + actions/inbox/log/lessons/context) dùng đồng nhất qua các task. Mapping `thought → context` định nghĩa ở Task 3 (schemas) và dùng ở Task 8 (flow-inbox), khớp tên.

---

## Bàn giao thực thi (Execution Handoff)

**Plan hoàn tất, lưu tại `docs/superpowers/plans/2026-06-04-chong-file-mo-coi.md`. Hai lựa chọn thực thi:**

**1. Subagent-Driven (khuyến nghị)** - mỗi task một subagent mới, review giữa các task, lặp nhanh. REQUIRED SUB-SKILL: superpowers:subagent-driven-development.

**2. Inline Execution** - thực thi trong phiên này theo executing-plans, chạy theo lô với checkpoint để review. REQUIRED SUB-SKILL: superpowers:executing-plans.

Chọn cách nào?
