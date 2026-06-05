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
