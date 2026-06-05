# Đợt 16: Đúc kết quy trình/skill mới ở runtime (retro 3 ngăn) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Mở `okr-retro` từ 2 ngăn bài học (skill / project) thành 3 ngăn theo định nghĩa **skill = công cụ (tool) / workflow = bản đồ điều phối / project = tri thức**, để khi chạy OKR thật mà phát sinh một QUY TRÌNH mới hoặc một NĂNG LỰC mới, retro đúc kết đúng ngăn, có đường thăng hạng từ project-local lên harness-shared, và không làm phình harness bằng rác.

**Architecture:** Harness skill-only, toàn bộ "code" là markdown prompt. Đây là thay đổi **mức A: lớp nhãn khái niệm**, KHÔNG tách vật lý 8 module, KHÔNG đụng mô hình inline, bảng SOT field-level, Preload Contract. Chỉ sửa text trong `okr-retro` (schema + flow) + cập nhật mô tả ở các nơi đếm "2 loại" (harness, sot-ownership, skill-contract, CLAUDE/CHANGELOG). Reachability (Đợt 15) tái dùng cơ chế neo: file `Wxxx` neo vào `lessons/index.md` như mọi bài học. Một ngoại lệ duy nhất: `preload.md` liệt kê thư mục con của `lessons/` theo tên (`skill/`, `project/`) ở "vùng cấu trúc đã biết" của audit, nên phải thêm `workflow/` vào đó, nếu không audit Deep sẽ flag nhầm `Wxxx` là mồ côi.

**Tech Stack:** Markdown prompt files trong `skills/`, Bash (`grep`, `rg`) để verify. KHÔNG có test runner tự động.

---

## Bối cảnh thực thi (đọc trước khi bắt đầu)

Đây KHÔNG phải codebase thông thường. Mỗi "skill" là một thư mục markdown trong `skills/`. Không biên dịch, không unit test. Vì vậy:

- **"Test" = grep + Read đối chiếu.** Mỗi task verify bằng `grep` (chuỗi canonical phải xuất hiện) và đọc lại để xác nhận khớp ngữ cảnh.
- **Nguyên tắc thứ tự (CLAUDE.md repo):** "Sửa canonical trước, bảng/skill link sau". Task 1 (`okr-retro/references/data-format.md`, canonical schema) chạy đầu; `SKILL.md` (Task 2) và các nơi mô tả (Task 3-4) trỏ về nó.
- **Cấm em-dash (—) và en-dash (–)** trong mọi file sửa. Dùng dấu phẩy, dấu hai chấm, hoặc tách câu. Plan này cũng tuân thủ.
- **Phạm vi: CHỈ `okr-retro` + các nơi đếm "2 loại".** KHÔNG đụng init/plan/track/analyze/capture/shared schema (trừ 1 dòng mô tả ở sot-ownership). KHÔNG tách thư mục module.
- **Định nghĩa lõi (nguồn gốc thiết kế, ghi để khỏi trôi):**
  - **Skill (công cụ)** trả lời "làm việc này NHƯ THẾ NÀO?" (năng lực nguyên tử).
  - **Workflow (bản đồ)** trả lời "KHI NÀO dùng công cụ nào, theo THỨ TỰ nào để đạt mục tiêu?" (điều phối).
  - **Project (tri thức)** là dữ kiện tĩnh của project (định nghĩa KR, capacity, đặc thù domain).

Commit convention (CLAUDE.md): prefix `feat|fix|refactor|docs|style`, scope chủ đạo `(okr-retro)`, các task khác `(okr-shared)|(okr-harness)|(docs)`. Mỗi task kết thúc bằng 1 commit.

## Quyết định thiết kế đã chốt (qua phiên grill, KHÔNG mở lại khi thực thi)

| # | Nhánh | Chốt |
| --- | --- | --- |
| 1 | Sân chơi | Runtime (cầu nối runtime → harness), KHÔNG phải dev-time |
| 2 | Mức áp dụng | Mức A: lớp nhãn. KHÔNG tách vật lý, KHÔNG đổi danh xưng toàn harness |
| 3 | Phân tuyến shared/local | Phép thử: "Copy harness sang project OKR khác, cái này còn dùng được không?" Còn = shared, phụ thuộc domain = local |
| 4 | 3 ngăn | A skill (luôn shared) · C workflow (lưỡng tính) · B project (luôn local) |
| 5 | Phép thử phân loại | "how?" = skill · "thứ tự/khi nào?" = workflow · "dữ kiện tĩnh, không làm theo bước được?" = project |
| 6 | Ngưỡng đúc kết (bất đối xứng theo chi phí-khi-sai) | improve = 1 lần · create-local = 1 lần · **create-shared = ≥3 lần thực tế HOẶC user chủ động yêu cầu** |
| 7 | Schema | Thêm đúng 2 field cứng: `mode` (improve\|create), `scope` (shared\|local). Mở rộng `type` enum thêm `workflow`. Số lần lặp ghi ở body `## Bằng chứng` |
| 8 | Tích luỹ xuyên phiên | **workflow-local là phòng chờ của workflow-shared**: mỗi lần gặp lại dùng dedup cập nhật mốc; đủ 3 mốc + qua phép thử tổng quát thì thăng `scope` local→shared (tái dùng dedup, không thêm cơ chế đếm) |

## File Structure (file đụng tới + trách nhiệm)

| File | Vai trò sau thay đổi | Task |
| --- | --- | --- |
| `skills/okr-retro/references/data-format.md` | Canonical schema: thêm ngăn `workflow/` + file `Wxxx`, 2 field `mode`/`scope`, mở rộng `type`, bảng status theo (type, scope), bảng index thứ 3 (Workflow), ràng buộc đóng | 1 |
| `skills/okr-retro/SKILL.md` | "2 loại" → "3 loại"; 2 phép thử (phân loại + phân tuyến); ngưỡng bất đối xứng; hành động "thăng scope" local→shared; báo cáo tách hàng đợi port | 2 |
| `skills/okr-shared/references/sot-ownership.md` | Cập nhật mô tả dòng `Bài học`: 3 ngăn (skill/workflow/project). Owner KHÔNG đổi (vẫn `okr-retro`) | 3 |
| `skills/okr-shared/references/preload.md` | Thêm `workflow/` vào vùng cấu trúc đã biết của audit reachability + danh sách on-demand; mô tả lesson theo target/area gồm cả workflow | 3 |
| `skills/okr-harness/SKILL.md` + `references/flows.md` + `references/skill-contract.md` | Mô tả 2 loại → 3 loại; flows phân loại A/B/C; output schema thêm `scope` + `lessons_promoted` + đổi `pending_skill_lessons` → `pending_port_lessons` | 3 |
| `CLAUDE.md` | Cây skill + mô tả okr-retro: 3 loại; mục lịch sử Đợt 16 | 4 |
| `CHANGELOG.md` | Một dòng Đợt 16 | 4 |

---

## Task 1: data-format.md - canonical schema 3 ngăn

**Files:**
- Modify: `skills/okr-retro/references/data-format.md` (hiện 94 dòng)

Đây là nền của cả đợt. `SKILL.md` và các nơi khác trỏ về đây.

- [ ] **Step 1: Mở rộng cây thư mục lessons**

Thay block cây thư mục (dưới heading `## Cây thư mục`, từ ```` ```` `.okr/lessons/` ... `project/` ... ```` ````) bằng:

````
.okr/lessons/
├── index.md              # Overview + link. AUTO-LOAD mỗi phiên OKR.
├── skill/                # Loại A: công cụ (tool) - năng lực harness, luôn shared
│   └── Sxxx-slug.md
├── workflow/             # Loại C: bản đồ điều phối - lưỡng tính shared/local
│   └── Wxxx-slug.md
└── project/              # Loại B: tri thức tĩnh về project, luôn local
    └── Pxxx-slug.md
````

Ngay dưới block, thay 2 dòng mô tả loại A/B hiện có bằng:

````markdown
- Loại A (`skill/`): công cụ ("how"). Năng lực nguyên tử của harness. Luôn `scope=shared`. Hàng đợi cải tiến, user port thủ công về repo gốc `objective-kit`.
- Loại C (`workflow/`): bản đồ ("khi nào/thứ tự nào"). **Lưỡng tính**: `shared` = điều phối tổng quát, port về harness thành flow; `local` = playbook riêng project, sống tại chỗ.
- Loại B (`project/`): tri thức tĩnh về project đích (dữ kiện, định nghĩa). Luôn `scope=local`, sống tại chỗ.
````

- [ ] **Step 2: Thay frontmatter template (thêm 2 field cứng + mở rộng type)**

Thay block YAML template (dưới `## File bài học (template unified)`) bằng:

````yaml
---
id: W001                 # S### skill · W### workflow · P### project (đếm riêng từng loại)
type: workflow           # skill | workflow | project
mode: create             # improve | create. CHỈ skill/workflow. project bỏ trống.
scope: local             # shared | local. workflow lưỡng tính; skill luôn shared; project luôn local.
status: active           # xem bảng "Status values"
date: "YYYY-MM-DD"        # ngày trích xuất
target:                  # skill/workflow: file cần sửa (mode=improve) HOẶC vị trí đề xuất (mode=create). project bỏ trống.
area:                    # CHỈ project: mảng tri thức (vd "định nghĩa KR", "capacity"). skill/workflow bỏ trống.
essence: "Câu lõi hành động 1 dòng. Đây là dòng đưa lên index."
---

## Bối cảnh
[Phiên này xảy ra gì dẫn tới bài học. 1-3 câu.]

## Bài học
[Lần sau làm khác đi thế nào. Rule cụ thể, actionable.]

## Bằng chứng
[Link action/log/file hoặc trích dẫn. mode=create + scope=shared: BẮT BUỘC liệt kê ≥3 mốc thực tế (mỗi mốc 1 dòng: ngày + nơi xảy ra). Đây là cái gác cổng minh bạch cho ngưỡng ≥3 lần. Các loại khác: 1 artifact hoặc "phiên YYYY-MM-DD".]
````

- [ ] **Step 3: Thêm bảng ràng buộc đóng (validate khi ghi)**

Ngay sau block template, thêm section:

````markdown
### Ràng buộc đóng (agent tự kiểm trước khi ghi)

| type | scope | mode | target | area |
| --- | --- | --- | --- | --- |
| `skill` | luôn `shared` | `improve` \| `create` | file cần sửa / vị trí đề xuất | trống |
| `workflow` | `shared` \| `local` (BẮT BUỘC chọn) | `improve` \| `create` | file cần sửa / vị trí đề xuất | trống |
| `project` | luôn `local` | trống | trống | mảng tri thức |

- `type=skill` mà `scope=local` là SAI: công cụ nguyên tử gần như luôn tổng quát. Nếu thật sự riêng project, nó là `workflow` local hoặc `project`, không phải skill.
- `mode=create` + chưa có file thật: `target` ghi chuỗi vị trí đề xuất, vd `"flow mới: okr-track/flow-weekly-sync"` hoặc `"skill mới: okr-xxx"`.
- `mode=create` + `scope=shared`: chỉ hợp lệ khi `## Bằng chứng` có ≥3 mốc, HOẶC bài sinh do user chủ động yêu cầu đưa vào harness (ghi rõ "user yêu cầu" ở Bằng chứng).
````

- [ ] **Step 4: Thay bảng "Status values"**

Thay bảng dưới `## Status values` bằng:

````markdown
| type | scope | status | Nghĩa |
| --- | --- | --- | --- |
| skill | shared | `pending` | Chưa port về repo gốc |
| skill | shared | `ported` | User đã port + cải tiến ở source |
| workflow | shared | `pending` | Bản đồ tổng quát, chờ port thành flow ở harness |
| workflow | shared | `ported` | Đã port về harness |
| workflow | local | `active` | Playbook riêng project, còn áp dụng |
| workflow | local | `obsolete` | Playbook không còn đúng |
| project | local | `active` | Tri thức còn đúng |
| project | local | `obsolete` | Tri thức hết đúng (objective đổi, đã khắc phục) |

> **Vòng đời theo scope, không theo type:** `shared` đi `pending → ported` (hàng đợi port). `local` đi `active → obsolete` (sống tại chỗ). Workflow là loại duy nhất có thể đổi scope: xem "Thăng scope" ở `SKILL.md`.
````

- [ ] **Step 5: Thêm bảng Workflow vào template index.md**

Trong block markdown index (dưới `## Index (`index.md`)`), GIỮA bảng loại A và bảng loại B (project), chèn:

````markdown
## Workflow (loại C)

| ID | Essence | Scope | Target/Vị trí | Status | Link |
|----|---------|-------|---------------|--------|------|
| W001 | Câu lõi 1 dòng | local | playbook tuần | active | [W001](workflow/W001-slug.md) |
````

Đồng thời cập nhật câu mô tả ngay dưới các bảng (chỗ giải thích cột `Target`/`Area`):

````markdown
- Cột `Target` (skill/workflow) = file skill cần sửa (improve) hoặc vị trí đề xuất (create). Cột `Area` (project) = mảng tri thức (tự do). Cột `Scope` (workflow) = shared | local.
- Ba bảng A/C/B chỉ liệt kê bài còn hiệu lực (`pending` / `active`). Bài `ported` / `obsolete` chuyển xuống "Lưu trữ".
````

- [ ] **Step 6: Cập nhật mục "Quy ước"**

Trong `## Quy ước`, thay dòng `- `id` tăng dần riêng từng loại (S001, S002... · P001, P002...)...` bằng:

````markdown
- `id` tăng dần riêng từng loại (`S001`... skill · `W001`... workflow · `P001`... project). Tìm max id trong thư mục tương ứng rồi +1.
````

- [ ] **Step 7: Verify**

```bash
grep -n "workflow/\|Wxxx\|mode:\|scope:\|Ràng buộc đóng\|Thăng scope\|lưỡng tính" skills/okr-retro/references/data-format.md
rg -n "[\x{2013}\x{2014}]" skills/okr-retro/references/data-format.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥6 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-retro/references/data-format.md
git commit -m "feat(okr-retro): schema 3 ngan lessons (skill/workflow/project) + 2 field mode-scope (Dot 16)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 2: SKILL.md okr-retro - 3 loại + phép thử + ngưỡng + thăng scope

**Files:**
- Modify: `skills/okr-retro/SKILL.md` (hiện 95 dòng)

- [ ] **Step 1: Thay bảng "Hai loại bài học" thành "Ba loại"**

Thay heading `## Hai loại bài học` thành `## Ba loại bài học` và thay bảng + dòng "Phân biệt nhanh" bằng:

````markdown
| Loại | Là gì | Phép thử nhận diện | Lưu ở | Scope | Vòng đời |
|------|-------|--------------------|-------|-------|----------|
| A - skill (công cụ) | Năng lực nguyên tử của harness: làm được việc gì mới | Trả lời **"how?"** | `lessons/skill/` | luôn `shared` | `pending` → port → `ported` |
| C - workflow (bản đồ) | Trình tự điều phối công cụ để đạt mục tiêu | Trả lời **"khi nào / thứ tự nào?"** | `lessons/workflow/` | **lưỡng tính** | shared: `pending`→`ported` · local: `active`→`obsolete` |
| B - project (tri thức) | Sự thật/định nghĩa tĩnh của project | Là **dữ kiện**, không làm theo bước được | `lessons/project/` | luôn `local` | `active` → `obsolete` |

### Hai phép thử khi quét phiên

1. **Phân loại (A/C/B)**: "how?" → skill · "thứ tự/khi nào dùng cái gì?" → workflow · "dữ kiện tĩnh, không thực hiện theo bước?" → project.
2. **Phân tuyến scope (chỉ workflow)**: hỏi **"Copy harness sang một project OKR khác, cái này còn dùng được không?"** Còn → `shared` (ứng viên port về harness). Phụ thuộc domain/objective cụ thể → `local` (playbook sống tại project).
````

- [ ] **Step 2: Thêm mục "Ngưỡng đúc kết" (chống phình harness)**

Ngay sau section ba loại, thêm:

````markdown
## Ngưỡng đúc kết (bất đối xứng theo chi phí-khi-sai)

Càng đụng vào harness chung, càng phải có bằng chứng lặp lại. Tiêu chí chất lượng cũ (tái dùng + không hiển nhiên + actionable) giữ nguyên cho cả ba loại; ngưỡng lặp-lại là lớp lọc CỘNG THÊM chỉ cho nhánh create-shared.

| Việc đúc kết | Chi phí nếu sai | Ngưỡng |
| --- | --- | --- |
| `mode=improve` cái có sẵn (skill/workflow shared) | Thấp (có file để sửa) | **1 lần** gặp, nếu rõ + actionable |
| `mode=create` + `scope=local` (playbook / tri thức riêng) | Thấp (sống tại `.okr/`) | **1 lần**, nếu tái dùng được |
| `mode=create` + `scope=shared` (port về harness) | **Cao** (mọi project gánh, khó gỡ) | **≥3 lần thực tế** (≥3 mốc ở `## Bằng chứng`), HOẶC user chủ động yêu cầu đưa vào harness |

### Phòng chờ: local là cửa vào của shared

Một workflow hiếm khi lặp 3 lần trong CÙNG một phiên. Bằng chứng ≥3 lần tích luỹ XUYÊN phiên qua dedup đã có (Bước 4):

1. Lần đầu gặp pattern chưa chắc tổng quát: ghi `workflow`, `scope=local`, `mode=create` (ngưỡng local = 1, hợp lệ ngay, rẻ). Bằng chứng: [mốc 1].
2. Phiên sau gặp lại: dedup, cập nhật ĐÚNG file đó, thêm [mốc 2], [mốc 3].
3. Đủ 3 mốc + qua phép thử "copy sang project khác vẫn đúng": ĐỀ XUẤT **thăng scope** local → shared (đổi `scope=shared`, `status` active → pending). Giờ là ứng viên port về harness.

Hệ quả có chủ đích: không ý tưởng nào nhảy thẳng vào "đề xuất sửa harness chung". Phải sống ở project, chứng minh qua 3 lần, mới được xét lên.
````

- [ ] **Step 3: Cập nhật Flow Bước 4 (phân loại) + Bước 5 (bảng ứng viên)**

Trong `### Bước 4`, thay câu mô tả bằng:

````markdown
Mỗi bài: chạy 2 phép thử (phân loại A/C/B + phân tuyến scope), gán `type` + `mode` + `scope`, soạn `essence` 1 dòng, gán `target` (skill/workflow) hoặc `area` (project). Áp ngưỡng đúc kết: bài `create-shared` chưa đủ 3 mốc thì hạ xuống `workflow local` (phòng chờ), KHÔNG ghi thẳng shared. Đối chiếu index: trùng/giao bài cũ → "Cập nhật Sxxx/Wxxx/Pxxx". Workflow-local đã đủ 3 mốc + tổng quát → đánh dấu "Thăng W0xx → shared".
````

Trong `### Bước 5`, thay bảng ứng viên mẫu bằng:

````markdown
```
| # | Loại | Mode | Scope | Essence | Target/Area | Mới/Cập nhật/Thăng |
|---|------|------|-------|---------|-------------|--------------------|
| 1 | A skill | improve | shared | ... | okr-track/flow-light.md | Mới |
| 2 | C workflow | create | local | ... | playbook tuần | Mới |
| 3 | C workflow | create | shared | ... | flow mới: okr-track/flow-weekly-sync | Thăng W002 → shared |
| 4 | B project | - | local | ... | capacity | Cập nhật P003 |
```
````

- [ ] **Step 4: Cập nhật Bước 6 (ghi) thêm hành động thăng scope**

Trong `### Bước 6`, sau bullet "Cập nhật", thêm bullet:

````markdown
- **Thăng scope** (chỉ workflow local → shared): sửa file `Wxxx` cùng chỗ, đổi `scope: local` → `shared`, `status: active` → `pending`. Chuyển dòng index từ cột scope `local`/`active` sang `shared`/`pending`. KHÔNG đổi `id`, KHÔNG tạo file mới.
````

Cập nhật bullet "Folder ... chưa có → tạo": thêm `workflow/` vào danh sách thư mục tạo:

````markdown
- Folder `.okr/lessons/` chưa có → tạo `index.md` + `skill/` + `workflow/` + `project/`.
````

- [ ] **Step 5: Cập nhật Bước 7 (báo cáo) tách hàng đợi port**

Trong `### Bước 7`, thay câu cuối (nhắc loại A pending) bằng:

````markdown
Tóm tắt: ghi mới mấy bài, cập nhật mấy, thăng scope mấy, đánh dấu lỗi thời mấy. **Hàng đợi port** = skill `pending` + workflow `shared pending`: nếu >0 → nhắc "X bài chờ port về repo gốc objective-kit (Y công cụ + Z workflow tổng quát)."
````

- [ ] **Step 6: Cập nhật phần mở đầu + Quy tắc + Test scenarios**

Dòng 8 mở đầu, "phân loại, ghi vào `.okr/lessons/`": giữ nguyên (vẫn đúng). Trong `## Quy tắc`, dòng `**Trùng thì cập nhật**`: thêm câu "Workflow local đủ chín thì thăng scope, không đẻ bản shared mới."

Trong `## Test scenarios`, thêm 1 scenario sau "Dedup":

````markdown
### Thăng workflow local lên shared
1. Phiên 1: user lập quy trình "thứ Hai sync action với lịch rồi mới track". Ghi W001 workflow/local/create, 1 mốc.
2. Phiên 2, 3: gặp lại cùng quy trình. Dedup cập nhật W001, thêm mốc 2, 3.
3. Phiên 3: đủ 3 mốc + phép thử "copy sang project khác vẫn đúng" = PASS (không phụ thuộc domain).
4. Đề xuất thăng W001 → shared. User đồng ý. Đổi scope=shared, status=pending.
5. Báo: "Thăng 1 workflow. 1 bài chờ port về repo gốc (workflow tổng quát)."
````

- [ ] **Step 7: Verify**

```bash
grep -n "Ba loại\|workflow\|phép thử\|Ngưỡng đúc kết\|Phòng chờ\|Thăng scope\|create-shared\|≥3" skills/okr-retro/SKILL.md
rg -n "[\x{2013}\x{2014}]" skills/okr-retro/SKILL.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥8 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 8: Commit**

```bash
git add skills/okr-retro/SKILL.md
git commit -m "feat(okr-retro): 3 loai + 2 phep thu + nguong bat doi xung + thang scope (Dot 16)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: Cập nhật các nơi đếm "2 loại" (harness + sot-ownership + skill-contract)

**Files:**
- Modify: `skills/okr-shared/references/sot-ownership.md` (dòng 19)
- Modify: `skills/okr-harness/SKILL.md` (dòng 180-185)
- Modify: `skills/okr-harness/references/flows.md` (dòng 133, 136, 137, 140)
- Modify: `skills/okr-harness/references/skill-contract.md` (Output schema dòng 110-117)
- Modify: `skills/okr-shared/references/preload.md` (dòng 52, 57, 87)

- [ ] **Step 1: sot-ownership.md - mô tả 3 ngăn**

Thay dòng 19 (`| Bài học (.okr/lessons/**...) | okr-retro |`) bằng:

````markdown
| Bài học (`.okr/lessons/**`: 3 ngăn skill/workflow/project, tạo/sửa/thăng scope/đánh dấu obsolete, ported) | `okr-retro` |
````

(Owner KHÔNG đổi. Chỉ làm rõ phạm vi 3 ngăn.)

- [ ] **Step 2: okr-harness SKILL.md - 2 loại → 3 loại**

Thay đoạn dòng 180-183 (`okr-retro ghi 2 loại bài học...` + 2 bullet A/B) bằng:

````markdown
`okr-retro` ghi 3 loại bài học vào `.okr/lessons/` (skill = công cụ, workflow = bản đồ điều phối, project = tri thức):

- **Loại A (skill/công cụ)**: năng lực harness, luôn shared. Hàng đợi port thủ công về repo gốc `objective-kit`.
- **Loại C (workflow/bản đồ)**: lưỡng tính. Shared = quy trình tổng quát chờ port; local = playbook riêng project, sống tại chỗ.
- **Loại B (project/tri thức)**: dữ kiện tại chỗ, auto-load mỗi phiên qua index.
````

- [ ] **Step 3: flows.md - phân loại A/B/C + nhắc port**

Dòng 133 (`→ okr-retro: phân loại A/B + soạn essence + dedup`): đổi `A/B` thành `A/C/B (skill/workflow/project) + scope`.

Dòng 136 (`Ghi file lessons/{skill,project}/ + cập nhật index.md`): đổi `{skill,project}` thành `{skill,workflow,project}`. (Residue 2 ngăn trong mermaid, grep `"workflow"` ở Step 5 KHÔNG bắt được dòng này.)

Dòng 137 (`Báo: ghi mới/cập nhật/lỗi thời + nhắc port loại A`): đổi `nhắc port loại A` thành `nhắc port (skill + workflow shared)`.

Dòng 140 (`Loại A là hàng đợi port thủ công...`): đổi thành `Skill và workflow shared là hàng đợi port thủ công về repo gốc.`

- [ ] **Step 4: skill-contract.md - Output schema**

Thay block YAML Output (dòng 109-118) bằng:

````yaml
status: success | nothing       # nothing = không có bài học đáng ghi
lessons_added:                  # bài mới
  - id: string                  # Sxxx | Wxxx | Pxxx
    type: skill | workflow | project
    scope: shared | local
    essence: string
lessons_updated: [Sxxx, Wxxx, Pxxx]   # bài cập nhật (dedup)
lessons_promoted: [Wxxx]        # workflow local → shared (thăng scope)
lessons_obsoleted: [Pxxx, Wxxx]       # bài đánh dấu lỗi thời
pending_port_lessons: number    # skill pending + workflow shared pending, nhắc port về repo gốc
````

- [ ] **Step 5: preload.md - thêm `workflow/` vào reachability + on-demand**

Lý do: audit reachability (Đợt 15) liệt kê thư mục con của `lessons/` theo TÊN ở "vùng cấu trúc đã biết". Thiếu `workflow/` → audit Deep flag nhầm file `Wxxx` là mồ côi.

Dòng 87 (`- Thư mục: ... lessons/ (gồm index.md, skill/, project/), context/ ...`): thêm `workflow/` vào ngoặc, thành `lessons/ (gồm index.md, skill/, workflow/, project/)`.

Dòng 52 (`lessons/skill/*, lessons/project/* (file detail) | Đọc khi cần detail...`): thêm `workflow/*`, thành `lessons/skill/*, lessons/workflow/*, lessons/project/*`.

Dòng 57 (`...(loại B theo area, loại A theo target)...`): mở rộng thành `(loại B theo area; loại A và workflow theo target)`.

- [ ] **Step 6: Verify**

```bash
grep -n "3 ngăn\|3 loại\|workflow\|lessons_promoted\|pending_port_lessons\|A/C/B" skills/okr-shared/references/sot-ownership.md skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md skills/okr-harness/references/skill-contract.md skills/okr-shared/references/preload.md
grep -rn "{skill,project}\|2 loại\|hai loại" skills/ && echo "FAIL: con residue 2 ngan" || echo "OK: khong con residue 2 ngan"
rg -n "[\x{2013}\x{2014}]" skills/okr-shared/references/sot-ownership.md skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md skills/okr-harness/references/skill-contract.md skills/okr-shared/references/preload.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥6 match dòng đầu; `OK: khong con residue 2 ngan`; `OK: khong co em-dash/en-dash`.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-shared/references/sot-ownership.md skills/okr-harness/SKILL.md skills/okr-harness/references/flows.md skills/okr-harness/references/skill-contract.md skills/okr-shared/references/preload.md
git commit -m "feat(okr-harness): mo ta 3 loai retro + output schema scope/promoted + reachability workflow (Dot 16)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: CLAUDE.md + CHANGELOG.md - lịch sử Đợt 16

**Files:**
- Modify: `CLAUDE.md` (dòng 29, 45 mô tả okr-retro; mục Lịch sử phát triển)
- Modify: `CHANGELOG.md`

- [ ] **Step 1: CLAUDE.md - cây skill + bảng vai trò**

Dòng 29 (`okr-retro/ ← rút bài học từ phiên, ghi .okr/lessons/ (record-only)`): đổi thành:

````markdown
├── okr-retro/                  ← rút bài học 3 ngăn (skill/workflow/project), ghi .okr/lessons/ (record-only)
````

Dòng 45 (bảng Skill → Vai trò, okr-retro): đổi mô tả thành:

````markdown
| okr-retro   | Rút bài học từ phiên, ghi `.okr/lessons/` (3 ngăn: skill/workflow/project). Record-only, confirm trước ghi. |
````

Dòng 105 (cây runtime, lessons): đổi thành:

````markdown
└── lessons/              # Bài học (okr-retro): index.md + skill/ + workflow/ + project/
````

- [ ] **Step 2: CLAUDE.md - mục Lịch sử phát triển**

Sau mục Đợt 15 (dòng 167), thêm:

````markdown
13. Đợt 16: Đúc kết quy trình/skill mới ở runtime (retro 3 ngăn). Áp định nghĩa skill = công cụ ("how") / workflow = bản đồ điều phối ("khi nào/thứ tự") / project = tri thức tĩnh. Mức A (lớp nhãn, KHÔNG tách vật lý). `okr-retro` từ 2 ngăn thành 3: thêm `lessons/workflow/` (Wxxx) + 2 field cứng `mode` (improve/create) + `scope` (shared/local), mở rộng `type`. Hai phép thử: phân loại (how/thứ tự/dữ kiện) + phân tuyến scope ("copy sang project khác còn dùng?"). Ngưỡng bất đối xứng chống phình harness: improve = 1, create-local = 1, create-shared ≥3 mốc (hoặc user chủ động). Workflow-local là phòng chờ của shared: đủ 3 mốc + tổng quát thì thăng scope. Cập nhật output schema harness (scope, lessons_promoted, pending_port_lessons). Chỉ `.okr/` runtime.
````

- [ ] **Step 3: CHANGELOG.md - một dòng**

Thêm dòng mới ở ĐẦU bảng dữ liệu, ngay dưới hàng separator của header và TRÊN dòng Đợt 15 (hiện là dòng mới nhất, 2026-06-04). Đợt 16 là 2026-06-05 nên phải đứng trên Đợt 15:

````markdown
| 2026-06-05 | Đợt 16: retro 3 ngăn. Áp định nghĩa skill=công cụ / workflow=bản đồ / project=tri thức. `okr-retro` thêm ngăn `lessons/workflow/` (Wxxx) + 2 field `mode`/`scope`, mở rộng `type`. 2 phép thử (phân loại how/thứ tự/dữ kiện + phân tuyến scope "copy sang project khác còn dùng?"). Ngưỡng bất đối xứng: improve=1, create-local=1, create-shared≥3 mốc. Workflow-local là phòng chờ của shared (đủ 3 mốc + tổng quát → thăng scope). Output schema harness +scope/+lessons_promoted, đổi pending_skill→pending_port. | okr-retro (data-format + SKILL), okr-shared (sot-ownership), okr-harness (SKILL+flows+skill-contract), CLAUDE.md | Vá gap "quy trình/năng lực mới phát sinh ở runtime không có nhà đúc kết": phân biệt rõ công cụ vs bản đồ, có đường thăng hạng local→shared, chặn rác chảy vào harness chung |
````

- [ ] **Step 4: Verify**

```bash
grep -n "Đợt 16\|3 ngăn\|workflow/" CLAUDE.md CHANGELOG.md
rg -n "[\x{2013}\x{2014}]" CLAUDE.md CHANGELOG.md && echo "FAIL: con em-dash/en-dash" || echo "OK: khong co em-dash/en-dash"
```
Expected: ≥4 match dòng đầu; `OK: khong co em-dash/en-dash`.

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md CHANGELOG.md
git commit -m "docs: lich su Dot 16 retro 3 ngan (CLAUDE.md, CHANGELOG.md)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Done khi (acceptance toàn đợt)

1. `lessons/` có 3 ngăn: `skill/` (Sxxx), `workflow/` (Wxxx), `project/` (Pxxx). Schema canonical ở `okr-retro/references/data-format.md`.
2. Frontmatter có đúng 2 field cứng mới (`mode`, `scope`); `type` enum gồm `workflow`. Ràng buộc đóng có bảng tự kiểm.
3. `okr-retro/SKILL.md` có: bảng 3 loại, 2 phép thử (phân loại + phân tuyến), bảng ngưỡng bất đối xứng, mô hình phòng chờ, hành động thăng scope, báo cáo tách hàng đợi port.
4. Mọi nơi từng nói retro "2 loại" (harness SKILL/flows gồm cả dòng mermaid `{skill,project}`/skill-contract, sot-ownership, CLAUDE, CHANGELOG) đã thành "3 loại" + output schema có `scope`/`lessons_promoted`/`pending_port_lessons`. `grep -rn "{skill,project}\|2 loại\|hai loại" skills/` trả rỗng.
5. `preload.md` liệt kê `workflow/` ở vùng cấu trúc đã biết (dòng 87) + on-demand (dòng 52): audit reachability Đợt 15 KHÔNG flag nhầm file `Wxxx` là mồ côi.
6. KHÔNG đụng init/plan/track/analyze/capture schema (trừ 1 dòng mô tả sot-ownership + thêm `workflow/` ở preload.md). KHÔNG tách thư mục module. Mô hình inline + Preload Contract nguyên vẹn.
7. KHÔNG còn em-dash/en-dash ở mọi file sửa.

## Ngoài phạm vi (ghi để khỏi trôi)

- Đổi danh xưng "skill/flow" → "skill(tool)/workflow" toàn harness (mức B): KHÔNG làm đợt này.
- Tách vật lý 8 module thành tool/ + workflow/ (mức C): KHÔNG làm, chỉ đáng khi mở harness đa-objective/đa-user.
- Tự động port workflow shared về repo gốc: vẫn thủ công (record-only), đúng tinh thần `okr-retro` hiện tại.
</content>
</invoke>
