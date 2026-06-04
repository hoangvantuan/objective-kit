---
name: okr-retro
description: "Rút bài học từ phiên OKR, lưu vào layer lessons. Record-only."
---

# OKR Retro: Rút bài học từ phiên

Chạy inline. Trích bài học từ hội thoại phiên hiện tại, phân loại, ghi vào `.okr/lessons/`. **Record-only**: KHÔNG tự sửa skill, KHÔNG tự áp dụng cải tiến.

## Khi nào chạy

- **User chủ động**: "rút bài học", "tổng kết phiên", "retro".
- **Harness gợi ý cuối flow** (1 dòng), user đồng ý mới chạy. KHÔNG tự chạy ngầm.

## Hai loại bài học

| Loại | Là gì | Lưu ở | Vòng đời |
|------|-------|-------|----------|
| A - Cải tiến skill | Bài học về chính bộ skill OKR (flow vướng, routing sai, thiếu field, mô tả không trigger) | `.okr/lessons/skill/` | Record-only, `pending` → user port về repo gốc → `ported` |
| B - Project cụ thể | Tri thức về project đang làm (định nghĩa KR, ước lượng capacity, đặc thù domain) | `.okr/lessons/project/` | `active` → `obsolete` khi hết đúng |

Phân biệt nhanh: sửa được bằng cách đổi file trong `.claude/skills/` → **loại A**. Về nội dung công việc/mục tiêu → **loại B**.

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
Mỗi bài: gán `type` (A/B), soạn `essence` 1 dòng (câu lõi hành động), gán `target` (loại A) hoặc `area` (loại B). Đối chiếu index: trùng/giao bài cũ → đánh dấu "Cập nhật Sxxx/Pxxx" thay vì tạo mới.

### Bước 5: Trình bảng ứng viên + tự dọn nhẹ
Hiển thị bảng cho user tick/cắt/sửa:

```
| # | Loại | Essence | Target/Area | Mới/Cập nhật |
|---|------|---------|-------------|--------------|
| 1 | A | ... | okr-plan/flow-new.md | Mới |
| 2 | B | ... | capacity | Cập nhật P003 |
```

Đồng thời nêu bài loại B nghi **lỗi thời** (objective đã đổi, blocker đã hết) để user xác nhận `obsolete`/xoá. Chờ user duyệt. KHÔNG ghi trước khi user chọn.

### Bước 6: Ghi
Với mỗi bài user giữ:
- **Mới**: tạo file theo template (`references/data-format.md`), `id` tăng dần trong loại.
- **Cập nhật**: sửa file cũ (tinh chỉnh essence/bối cảnh).
- Cập nhật `index.md`: thêm/sửa dòng. Bài `obsolete`/`ported` chuyển mục "Lưu trữ".
- Folder `.okr/lessons/` chưa có → tạo `index.md` + `skill/` + `project/`.

### Bước 7: Báo cáo
Tóm tắt: ghi mới mấy bài, cập nhật mấy, đánh dấu lỗi thời mấy. Nếu có loại A `pending` → nhắc: "X bài cải tiến skill đang chờ port về repo gốc objective-kit."

## Quy tắc

- **Record-only**. KHÔNG sửa file trong `.claude/skills/`. KHÔNG tự áp dụng cải tiến.
- **Confirm trước ghi** (bảng ứng viên).
- **Trùng thì cập nhật**, không đẻ bản mới.
- **SOT**: chỉ `okr-retro` ghi `.okr/lessons/**`. Xem `okr-shared/references/sot-ownership.md`.
- **Schema**: `references/data-format.md`.

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

### Không có bài học đáng ghi
1. Phiên ngắn, chỉ xem dashboard.
2. okr-retro quét, không có gì tái dùng được.
3. Báo: "Phiên này chưa có bài học đáng ghi." Không ghi gì.
