---
name: okr-retro
description: "Rút bài học từ phiên OKR, lưu vào layer lessons. Record-only. Cũng dọn kho lessons khi user gọi: dọn kho lessons, consolidate, sắp xếp lại bài học, gộp bài học, rà soát lessons."
---

# OKR Retro: Rút bài học từ phiên

Chạy inline. Trích bài học từ hội thoại phiên hiện tại, phân loại, ghi vào `.okr/lessons/`. **Record-only**: KHÔNG tự sửa skill, KHÔNG tự áp dụng cải tiến.

## Khi nào chạy

- **User chủ động (retro-thường)**: "rút bài học", "tổng kết phiên", "retro".
- **User chủ động (consolidate)**: "dọn kho lessons", "consolidate", "sắp xếp lại bài học", "gộp bài học", "rà soát lessons". Chạy mode `consolidate`, xem dưới.
- **Harness gợi ý cuối flow** (1 dòng), user đồng ý mới chạy. KHÔNG tự chạy ngầm.

### Hai mode

- **retro-thường** (mặc định, Flow Bước 1-7 dưới): quét PHIÊN hiện tại, trích bài MỚI.
- **consolidate**: quét toàn KHO `lessons/` đã có (không quan tâm phiên), đề xuất gộp / tách / reclassify / giải mâu thuẫn. Flow đầy đủ ở `references/flow-consolidate.md`. Đọc file đó khi trigger consolidate kích hoạt.

## Ba loại bài học

| Loại | Là gì | Phép thử nhận diện | Lưu ở | Scope | Vòng đời |
|------|-------|--------------------|-------|-------|----------|
| A - skill (công cụ) | Năng lực nguyên tử của harness: làm được việc gì mới | Trả lời **"how?"** | `lessons/skill/` | luôn `shared` | `pending` → port → `ported` |
| C - workflow (bản đồ) | Trình tự điều phối công cụ để đạt mục tiêu | Trả lời **"khi nào / thứ tự nào?"** | `lessons/workflow/` | **lưỡng tính** | shared: `pending`→`ported` · local: `active`→`obsolete` |
| B - project (tri thức) | Sự thật/định nghĩa tĩnh của project | Là **dữ kiện**, không làm theo bước được | `lessons/project/` | luôn `local` | `active` → `obsolete` |

### Hai phép thử khi quét phiên

1. **Phân loại (A/C/B)**: "how?" → skill · "thứ tự/khi nào dùng cái gì?" → workflow · "dữ kiện tĩnh, không thực hiện theo bước?" → project.
2. **Phân tuyến scope (chỉ workflow)**: hỏi **"Copy harness sang một project OKR khác, cái này còn dùng được không?"** Còn → `shared` (ứng viên port về harness). Phụ thuộc domain/objective cụ thể → `local` (playbook sống tại project).

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

## Flow

### Bước 1: Đọc index hiện có (Preload Contract Tier 2)
Đảm bảo nền Tier 2 đã có trong context (`../okr-shared/references/preload.md` Tier 2): `.okr/lessons/index.md` (toàn bộ, để dedup) + `objective.md` frontmatter (để gắn `area`/`period` cho bài học). Idempotent: qua harness đã có, chạy lẻ tự đọc phần thiếu. Folder `lessons/` chưa tồn tại → sẽ tạo khi ghi (Bước 6).

### Bước 2: Quét phiên
Nhìn lại TOÀN BỘ hội thoại phiên hiện tại (kể cả phần không phải OKR). Tìm khoảnh khắc sinh bài học:
- Chỗ user sửa/bác bỏ cách làm của bạn.
- Chỗ vướng, làm lại, hiểu nhầm rồi vỡ ra.
- Quy ước/ràng buộc/đặc thù project mới lộ ra.
- Giới hạn/lỗi/điểm khó dùng của bộ skill.

### Bước 3: Lọc theo chất lượng
Chỉ giữ bài học **tái dùng được + không hiển nhiên + actionable**. Bỏ:
- Sự kiện một lần, không lặp lại.
- Điều quá hiển nhiên ("nên đọc file trước khi sửa").
- Việc đã ghi sẵn trong skill/CLAUDE.md.

### Bước 4: Phân loại + soạn essence + dedup
Mỗi bài: chạy 2 phép thử (phân loại A/C/B + phân tuyến scope), gán `type` + `mode` + `scope`, soạn `essence` 1 dòng, gán `target` (skill/workflow) hoặc `area` (project). Áp ngưỡng đúc kết: bài `workflow create-shared` chưa đủ 3 mốc thì hạ xuống `workflow local` (phòng chờ), KHÔNG ghi thẳng shared. Đối chiếu index: trùng/giao bài cũ → "Cập nhật Sxxx/Wxxx/Pxxx". Workflow-local đã đủ 3 mốc + tổng quát → đánh dấu "Thăng W0xx → shared".

### Bước 5: Trình bảng ứng viên + tự dọn nhẹ
Hiển thị bảng cho user tick/cắt/sửa:

```
| # | Loại | Mode | Scope | Essence | Target/Area | Mới/Cập nhật/Thăng |
|---|------|------|-------|---------|-------------|--------------------|
| 1 | A skill | improve | shared | ... | okr-track/flow-light.md | Mới |
| 2 | C workflow | create | local | ... | playbook tuần | Mới |
| 3 | C workflow | create | shared | ... | flow mới: okr-track/flow-weekly-sync | Thăng W002 local→shared |
| 4 | B project | - | local | ... | capacity | Cập nhật P003 |
```

Đồng thời nêu bài loại B nghi **lỗi thời** (objective đã đổi, blocker đã hết) để user xác nhận `obsolete`/xoá. Chờ user duyệt. KHÔNG ghi trước khi user chọn.

### Bước 6: Ghi
Với mỗi bài user giữ:
- **Mới**: tạo file theo template (`references/data-format.md`), `id` tăng dần trong loại.
- **Cập nhật**: sửa file cũ (tinh chỉnh essence/bối cảnh).
- **Thăng scope** (chỉ workflow local → shared): sửa file `Wxxx` cùng chỗ, đổi `scope: local` → `shared`, `status: active` → `pending`. Chuyển dòng index từ cột scope `local`/`active` sang `shared`/`pending`. KHÔNG đổi `id`, KHÔNG tạo file mới.
- Cập nhật `index.md`: thêm/sửa dòng. Bài `obsolete`/`ported` chuyển mục "Lưu trữ".
- Folder `.okr/lessons/` chưa có → tạo `index.md` + `skill/` + `workflow/` + `project/`.

### Bước 7: Báo cáo
Tóm tắt: ghi mới mấy bài, cập nhật mấy, thăng scope mấy, đánh dấu lỗi thời mấy. **Hàng đợi port** = skill `pending` + workflow `shared pending`: nếu >0 → nhắc "X bài chờ port về repo gốc objective-kit (Y công cụ + Z workflow tổng quát)."

**Nhắc ngưỡng consolidate** (1 dòng, confirm-gated): SAU khi ghi xong, đếm bài còn hiệu lực (`pending`+`active`). Nếu **tổng > `NGUONG_TONG`** (mặc định 12) HOẶC **một ngăn bất kỳ > `NGUONG_NGAN`** (mặc định 8) → nhắc 1 dòng, vd: "Kho lessons đã 14 bài (workflow 9). Cân nhắc chạy `consolidate` để gộp/dọn?" User đồng ý mới chạy consolidate. KHÔNG tự chạy. Hằng số định nghĩa ở `references/flow-consolidate.md`. Chấp nhận có thể nhắc lại phiên sau (nhẹ, user bỏ qua được), không ghi nhớ "đã nhắc".

## Quy tắc

- **Record-only**. KHÔNG sửa file trong `.claude/skills/`. KHÔNG tự áp dụng cải tiến.
- **Confirm trước ghi** (bảng ứng viên).
- **Trùng thì cập nhật**, không đẻ bản mới. Workflow local đủ chín thì thăng scope, không đẻ bản shared mới.
- **SOT**: chỉ `okr-retro` ghi `.okr/lessons/**`. Xem `okr-shared/references/sot-ownership.md`.
- **Schema**: `references/data-format.md`.
- **Consolidate**: dọn kho toàn diện ở `references/flow-consolidate.md`. Cùng record-only + confirm từng dòng, zero schema mới.

## Test scenarios

### Trích xuất sau phiên track
1. User vừa track xong, gõ "rút bài học".
2. okr-retro đọc index, quét hội thoại.
3. Tìm thấy: user phàn nàn flow track hỏi quá nhiều (loại A, target `okr-track/flow-light.md`) + nhận ra KR2 đặt baseline sai (loại B, area "định nghĩa KR").
4. Trình bảng 2 ứng viên. User giữ cả 2.
5. Ghi S001 + P001, cập nhật index.
6. Báo: "Ghi 2 bài. 1 bài cải tiến skill (S001) chờ port về repo gốc."

### Dedup
1. Phiên sau, user lại "rút bài học".
2. Bài về capacity trùng P002 đã có.
3. Bảng đánh dấu "Cập nhật P002". User đồng ý.
4. Tinh chỉnh essence P002, KHÔNG tạo P00x mới.

### Thăng workflow local lên shared
1. Phiên 1: user lập quy trình "thứ Hai sync action với lịch rồi mới track". Ghi W001 workflow/local/create, 1 mốc.
2. Phiên 2, 3: gặp lại cùng quy trình. Dedup cập nhật W001, thêm mốc 2, 3.
3. Phiên 3: đủ 3 mốc + phép thử "copy sang project khác vẫn đúng" = PASS (không phụ thuộc domain).
4. Đề xuất thăng W001 → shared. User đồng ý. Đổi scope=shared, status=pending.
5. Báo: "Thăng 1 workflow. 1 bài chờ port về repo gốc (workflow tổng quát)."

### Không có bài học đáng ghi
1. Phiên ngắn, chỉ xem dashboard.
2. okr-retro quét, không có gì tái dùng được.
3. Báo: "Phiên này chưa có bài học đáng ghi." Không ghi gì.
