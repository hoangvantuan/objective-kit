# Design: okr-retro mode `consolidate` (Đợt 17, dọn kho lessons)

> Spec thiết kế. Nền: Đợt 16 (lessons 3 ngăn skill/workflow/project) ĐÃ hoàn thành trong repo `skills/`. Đợt 17 bổ sung cơ chế rà soát chủ động toàn kho.

## 1. Vấn đề (Khổ Đế)

`okr-retro` hiện chỉ dọn dẹp **per-item, reactive**, kích hoạt lúc trích một bài cụ thể từ phiên:

- **Dedup** (Bước 4): so bài ĐANG ghi với index, trùng thì cập nhật. Chỉ bắt trùng 1-1, không gộp 3 bài rời rạc đã tồn tại.
- **Đánh obsolete** (Bước 5): nêu bài loại B nghi lỗi thời. Vẫn per-item.
- **Thăng scope** (Đợt 16): workflow local→shared khi đủ 3 mốc. Theo từng bài, không tái cấu trúc kho.

Cả ba chạy khi đang trích bài MỚI từ phiên. Không bước nào "mở toàn bộ `lessons/`, đọc hết, rồi gộp nhóm / tách bài quá tải / sắp xếp lại / phát hiện 2 bài mâu thuẫn".

Hệ quả khi kho lớn dần: bài trùng lặp, bài quá tải nhiều rule, bài đặt sai ngăn, hai bài khuyên ngược nhau. Kho mất tin cậy, index phình, tốn token auto-load mỗi phiên.

## 2. Mục tiêu (Diệt Đế)

Thêm mode `consolidate` cho `okr-retro`: một lượt quét CHỦ ĐỘNG toàn kho, đề xuất 4 thao tác cho user duyệt:

1. **Gộp nhóm**: nhiều bài nhỏ cùng chủ đề → 1 bài tổng quát hơn.
2. **Tách**: 1 bài ôm quá nhiều rule → tách theo target.
3. **Sắp xếp lại (reclassify)**: bài đặt sai ngăn → chuyển ngăn; reorder index; bắt workflow-local đủ 3 mốc mà lỡ chưa thăng scope.
4. **Giải mâu thuẫn**: 2 bài khuyên ngược → giữ 1, archive 1.

Cộng thêm: cơ chế **nhắc theo ngưỡng** để user biết khi nào nên chạy.

### Done trông thế nào (tiêu chí verify)

1. `okr-retro` có mode `consolidate` chạy được khi user gõ trigger ("dọn kho lessons", "consolidate", "sắp xếp lại bài học", "gộp bài học", "rà soát lessons").
2. Flow C1-C6 đầy đủ trong `okr-retro/references/flow-consolidate.md`, SKILL.md trỏ sang.
3. Bốn thao tác phân rã hết về primitive ghi sẵn có. KHÔNG thêm field schema, KHÔNG thêm status enum.
4. Confirm từng dòng trước ghi. Không xoá file (archive để trace).
5. Cuối flow retro-thường (Bước 7) có nhắc ngưỡng 1 dòng, confirm-gated.
6. Reachability Đợt 15 không flag nhầm: bài mới neo index, bài obsolete neo Lưu trữ.
7. KHÔNG còn em-dash/en-dash ở mọi file sửa.

## 3. Quyết định thiết kế đã chốt (qua phiên brainstorm)

| # | Nhánh | Chốt |
| --- | --- | --- |
| 1 | Nền | Xây trên Đợt 16 (3 ngăn) đã hoàn thành. Đây là Đợt 17. |
| 2 | Chủ sở hữu | Mode mới trong `okr-retro` (1 skill làm cả đọc lẫn ghi, đúng SOT lessons/**). |
| 3 | Kích hoạt | User chủ động gọi + nhắc theo ngưỡng (confirm-gated). KHÔNG chạy ngầm. |
| 4 | Phạm vi v1 | Cả 4 thao tác: gộp, tách, reclassify, giải mâu thuẫn. |
| 5 | Cấu trúc quét | Hướng A: gom cụm trước, xử lý sau (bám target/area/chủ đề, ít nhiễu, trace tốt). |
| 6 | Đo ngưỡng | Tổng > 12 HOẶC một ngăn bất kỳ > 8 (bài còn hiệu lực pending+active). Hai hằng số tunable. |

## 4. Lõi thiết kế

**Consolidation KHÔNG thêm thao tác ghi mới.** Mọi việc gộp/tách/reclassify/giải-mâu-thuẫn phân rã thành 4 primitive ghi đã có:

- Tạo file mới (Bước 6 hiện tại).
- Cập nhật file (Bước 6 hiện tại).
- Đánh `obsolete` + chuyển "Lưu trữ" (đã có).
- Thăng scope local→shared (Đợt 16).

Cái mới duy nhất là lớp **ĐỌC + PHÁT HIỆN + ĐỀ XUẤT** toàn kho. Đây là điểm giữ thay đổi nhỏ và an toàn: không đụng schema, không đụng status enum, tái dùng audit reachability Đợt 15.

### Quy tắc kế thừa (giữ nguyên)

- Record-only + confirm từng dòng trước ghi.
- Không xoá file. Bài bị gộp/thua mâu thuẫn → `obsolete`, thêm 1 dòng body trỏ về bài sống sót. Giữ trace.
- Chỉ `okr-retro` ghi `lessons/**` (SOT nguyên vẹn).
- Chỉ chạy khi user gọi hoặc đồng ý với lời nhắc.

## 5. Flow mode `consolidate`

Tách khỏi flow trích-bài hiện tại. Phân biệt rõ:
- **retro-thường**: quét PHIÊN hiện tại, trích bài MỚI.
- **consolidate**: quét toàn KHO đã có, không quan tâm phiên.

| Bước | Việc |
|------|------|
| **C1 Nạp kho** | Read `index.md` + body mọi bài còn hiệu lực (`pending`/`active`) trong `skill/`, `workflow/`, `project/`. Bỏ bài ở Lưu trữ (`ported`/`obsolete`). **Guard kho nhỏ**: tổng < 6 bài → báo "kho còn nhỏ, chưa cần dọn" rồi dừng (tránh tốn token vô ích). |
| **C2 Gom cụm** | Nhóm bài theo khóa: `(type, target)` cho skill/workflow, `(type, area)` cho project; cộng gom theo chủ đề `essence` khi khác target/area nhưng cùng việc. Bài lạc cụm vẫn xét riêng ở C3. |
| **C3 Xét 4 thao tác** | Trong cụm ≥2 bài: **gộp** (trùng/giao → 1 bài tổng quát), **mâu thuẫn** (khuyên ngược → giữ 1 theo date + bằng chứng). Mỗi bài (kể cả đứng riêng): **tách** (ôm ≥2 rule khác target → tách N), **reclassify** (chạy lại 2 phép thử Đợt 16: sai ngăn → chuyển ngăn; workflow-local đủ 3 mốc + qua phép thử shared mà chưa thăng → đề xuất thăng). Cuối: reorder index cho gọn. |
| **C4 Trình bảng** | Bảng đề xuất, user tick từng dòng. KHÔNG ghi trước duyệt. |
| **C5 Ghi** | Áp primitive tương ứng cho dòng user giữ (xem mục 6). Cập nhật index 3 bảng + Lưu trữ. |
| **C6 Báo cáo** | Gộp X, tách Y, reclassify Z, mâu thuẫn W, thăng scope V. Kho N→M bài. Cập nhật hàng đợi port nếu sinh shared pending. |

### Bảng đề xuất C4 (mẫu)

```
| # | Thao tác    | Bài liên quan     | Kết quả                          | Lý do                            |
|---|-------------|-------------------|----------------------------------|----------------------------------|
| 1 | Gộp         | W003 + W005       | → W003 tổng quát, W005 obsolete  | cùng target flow-light, giao 80% |
| 2 | Tách        | P002              | → P002 + P00x mới                | ôm 2 area: capacity + KR         |
| 3 | Reclassify  | P004 → W00x       | tạo W00x, P004 obsolete          | trả lời "thứ tự nào" = workflow  |
| 4 | Mâu thuẫn   | S002 vs S006      | giữ S006, S002 obsolete          | S006 mới hơn + 3 bằng chứng      |
| 5 | Thăng scope | W001 local→shared | scope=shared, status=pending     | đủ 3 mốc, qua phép thử           |
```

## 6. Mô hình ghi: mỗi thao tác phân rã thành primitive nào

| Thao tác | Phân rã |
|----------|---------|
| **Gộp** (N→1) | Cập nhật bài sống sót (hợp `essence` + gộp `## Bằng chứng` + bối cảnh). Các bài còn lại: `status=obsolete` + dòng body "Đã gộp vào Xxxx". Index: bài sống sót giữ ở bảng chính, bài bị gộp xuống Lưu trữ. |
| **Tách** (1→N) | Bài gốc: sửa `essence` còn 1 rule. Mỗi rule tách ra: tạo file mới (`id` mới trong ngăn), neo index. |
| **Reclassify** (sai ngăn) | Tạo file ở ngăn đúng (`id` mới theo prefix S/W/P ngăn đích), copy nội dung + chỉnh field (`type`/`scope`/`target`/`area`) cho khớp ràng buộc đóng. Bài cũ `obsolete` + dòng body "Chuyển ngăn thành Xxxx". |
| **Reclassify** (chỉ thăng scope) | Sửa tại chỗ: `scope: local→shared`, `status: active→pending`. KHÔNG đổi `id`. (Đúng cơ chế Đợt 16, không đẻ bản mới.) |
| **Mâu thuẫn** | Bài thắng: giữ/tinh chỉnh. Bài thua: `obsolete` + dòng body "Bị thay bởi Xxxx (mâu thuẫn, [lý do])". |
| **Reorder index** | Chỉ sắp lại thứ tự dòng trong 3 bảng, không đụng file body. |

Điểm mấu chốt: tất cả chỉ là tổ hợp tạo/cập nhật/obsolete/thăng-scope. **Zero schema mới, zero status enum mới.** Note "Đã gộp vào / Bị thay bởi / Chuyển ngăn" chỉ là 1 dòng text trong body bài `obsolete`, không phải field.

## 7. Cơ chế nhắc theo ngưỡng

- **Đặt ở đâu**: cuối flow retro-thường (Bước 7 báo cáo), SAU khi ghi bài mới. KHÔNG nhắc mỗi phiên (tránh nhiễu), chỉ nhắc đúng lúc kho vừa phình thêm.
- **Điều kiện**: sau khi ghi, đếm bài còn hiệu lực (`pending`+`active`). Nhắc nếu **tổng > `NGUONG_TONG`** (mặc định 12) **HOẶC một ngăn bất kỳ > `NGUONG_NGAN`** (mặc định 8).
- **Hình thức**: 1 dòng, confirm-gated. Ví dụ: "Kho lessons đã 14 bài (workflow 9). Cân nhắc chạy `consolidate` để gộp/dọn?" User đồng ý mới chạy. KHÔNG tự chạy. Giữ đúng record-only.
- **Hằng số**: `NGUONG_TONG=12`, `NGUONG_NGAN=8`. Documented + tunable, đặt ở `flow-consolidate.md`.
- **Chống nhiễu v1**: nhắc khi lượt ghi kết thúc mà kho đang vượt ngưỡng. Chấp nhận có thể nhắc lại ở phiên sau (nhẹ, 1 dòng, user bỏ qua được). Không thêm cơ chế ghi nhớ "đã nhắc" để giữ đơn giản.

## 8. Reachability (Đợt 15) tự lo

Bài mới (tách/gộp/reclassify) neo index như mọi bài. Bài `obsolete` neo ở "Lưu trữ". Không sinh file mồ côi, không cần sửa audit.

## 9. File đụng tới (dev-time, repo `skills/`)

| File | Thay đổi | Lý do |
|------|----------|-------|
| `okr-retro/references/flow-consolidate.md` | **Tạo mới**: flow C1-C6 + bảng phân rã primitive + hằng số ngưỡng + test scenarios | Tách riêng vì consolidate hiếm chạy, load on-demand, giữ SKILL.md gọn (đúng pattern flow-*.md ở init/plan/track) |
| `okr-retro/SKILL.md` | Thêm trigger consolidate ở "Khi nào chạy"; 1 đoạn ngắn trỏ sang `flow-consolidate.md`; phân biệt retro-thường (quét phiên) vs consolidate (quét kho); thêm nhắc ngưỡng vào Bước 7 | SKILL.md là entry, không nhồi chi tiết |
| `okr-retro` frontmatter `description` | Thêm trigger: "dọn kho lessons, consolidate, sắp xếp lại bài học, gộp bài học, rà soát lessons" | Để skill tự trigger |
| `okr-retro/references/data-format.md` | Thêm mục nhỏ "Quy ước consolidate": note obsolete trỏ-về, không thêm status | Ghi rõ convention archive-trace |
| `okr-harness/references/flows.md` | Thêm route intent "dọn/rà soát kho lessons" → okr-retro consolidate | Harness routing |
| `okr-shared/references/sot-ownership.md` | 1 dòng làm rõ consolidate vẫn là okr-retro ghi | SOT minh bạch |
| `CLAUDE.md` + `CHANGELOG.md` | Lịch sử Đợt 17 | Convention repo |

## 10. Test scenarios (đưa vào flow-consolidate.md)

### Gộp nhóm
1. Kho có W003, W005 cùng target `okr-track/flow-light.md`, essence giao 80%.
2. User gõ "dọn kho lessons". C2 gom 2 bài vào 1 cụm.
3. C3 đề xuất gộp → W003 tổng quát, W005 obsolete.
4. User duyệt. Cập nhật W003 (hợp bằng chứng), W005 obsolete + "Đã gộp vào W003", index chuyển W005 xuống Lưu trữ.

### Tách
1. P002 ôm 2 rule: capacity + định nghĩa KR.
2. C3 đề xuất tách → P002 (giữ capacity) + P00x mới (định nghĩa KR).
3. User duyệt. Sửa P002 essence, tạo P00x, neo index.

### Reclassify sai ngăn
1. P004 thực ra mô tả "thứ tự nào" (workflow), không phải dữ kiện tĩnh.
2. C3 chạy lại 2 phép thử, phát hiện sai ngăn → đề xuất chuyển P004 thành W00x.
3. User duyệt. Tạo W00x (type=workflow, scope theo phép thử), P004 obsolete + "Chuyển ngăn thành W00x".

### Giải mâu thuẫn
1. S002 và S006 khuyên ngược nhau về cùng target.
2. C3 phát hiện mâu thuẫn → đề xuất giữ S006 (mới hơn + 3 bằng chứng), S002 obsolete.
3. User duyệt. S002 obsolete + "Bị thay bởi S006 (mâu thuẫn, S006 mới hơn + nhiều bằng chứng)".

### Nhắc ngưỡng
1. User chạy retro-thường, ghi xong tổng kho thành 14 bài (workflow 9 > 8).
2. Bước 7 báo cáo thêm 1 dòng: "Kho lessons đã 14 bài (workflow 9). Cân nhắc chạy consolidate?"
3. User đồng ý → chạy consolidate. Không đồng ý → dừng, không ghi gì thêm.

### Guard kho nhỏ
1. Kho mới có 4 bài. User gõ "consolidate".
2. C1 thấy tổng < 6 → báo "kho còn nhỏ, chưa cần dọn". Dừng, không quét.

## 11. Ngoài phạm vi v1 (ghi để khỏi trôi)

- Tự động chạy ngầm không hỏi: KHÔNG (giữ record-only). Nhắc theo ngưỡng là confirm-gated, không phải tự chạy.
- Nhắc cuối period/closure: KHÔNG ở v1 (chỉ nhắc theo ngưỡng sau retro-thường).
- Gộp xuyên ngăn (project + workflow trực tiếp): KHÔNG. Phải reclassify về cùng ngăn trước, rồi mới gộp.
- Đụng archive (`ported`/`obsolete` cũ): KHÔNG. Consolidate chỉ làm việc trên bài còn hiệu lực.
- Tự động port workflow shared về repo gốc: vẫn thủ công (record-only), đúng tinh thần okr-retro.
