# okr-retro mode `consolidate` Implementation Plan (Đợt 17)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Thêm mode `consolidate` cho skill `okr-retro`: một lượt quét CHỦ ĐỘNG toàn kho `lessons/`, đề xuất 4 thao tác dọn dẹp (gộp, tách, reclassify, giải mâu thuẫn) cho user duyệt, cộng cơ chế nhắc theo ngưỡng.

**Architecture:** Harness skill-only, toàn bộ "code" là markdown prompt. Consolidate KHÔNG thêm thao tác ghi mới: mọi việc phân rã về 4 primitive ghi đã có (tạo file, cập nhật file, đánh `obsolete` + chuyển Lưu trữ, thăng scope local→shared). Cái mới duy nhất là lớp ĐỌC + PHÁT HIỆN + ĐỀ XUẤT toàn kho, tách riêng vào `flow-consolidate.md` (load on-demand). Record-only, confirm từng dòng trước ghi, không xoá file.

**Tech Stack:** Markdown prompt files dưới `skills/`. Không có test runner. "Verify" = grep (chống em-dash/en-dash) + đọc lại đối chiếu consistency cross-file. Mỗi task commit riêng.

---

## Bối cảnh cho engineer (đọc trước khi làm)

Repo này là **source code của một harness OKR**, mọi skill là thư mục markdown trong `skills/`. Bạn KHÔNG chạy OKR ở đây, bạn SỬA prompt.

- Skill liên quan: `skills/okr-retro/` (SOT của `.okr/lessons/**`). Gồm `SKILL.md` + `references/data-format.md`. Task này thêm `references/flow-consolidate.md`.
- Pattern tham chiếu: `skills/okr-track/references/flow-*.md` (cách 1 SKILL.md gọn trỏ sang nhiều `flow-*.md` load on-demand). Đọc `skills/okr-track/references/flow-inbox.md` để bắt giọng văn + cách trình bảng + dòng confirm gate.
- **Ba ngăn lessons**: `skill/` (Sxxx, loại A công cụ, luôn shared), `workflow/` (Wxxx, loại C bản đồ, lưỡng tính shared/local), `project/` (Pxxx, loại B tri thức, luôn local). Định nghĩa đầy đủ ở `skills/okr-retro/SKILL.md` mục "Ba loại bài học".
- **Quy tắc giao tiếp repo (CLAUDE.md)**: viết tiếng Việt đủ dấu, câu ngắn. **CẤM TUYỆT ĐỐI em-dash `—` và en-dash `–`** ở mọi vị trí kể cả tiêu đề. Dùng dấu phẩy, hai chấm, hoặc tách 2 câu. Cấm `---` 3 gạch ngang trong nội dung (ngoài frontmatter YAML và header phân cách Markdown đã có sẵn).
- **Hai phép thử Đợt 16** (consolidate tái dùng): (1) Phân loại A/C/B: "how?" → skill · "thứ tự/khi nào?" → workflow · "dữ kiện tĩnh?" → project. (2) Phân tuyến scope (chỉ workflow): "Copy harness sang project OKR khác còn dùng được không?" Còn → shared, phụ thuộc domain → local.

### Quy ước verify cho repo prompt-only

Không có pytest. Mỗi task "test" bằng:
- **Grep chống dash**: `grep -nP '[\x{2014}\x{2013}]' <file>` phải trả về RỖNG (exit code 1, không match). Ký tự `\x{2014}`=em-dash, `\x{2013}`=en-dash.
- **Đọc lại đối chiếu**: mở file vừa sửa, xác nhận nội dung yêu cầu có mặt + nhất quán với file khác (id, tên flow, tên field giống nhau mọi nơi).

---

## File Structure

| File | Trách nhiệm | Thay đổi |
|------|-------------|----------|
| `skills/okr-retro/references/flow-consolidate.md` | Toàn bộ flow C1-C6 + bảng phân rã primitive + hằng số ngưỡng + test scenarios | **Tạo mới** |
| `skills/okr-retro/SKILL.md` | Entry point: frontmatter description thêm trigger; mục "Khi nào chạy" thêm consolidate; phân biệt retro-thường vs consolidate + trỏ sang flow-consolidate.md; Bước 7 thêm nhắc ngưỡng | Sửa |
| `skills/okr-retro/references/data-format.md` | Thêm mục "Quy ước consolidate" (note obsolete trỏ-về) + mở rộng bảng Status values cho `obsolete` áp mọi ngăn khi consolidate | Sửa |
| `skills/okr-harness/references/flows.md` | Section 7 thêm route intent "dọn/rà soát kho lessons" → okr-retro consolidate | Sửa |
| `skills/okr-shared/references/sot-ownership.md` | 1 dòng làm rõ consolidate vẫn là okr-retro ghi | Sửa |
| `CLAUDE.md` | Thêm mục Đợt 17 vào "Lịch sử phát triển" | Sửa |
| `CHANGELOG.md` | Thêm 1 hàng Đợt 17 | Sửa |

Không đụng `okr-analyze` (reachability audit Đợt 15 tự lo: bài mới neo index, bài obsolete neo Lưu trữ). Không đụng schema frontmatter, không thêm status enum value mới.

---

## Task 1: Tạo `flow-consolidate.md`

**Files:**
- Create: `skills/okr-retro/references/flow-consolidate.md`

- [ ] **Step 1: Tạo file với toàn bộ nội dung flow**

Tạo `skills/okr-retro/references/flow-consolidate.md` với chính xác nội dung sau:

````markdown
# Flow: Consolidate (dọn kho lessons)

Quét CHỦ ĐỘNG toàn kho `lessons/`, đề xuất 4 thao tác dọn dẹp cho user duyệt. Phân biệt rõ với flow trích-bài:

- **retro-thường** (SKILL.md Flow Bước 1-7): quét PHIÊN hiện tại, trích bài MỚI.
- **consolidate** (flow này): quét KHO đã có, không quan tâm phiên.

Chạy khi: user gõ trigger ("dọn kho lessons", "consolidate", "sắp xếp lại bài học", "gộp bài học", "rà soát lessons") HOẶC user đồng ý lời nhắc ngưỡng cuối retro-thường (SKILL.md Bước 7).

## Hằng số (documented, tunable)

| Hằng số | Mặc định | Nghĩa |
|---------|----------|-------|
| `NGUONG_TONG` | 12 | Tổng bài còn hiệu lực (`pending`+`active`) vượt mức này thì retro-thường nhắc consolidate. |
| `NGUONG_NGAN` | 8 | Một ngăn bất kỳ vượt mức này thì retro-thường nhắc consolidate. |
| `GUARD_NHO` | 6 | Tổng bài còn hiệu lực < mức này thì kho còn nhỏ, dừng không quét. |

## Nguyên tắc (kế thừa retro-thường)

- Record-only + confirm TỪNG DÒNG trước ghi (bảng C4).
- KHÔNG xoá file. Bài bị gộp / thua mâu thuẫn / chuyển ngăn thành `status: obsolete` + thêm 1 dòng body trỏ về bài sống sót. Giữ trace.
- Chỉ `okr-retro` ghi `lessons/**`. SOT nguyên vẹn.
- **Zero schema mới, zero status enum value mới.** Bốn thao tác phân rã hết về 4 primitive ghi đã có (xem mục "Mô hình ghi"). Note "Đã gộp vào / Bị thay bởi / Chuyển ngăn" chỉ là 1 dòng text trong body bài `obsolete`, không phải field.
- Consolidate CHỈ làm việc trên bài còn hiệu lực (`pending`/`active`). KHÔNG đụng bài đã ở Lưu trữ (`ported`/`obsolete`).

## C1: Nạp kho

Read `lessons/index.md` + body MỌI bài còn hiệu lực (`pending`/`active`) trong cả 3 ngăn `skill/`, `workflow/`, `project/`. Bỏ qua bài đã ở Lưu trữ (`ported`/`obsolete`).

**Guard kho nhỏ**: đếm tổng bài còn hiệu lực. Nếu tổng < `GUARD_NHO` (6) thì báo 1 dòng "Kho lessons còn nhỏ (N bài), chưa cần dọn." rồi DỪNG, không quét tiếp (tránh tốn token vô ích).

## C2: Gom cụm (Hướng A: gom trước, xử lý sau)

Nhóm bài theo khóa:

- skill / workflow: khóa `(type, target)`.
- project: khóa `(type, area)`.
- Cộng: gom theo chủ đề khi `essence` cùng việc dù khác `target`/`area`.

Bài lạc cụm (không gom được với ai) vẫn xét riêng ở C3 cho thao tác per-item (tách, reclassify).

## C3: Xét 4 thao tác

**Trong cụm ≥2 bài:**

- **Gộp**: 2+ bài trùng hoặc giao nhiều (vd giao ≥80% nội dung) cùng target/area → 1 bài tổng quát hơn.
- **Mâu thuẫn**: 2 bài khuyên NGƯỢC nhau về cùng target → giữ 1, theo thứ tự ưu tiên: bài mới hơn (`date`) + nhiều bằng chứng hơn (`## Bằng chứng`).

**Mỗi bài (kể cả đứng riêng, lạc cụm):**

- **Tách**: 1 bài ôm ≥2 rule thuộc ≥2 target/area khác nhau → tách thành N bài, mỗi bài 1 rule.
- **Reclassify**: chạy lại 2 phép thử Đợt 16 (xem SKILL.md "Hai phép thử"):
  - Sai ngăn (phép thử phân loại A/C/B cho kết quả khác ngăn hiện tại) → chuyển ngăn.
  - Workflow-local đủ 3 mốc ở `## Bằng chứng` + qua phép thử phân tuyến scope ("copy sang project khác vẫn đúng") mà chưa thăng → đề xuất thăng scope.

**Cuối C3**: đề xuất reorder index cho gọn (gom dòng cùng target/area cạnh nhau trong mỗi bảng). Reorder chỉ sắp lại thứ tự dòng, không đụng file body.

## C4: Trình bảng đề xuất

Hiển thị bảng, user tick TỪNG DÒNG. KHÔNG ghi trước khi user duyệt.

```
| # | Thao tác    | Bài liên quan     | Kết quả                          | Lý do                            |
|---|-------------|-------------------|----------------------------------|----------------------------------|
| 1 | Gộp         | W003 + W005       | → W003 tổng quát, W005 obsolete  | cùng target flow-light, giao 80% |
| 2 | Tách        | P002              | → P002 + P00x mới                | ôm 2 area: capacity + KR         |
| 3 | Reclassify  | P004 → W00x       | tạo W00x, P004 obsolete          | trả lời "thứ tự nào" = workflow  |
| 4 | Mâu thuẫn   | S002 vs S006      | giữ S006, S002 obsolete          | S006 mới hơn + 3 bằng chứng      |
| 5 | Thăng scope | W001 local→shared | scope=shared, status=pending     | đủ 3 mốc, qua phép thử           |
```

User trả lời dạng tick (vd "1,3,5 giữ; 2 cắt; 4 sửa giữ S002"). Dòng nào user cắt thì bỏ, không ghi.

## C5: Ghi

Áp primitive tương ứng cho mỗi dòng user GIỮ (xem mục "Mô hình ghi"). Sau đó cập nhật `index.md`: 3 bảng chính (skill / workflow / project) + bảng Lưu trữ. Bài thành `obsolete` chuyển từ bảng chính xuống Lưu trữ. Bài mới (tách / reclassify) thêm dòng vào bảng chính đúng ngăn. Áp reorder nếu user giữ.

## C6: Báo cáo

Tóm tắt số đếm từng thao tác:

```
Consolidate xong:
  - Gộp: X cụm (Y bài → obsolete)
  - Tách: Z bài → thêm W bài mới
  - Reclassify: V bài chuyển ngăn
  - Mâu thuẫn: U cặp (U bài thua → obsolete)
  - Thăng scope: T workflow local→shared
Kho: N → M bài còn hiệu lực.
```

Nếu sinh thêm bài `shared pending` (do reclassify sang skill/workflow-shared hoặc thăng scope): cập nhật nhắc hàng đợi port "+K bài chờ port về repo gốc objective-kit."

## Mô hình ghi: phân rã từng thao tác về primitive

| Thao tác | Phân rã |
|----------|---------|
| **Gộp** (N→1) | Cập nhật bài sống sót (hợp `essence` + gộp `## Bằng chứng` + bối cảnh). Các bài còn lại: `status: obsolete` + 1 dòng body "Đã gộp vào Xxxx". Index: bài sống sót giữ bảng chính, bài bị gộp xuống Lưu trữ. |
| **Tách** (1→N) | Bài gốc: sửa `essence` còn 1 rule. Mỗi rule tách ra: tạo file mới (`id` mới trong ngăn, max id +1), neo index bảng chính. |
| **Reclassify** (sai ngăn) | Tạo file ở ngăn đúng (`id` mới theo prefix S/W/P ngăn đích), copy nội dung + chỉnh field (`type`/`scope`/`target`/`area`) cho khớp Ràng buộc đóng (data-format.md). Bài cũ `status: obsolete` + 1 dòng body "Chuyển ngăn thành Xxxx". |
| **Reclassify** (chỉ thăng scope) | Sửa TẠI CHỖ: `scope: local→shared`, `status: active→pending`. KHÔNG đổi `id`, KHÔNG đẻ file mới. (Đúng cơ chế Đợt 16.) |
| **Mâu thuẫn** | Bài thắng: giữ / tinh chỉnh. Bài thua: `status: obsolete` + 1 dòng body "Bị thay bởi Xxxx (mâu thuẫn, [lý do])". |
| **Reorder index** | Chỉ sắp lại thứ tự dòng trong 3 bảng chính, không đụng file body. |

Tất cả chỉ là tổ hợp tạo / cập nhật / obsolete / thăng-scope. Bài skill hoặc workflow-shared khi bị gộp / thua / chuyển ngăn cũng nhận `status: obsolete` (xem data-format.md "Status values": obsolete áp mọi ngăn khi consolidate).

## Reachability (Đợt 15) tự lo

Bài mới (tách / gộp / reclassify) neo `index.md` như mọi bài. Bài `obsolete` neo ở bảng "Lưu trữ". Không sinh file mồ côi, không cần sửa audit `okr-analyze`.

## Test scenarios

### Gộp nhóm
1. Kho có W003, W005 cùng target `okr-track/flow-light.md`, essence giao 80%.
2. User gõ "dọn kho lessons". C2 gom 2 bài vào 1 cụm.
3. C3 đề xuất gộp → W003 tổng quát, W005 obsolete.
4. User duyệt. Cập nhật W003 (hợp bằng chứng), W005 `obsolete` + "Đã gộp vào W003", index chuyển W005 xuống Lưu trữ.

### Tách
1. P002 ôm 2 rule: capacity + định nghĩa KR.
2. C3 đề xuất tách → P002 (giữ capacity) + P00x mới (định nghĩa KR).
3. User duyệt. Sửa P002 essence, tạo P00x, neo index.

### Reclassify sai ngăn
1. P004 thực ra mô tả "thứ tự nào" (workflow), không phải dữ kiện tĩnh.
2. C3 chạy lại 2 phép thử, phát hiện sai ngăn → đề xuất chuyển P004 thành W00x.
3. User duyệt. Tạo W00x (type=workflow, scope theo phép thử phân tuyến), P004 `obsolete` + "Chuyển ngăn thành W00x".

### Giải mâu thuẫn
1. S002 và S006 khuyên ngược nhau về cùng target.
2. C3 phát hiện mâu thuẫn → đề xuất giữ S006 (mới hơn + 3 bằng chứng), S002 obsolete.
3. User duyệt. S002 `obsolete` + "Bị thay bởi S006 (mâu thuẫn, S006 mới hơn + nhiều bằng chứng)".

### Nhắc ngưỡng (kích hoạt từ retro-thường)
1. User chạy retro-thường, ghi xong tổng kho thành 14 bài (workflow 9 > `NGUONG_NGAN`).
2. SKILL.md Bước 7 báo cáo thêm 1 dòng: "Kho lessons đã 14 bài (workflow 9). Cân nhắc chạy consolidate?"
3. User đồng ý → chạy flow này. Không đồng ý → dừng, không ghi gì thêm.

### Guard kho nhỏ
1. Kho mới có 4 bài. User gõ "consolidate".
2. C1 thấy tổng < `GUARD_NHO` (6) → báo "Kho lessons còn nhỏ (4 bài), chưa cần dọn." Dừng, không quét.
````

- [ ] **Step 2: Verify không có em-dash/en-dash**

Run: `grep -nP '[\x{2014}\x{2013}]' skills/okr-retro/references/flow-consolidate.md`
Expected: KHÔNG output (exit 1). Nếu match dòng nào, sửa ký tự `—`/`–` thành dấu phẩy / hai chấm / tách câu.

- [ ] **Step 3: Verify nội dung đủ 6 bước + 4 thao tác**

Run: `grep -nE '^## C[1-6]:|^### (Gộp|Tách|Reclassify|Giải mâu thuẫn|Nhắc ngưỡng|Guard kho nhỏ)' skills/okr-retro/references/flow-consolidate.md`
Expected: thấy đủ C1 đến C6 + đủ 6 test scenario heading.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-retro/references/flow-consolidate.md
git commit -m "feat(okr-retro): them flow-consolidate.md (C1-C6 + primitive + nguong) (Dot 17)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 2: Cập nhật `okr-retro/SKILL.md`

**Files:**
- Modify: `skills/okr-retro/SKILL.md`

- [ ] **Step 1: Mở rộng frontmatter description (thêm trigger consolidate)**

Tìm dòng:

```
description: "Rút bài học từ phiên OKR, lưu vào layer lessons. Record-only."
```

Thay bằng:

```
description: "Rút bài học từ phiên OKR, lưu vào layer lessons. Record-only. Cũng dọn kho lessons khi user gọi: dọn kho lessons, consolidate, sắp xếp lại bài học, gộp bài học, rà soát lessons."
```

- [ ] **Step 2: Thêm trigger consolidate vào mục "Khi nào chạy"**

Tìm block:

```
## Khi nào chạy

- **User chủ động**: "rút bài học", "tổng kết phiên", "retro".
- **Harness gợi ý cuối flow** (1 dòng), user đồng ý mới chạy. KHÔNG tự chạy ngầm.
```

Thay bằng:

```
## Khi nào chạy

- **User chủ động (retro-thường)**: "rút bài học", "tổng kết phiên", "retro".
- **User chủ động (consolidate)**: "dọn kho lessons", "consolidate", "sắp xếp lại bài học", "gộp bài học", "rà soát lessons". Chạy mode `consolidate`, xem dưới.
- **Harness gợi ý cuối flow** (1 dòng), user đồng ý mới chạy. KHÔNG tự chạy ngầm.

### Hai mode

- **retro-thường** (mặc định, Flow Bước 1-7 dưới): quét PHIÊN hiện tại, trích bài MỚI.
- **consolidate**: quét toàn KHO `lessons/` đã có (không quan tâm phiên), đề xuất gộp / tách / reclassify / giải mâu thuẫn. Flow đầy đủ ở `references/flow-consolidate.md`. Đọc file đó khi trigger consolidate kích hoạt.
```

- [ ] **Step 3: Thêm nhắc ngưỡng vào Bước 7 (Báo cáo)**

Tìm block:

```
### Bước 7: Báo cáo
Tóm tắt: ghi mới mấy bài, cập nhật mấy, thăng scope mấy, đánh dấu lỗi thời mấy. **Hàng đợi port** = skill `pending` + workflow `shared pending`: nếu >0 → nhắc "X bài chờ port về repo gốc objective-kit (Y công cụ + Z workflow tổng quát)."
```

Thay bằng:

```
### Bước 7: Báo cáo
Tóm tắt: ghi mới mấy bài, cập nhật mấy, thăng scope mấy, đánh dấu lỗi thời mấy. **Hàng đợi port** = skill `pending` + workflow `shared pending`: nếu >0 → nhắc "X bài chờ port về repo gốc objective-kit (Y công cụ + Z workflow tổng quát)."

**Nhắc ngưỡng consolidate** (1 dòng, confirm-gated): SAU khi ghi xong, đếm bài còn hiệu lực (`pending`+`active`). Nếu **tổng > `NGUONG_TONG`** (mặc định 12) HOẶC **một ngăn bất kỳ > `NGUONG_NGAN`** (mặc định 8) → nhắc 1 dòng, vd: "Kho lessons đã 14 bài (workflow 9). Cân nhắc chạy `consolidate` để gộp/dọn?" User đồng ý mới chạy consolidate. KHÔNG tự chạy. Hằng số định nghĩa ở `references/flow-consolidate.md`. Chấp nhận có thể nhắc lại phiên sau (nhẹ, user bỏ qua được), không ghi nhớ "đã nhắc".
```

- [ ] **Step 4: Thêm dòng trỏ flow-consolidate vào mục "Quy tắc"**

Tìm block:

```
- **SOT**: chỉ `okr-retro` ghi `.okr/lessons/**`. Xem `okr-shared/references/sot-ownership.md`.
- **Schema**: `references/data-format.md`.
```

Thay bằng:

```
- **SOT**: chỉ `okr-retro` ghi `.okr/lessons/**`. Xem `okr-shared/references/sot-ownership.md`.
- **Schema**: `references/data-format.md`.
- **Consolidate**: dọn kho toàn diện ở `references/flow-consolidate.md`. Cùng record-only + confirm từng dòng, zero schema mới.
```

- [ ] **Step 5: Verify không có em-dash/en-dash**

Run: `grep -nP '[\x{2014}\x{2013}]' skills/okr-retro/SKILL.md`
Expected: KHÔNG output (exit 1).

- [ ] **Step 6: Verify trigger + mode + nhắc ngưỡng có mặt**

Run: `grep -nE 'consolidate|NGUONG_TONG|NGUONG_NGAN|Hai mode|flow-consolidate' skills/okr-retro/SKILL.md`
Expected: thấy trigger consolidate ở description + "Khi nào chạy", "Hai mode", nhắc ngưỡng với 2 hằng số, và link `flow-consolidate.md`.

- [ ] **Step 7: Commit**

```bash
git add skills/okr-retro/SKILL.md
git commit -m "feat(okr-retro): SKILL.md them trigger consolidate + nhac nguong Buoc 7 (Dot 17)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: Cập nhật `data-format.md` (Quy ước consolidate + obsolete mọi ngăn)

**Files:**
- Modify: `skills/okr-retro/references/data-format.md`

- [ ] **Step 1: Mở rộng bảng Status values cho `obsolete` áp mọi ngăn khi consolidate**

Tìm block (cuối bảng Status values + ghi chú vòng đời):

```
| project | local | `active` | Tri thức còn đúng |
| project | local | `obsolete` | Tri thức hết đúng (objective đổi, đã khắc phục) |

> **Vòng đời theo scope, không theo type:** `shared` đi `pending → ported` (hàng đợi port). `local` đi `active → obsolete` (sống tại chỗ). Workflow là loại duy nhất có thể đổi scope: xem "Thăng scope" ở `SKILL.md`.
```

Thay bằng:

```
| project | local | `active` | Tri thức còn đúng |
| project | local | `obsolete` | Tri thức hết đúng (objective đổi, đã khắc phục) |

> **Vòng đời theo scope, không theo type:** `shared` đi `pending → ported` (hàng đợi port). `local` đi `active → obsolete` (sống tại chỗ). Workflow là loại duy nhất có thể đổi scope: xem "Thăng scope" ở `SKILL.md`.
>
> **Ngoại lệ consolidate:** mode `consolidate` (xem `flow-consolidate.md`) có thể đánh `obsolete` cho bài Ở BẤT KỲ NGĂN NÀO khi bài đó bị gộp / thua mâu thuẫn / chuyển ngăn, kể cả `skill` (shared) và `workflow shared`. Đây KHÔNG phải status value mới (`obsolete` đã có), chỉ là mở rộng phạm vi áp dụng. Bài skill/workflow-shared `obsolete` rời hàng đợi port, xuống bảng "Lưu trữ".
```

- [ ] **Step 2: Thêm mục "Quy ước consolidate" cuối file**

Tìm block cuối file (mục Quy ước):

```
- `essence` là SOT của dòng index: sửa essence trong file thì cập nhật lại dòng index.
- Trùng bài: cập nhật file cũ (tinh chỉnh essence, có thể thêm ngày gặp lại vào Bối cảnh), KHÔNG tạo file mới.
```

Thêm NGAY SAU block đó (cuối file) mục mới:

```
## Quy ước consolidate (mode `consolidate`)

Dọn kho toàn diện. Flow đầy đủ: `flow-consolidate.md`. Convention archive-trace:

- **Không xoá file.** Bài bị gộp / thua mâu thuẫn / chuyển ngăn → `status: obsolete` (KHÔNG xoá), thêm 1 dòng body trỏ về bài sống sót. Giữ trace.
- **Dòng trỏ-về** chỉ là text trong body bài `obsolete`, KHÔNG phải field frontmatter. Ba mẫu:
  - Gộp: "Đã gộp vào Xxxx".
  - Mâu thuẫn: "Bị thay bởi Xxxx (mâu thuẫn, [lý do])".
  - Reclassify: "Chuyển ngăn thành Xxxx".
- **Zero schema mới, zero status enum value mới.** Bốn thao tác consolidate (gộp / tách / reclassify / giải mâu thuẫn) phân rã hết về 4 primitive ghi đã có: tạo file, cập nhật file, đánh `obsolete` + chuyển Lưu trữ, thăng scope local→shared.
- **Reclassify sai ngăn** đẻ file mới ở ngăn đích (`id` mới theo prefix S/W/P), bài cũ `obsolete`. **Thăng scope** sửa tại chỗ, KHÔNG đổi `id`. Phân biệt rõ hai nhánh ở `flow-consolidate.md` mục "Mô hình ghi".
```

- [ ] **Step 3: Verify không có em-dash/en-dash**

Run: `grep -nP '[\x{2014}\x{2013}]' skills/okr-retro/references/data-format.md`
Expected: KHÔNG output (exit 1).

- [ ] **Step 4: Verify mục mới có mặt**

Run: `grep -nE 'Quy ước consolidate|Ngoại lệ consolidate|Đã gộp vào|Bị thay bởi|Chuyển ngăn thành' skills/okr-retro/references/data-format.md`
Expected: thấy heading "Quy ước consolidate", note "Ngoại lệ consolidate" trong Status values, và 3 mẫu dòng trỏ-về.

- [ ] **Step 5: Commit**

```bash
git add skills/okr-retro/references/data-format.md
git commit -m "feat(okr-retro): data-format them Quy uoc consolidate + obsolete moi ngan (Dot 17)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: Cập nhật routing ở `okr-harness/references/flows.md`

**Files:**
- Modify: `skills/okr-harness/references/flows.md`

- [ ] **Step 1: Thêm route consolidate vào Section 7 (Retro flow)**

Tìm block cuối Section 7:

```
Record-only: `okr-retro` KHÔNG sửa file skill. Skill và workflow shared là hàng đợi port thủ công về repo gốc. Chỉ user chủ động (hoặc đồng ý gợi ý cuối flow) mới chạy.
```

Thay bằng:

```
Record-only: `okr-retro` KHÔNG sửa file skill. Skill và workflow shared là hàng đợi port thủ công về repo gốc. Chỉ user chủ động (hoặc đồng ý gợi ý cuối flow) mới chạy.

**Route consolidate**: intent "dọn kho lessons" / "consolidate" / "sắp xếp lại bài học" / "gộp bài học" / "rà soát lessons" → `okr-retro` mode `consolidate` (quét toàn KHO, không quét phiên). Flow C1-C6 ở `okr-retro/references/flow-consolidate.md`. Cũng kích hoạt khi user đồng ý lời nhắc ngưỡng cuối retro-thường (Bước 7). Vẫn record-only, confirm từng dòng.
```

- [ ] **Step 2: Verify không có em-dash/en-dash**

Run: `grep -nP '[\x{2014}\x{2013}]' skills/okr-harness/references/flows.md`
Expected: KHÔNG output (exit 1).

- [ ] **Step 3: Verify route có mặt**

Run: `grep -nE 'Route consolidate|consolidate' skills/okr-harness/references/flows.md`
Expected: thấy "Route consolidate" + danh sách intent.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-harness/references/flows.md
git commit -m "feat(okr-harness): route intent don kho lessons -> okr-retro consolidate (Dot 17)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 5: Làm rõ SOT ở `okr-shared/references/sot-ownership.md`

**Files:**
- Modify: `skills/okr-shared/references/sot-ownership.md`

- [ ] **Step 1: Mở rộng dòng lessons trong bảng SOT**

Tìm dòng:

```
| Bài học (`.okr/lessons/**`: 3 ngăn skill/workflow/project, tạo/sửa/thăng scope/đánh dấu obsolete, ported) | `okr-retro` |
```

Thay bằng:

```
| Bài học (`.okr/lessons/**`: 3 ngăn skill/workflow/project, tạo/sửa/thăng scope/đánh dấu obsolete, ported; gồm cả dọn kho mode `consolidate`) | `okr-retro` |
```

- [ ] **Step 2: Verify không có em-dash/en-dash**

Run: `grep -nP '[\x{2014}\x{2013}]' skills/okr-shared/references/sot-ownership.md`
Expected: KHÔNG output (exit 1).

- [ ] **Step 3: Verify dòng đã cập nhật**

Run: `grep -nE 'consolidate' skills/okr-shared/references/sot-ownership.md`
Expected: thấy "mode `consolidate`" trong dòng lessons.

- [ ] **Step 4: Commit**

```bash
git add skills/okr-shared/references/sot-ownership.md
git commit -m "docs(okr-shared): sot-ownership lam ro consolidate van la okr-retro ghi (Dot 17)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 6: Lịch sử Đợt 17 ở `CLAUDE.md` + `CHANGELOG.md`

**Files:**
- Modify: `CLAUDE.md`
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Thêm mục Đợt 17 vào "Lịch sử phát triển" của CLAUDE.md**

Tìm dòng cuối list "Lịch sử phát triển" (mục Đợt 16):

```
13. Đợt 16: Đúc kết quy trình/skill mới ở runtime (retro 3 ngăn). Áp định nghĩa skill = công cụ ("how") / workflow = bản đồ điều phối ("khi nào/thứ tự") / project = tri thức tĩnh. Mức A (lớp nhãn, KHÔNG tách vật lý). `okr-retro` từ 2 ngăn thành 3: thêm `lessons/workflow/` (Wxxx) + 2 field cứng `mode` (improve/create) + `scope` (shared/local), mở rộng `type`. Hai phép thử: phân loại (how/thứ tự/dữ kiện) + phân tuyến scope ("copy sang project khác còn dùng?"). Ngưỡng bất đối xứng chống phình harness: improve = 1, create-local = 1, create-shared ≥3 mốc (hoặc user chủ động). Workflow-local là phòng chờ của shared: đủ 3 mốc + tổng quát thì thăng scope. Cập nhật output schema harness (scope, lessons_promoted, pending_port_lessons). Chỉ `.okr/` runtime.
```

Thêm NGAY SAU dòng đó (dòng số 14 mới):

```
14. Đợt 17: Dọn kho lessons (mode `consolidate`). Vá gap "retro chỉ dọn per-item reactive, kho lớn dần sinh trùng/quá-tải/sai-ngăn/mâu-thuẫn". `okr-retro` thêm mode `consolidate`: quét CHỦ ĐỘNG toàn kho, đề xuất 4 thao tác (gộp N→1, tách 1→N, reclassify sai-ngăn/thăng-scope, giải mâu thuẫn) cho user duyệt từng dòng. Flow C1-C6 ở `flow-consolidate.md` (load on-demand). Lõi: zero schema mới, zero status enum value mới, 4 thao tác phân rã hết về 4 primitive ghi đã có (tạo/cập nhật/obsolete/thăng-scope); chỉ thêm lớp ĐỌC+PHÁT HIỆN+ĐỀ XUẤT. Cộng nhắc theo ngưỡng cuối retro-thường (Bước 7): tổng > 12 HOẶC một ngăn > 8, confirm-gated, không tự chạy. Guard kho nhỏ < 6 bài thì dừng. Mở rộng `obsolete` áp mọi ngăn (kể cả skill/workflow-shared) khi consolidate. Reachability Đợt 15 tự lo (obsolete neo Lưu trữ). Chỉ `.okr/` runtime.
```

- [ ] **Step 2: Thêm hàng Đợt 17 vào đầu bảng CHANGELOG.md**

Tìm hàng Đợt 16 (hàng đầu tiên của bảng, ngay sau header `| ---------- | ... |`):

```
| 2026-06-05 | Đợt 16: retro 3 ngăn. Áp định nghĩa skill=công cụ / workflow=bản đồ / project=tri thức. `okr-retro` thêm ngăn `lessons/workflow/` (Wxxx) + 2 field `mode`/`scope`, mở rộng `type`. 2 phép thử (phân loại how/thứ tự/dữ kiện + phân tuyến scope "copy sang project khác còn dùng?"). Ngưỡng bất đối xứng: improve=1, create-local=1, create-shared≥3 mốc. Workflow-local là phòng chờ của shared (đủ 3 mốc + tổng quát thăng scope). Output schema harness +scope/+lessons_promoted, đổi pending_skill thành pending_port. | okr-retro (data-format + SKILL), okr-shared (sot-ownership + preload), okr-harness (SKILL+flows+skill-contract), CLAUDE.md | Vá gap "quy trình/năng lực mới phát sinh ở runtime không có nhà đúc kết": phân biệt rõ công cụ vs bản đồ, có đường thăng hạng local thành shared, chặn rác chảy vào harness chung |
```

Chèn NGAY TRƯỚC hàng Đợt 16 đó (thành hàng đầu tiên của bảng) hàng mới:

```
| 2026-06-05 | Đợt 17: dọn kho lessons (mode `consolidate`). `okr-retro` thêm mode quét CHỦ ĐỘNG toàn kho, đề xuất 4 thao tác (gộp N→1, tách 1→N, reclassify sai-ngăn/thăng-scope, giải mâu thuẫn) cho user duyệt từng dòng. Flow C1-C6 ở `flow-consolidate.md` (on-demand). Zero schema mới, zero status enum value mới: 4 thao tác phân rã về 4 primitive ghi đã có, chỉ thêm lớp đọc+phát hiện+đề xuất. Nhắc theo ngưỡng cuối retro-thường (tổng>12 HOẶC ngăn>8, confirm-gated). Guard kho nhỏ <6 bài. Mở rộng `obsolete` áp mọi ngăn khi consolidate. | okr-retro (flow-consolidate mới + SKILL + data-format), okr-harness (flows), okr-shared (sot-ownership), CLAUDE.md | Vá gap "retro chỉ dọn per-item reactive": kho lớn dần sinh bài trùng/quá-tải/sai-ngăn/mâu-thuẫn, mất tin cậy + phình token auto-load. Thêm lượt rà soát toàn kho, an toàn (record-only, confirm từng dòng, không xoá) |
```

- [ ] **Step 3: Verify không có em-dash/en-dash ở cả hai file**

Run: `grep -nP '[\x{2014}\x{2013}]' CLAUDE.md CHANGELOG.md`
Expected: KHÔNG output (exit 1). Lưu ý: `≥` (U+2265) và `→` (U+2192) KHÔNG phải dash, được phép (đã dùng ở các đợt trước).

- [ ] **Step 4: Verify mục Đợt 17 có mặt**

Run: `grep -nE 'Đợt 17' CLAUDE.md CHANGELOG.md`
Expected: thấy Đợt 17 ở cả hai file.

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md CHANGELOG.md
git commit -m "docs(okr-retro): lich su Dot 17 consolidate (CLAUDE.md, CHANGELOG.md)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 7: Self-review consistency toàn bộ

**Files:** (read-only, không sửa trừ khi phát hiện lỗi)

- [ ] **Step 1: Grep dash toàn bộ file đã đụng**

Run:
```bash
grep -nP '[\x{2014}\x{2013}]' \
  skills/okr-retro/references/flow-consolidate.md \
  skills/okr-retro/SKILL.md \
  skills/okr-retro/references/data-format.md \
  skills/okr-harness/references/flows.md \
  skills/okr-shared/references/sot-ownership.md \
  CLAUDE.md CHANGELOG.md
```
Expected: KHÔNG output. Nếu match → sửa dòng đó, commit fix.

- [ ] **Step 2: Verify hằng số nhất quán mọi nơi**

Run: `grep -rnE 'NGUONG_TONG|NGUONG_NGAN|GUARD_NHO' skills/okr-retro/`
Expected: `NGUONG_TONG`=12, `NGUONG_NGAN`=8, `GUARD_NHO`=6 nhất quán giữa SKILL.md (nhắc ngưỡng) và flow-consolidate.md (định nghĩa). Nếu lệch số → sửa cho khớp.

- [ ] **Step 3: Verify tên flow nhất quán mọi nơi**

Run: `grep -rnE 'flow-consolidate' skills/ CLAUDE.md CHANGELOG.md`
Expected: mọi tham chiếu đều là `flow-consolidate.md` (không có biến thể `flow_consolidate` hay `consolidate-flow`).

- [ ] **Step 4: Verify đối chiếu spec (coverage)**

Đọc lại `docs/superpowers/specs/2026-06-05-okr-retro-consolidate-design.md` mục 2 "Done trông thế nào" (7 tiêu chí). Tự đối chiếu từng tiêu chí với file đã sửa:
  1. Trigger consolidate chạy được → SKILL.md description + Khi nào chạy (Task 2). ✓
  2. Flow C1-C6 ở flow-consolidate.md, SKILL.md trỏ sang → Task 1 + Task 2 Step 2/4. ✓
  3. 4 thao tác phân rã về primitive, không thêm field/status enum → flow-consolidate "Mô hình ghi" + data-format "Quy ước consolidate" (Task 1, 3). ✓
  4. Confirm từng dòng, không xoá file → C4 + Nguyên tắc (Task 1). ✓
  5. Bước 7 nhắc ngưỡng 1 dòng confirm-gated → Task 2 Step 3. ✓
  6. Reachability không flag nhầm → mục "Reachability tự lo" (Task 1), obsolete neo Lưu trữ. ✓
  7. Không còn em-dash/en-dash → Step 1 task này. ✓

Nếu tiêu chí nào CHƯA có file/đoạn tương ứng → quay lại task liên quan bổ sung.

- [ ] **Step 5: Verify build sạch (git status)**

Run: `git status --short && git log --oneline -7`
Expected: working tree clean, 6 commit Đợt 17 (Task 1-6) theo thứ tự.

---

## Tổng kết

7 task, mỗi task 1 commit (Task 7 chỉ verify, commit thêm nếu phát hiện lỗi). Thay đổi thuần markdown prompt, không đụng schema frontmatter, không thêm status enum value mới, không đụng `okr-analyze`. Lõi an toàn: 4 thao tác consolidate phân rã hết về primitive ghi đã có, record-only, confirm từng dòng, không xoá file.
