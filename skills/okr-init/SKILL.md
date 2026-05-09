---
name: okr-init
description: "Khởi tạo HOẶC cập nhật mục tiêu OKR + resource. Skill này chịu trách nhiệm cả 2 SOT: `objective.md` và `resources.md`. Hỗ trợ 3 sub-mode: `new` (tạo mới khi chưa có .okr/), `update-objective` (sửa mục tiêu/KR), `update-resource` (sửa người/tool/ngân sách/PIC). Skill được kích hoạt từ `/okr` khi `.okr/` chưa có HOẶC khi user nhắc tới mục tiêu/tài nguyên/PIC. KHÔNG gọi trực tiếp trừ khi user gõ `/okr init` hoặc `/okr-init`."
---

# okr-init: Khởi tạo + cập nhật objective & resource

Skill duy nhất quản lý 2 SOT khởi tạo: `objective.md` và `resources.md`. Hỗ trợ tạo mới và cập nhật.

## Nguyên tắc

- Hỏi từng câu một, không hỏi hàng loạt.
- BẮT BUỘC có phase confirm bảng trước khi ghi/sửa file.
- Tận dụng context user truyền từ `/okr`. Vd: "/okr thêm Dũng vào dự án QA 80%" → vào `update-resource` ngay với context Dũng.
- Đề xuất + lý do, để user quyết. Không tự áp đặt.

## Flow

### Phase 0: Detect mode

| State | Mode |
|-------|------|
| `.okr/` chưa có HOẶC `objective.md` thiếu | `new` |
| Có objective.md + user nhắc tới objective/KR/period | `update-objective` |
| Có objective.md + user nhắc tới người/tool/PIC/ngân sách | `update-resource` |
| User chọn explicit (vd "/okr init update-objective") | theo lựa chọn |
| Mơ hồ | hỏi user: "Tạo mới (ghi đè) / Sửa mục tiêu / Sửa resource?" |

---

## Mode NEW: tạo `.okr/` từ đầu

### Phase 1: Loại mục tiêu

Hỏi: **Project** (có deadline cụ thể) hay **Habit** (lặp lại không deadline).

### Phase 2: Thu thập Objective (hỏi từng câu)

- **WHY**: Tại sao mục tiêu này quan trọng?
- **WHAT**: Kết quả cụ thể mong đợi?
- **HOW**: Cách tiếp cận tổng quan?
- **Khung thời gian**: start_date, end_date (Project) hoặc frequency (Habit).

### Phase 3: Đề xuất Key Results / Key Indicators

Đọc `references/okr-guide.md` để check SMART. Mỗi KR/KI:
- Đo được (số liệu, không mơ hồ)
- Có baseline + target
- Khả thi nhưng thách thức

Đề xuất 2-4 KR/KI, hỏi feedback từng cái.

### Phase 4: CONFIRM Objective (BẮT BUỘC)

```
Tóm tắt Objective
| Field        | Value                                |
|--------------|--------------------------------------|
| Type         | project                              |
| Objective    | Tăng doanh thu sản phẩm A 30% Q4-2026|
| Period       | 2026-10-01 > 2026-12-31              |
| WHY          | [tóm tắt 1 câu]                      |
| KR1          | DAU: 1k > 5k (target)                |
| KR2          | MRR: 100M > 200M VND                 |

Xác nhận? (y / sửa <field>: <giá trị mới> / huỷ)
```

Lặp đến khi user `y`.

### Phase 5: Thu thập Resource

Hỏi tuần tự:
1. **Nhân sự**: Ai tham gia? Vai trò? Khả dụng (% thời gian)?
2. **Công cụ & Tài liệu**: Hệ thống/tool/doc nào hiện có hoặc cần?
3. **Ngân sách** (nếu Project): Có ngân sách không? Bao nhiêu?
4. **Thiếu hụt**: Có rủi ro/thiếu hụt nào nhận biết được không?

User skip được (`không có` / `để sau`).

### Phase 6: CONFIRM Resource (BẮT BUỘC)

```
Tóm tắt Resource
| Mục            | Giá trị                                  |
|----------------|------------------------------------------|
| Người          | An (PM, 100%), Bình (Dev, 50%)           |
| Công cụ        | Notion, GitHub, Figma                    |
| Tài liệu hiện có| brief.pdf, market-research.docx         |
| Ngân sách      | 50M VND                                  |
| Rủi ro         | Bình kiêm 2 dự án                        |

Xác nhận? (y / sửa / huỷ)
```

### Phase 7: Ghi file

1. Tạo `.okr/`.
2. Ghi `.okr/objective.md` theo schema.
3. Ghi `.okr/resources.md` theo schema (giữ section header dù rỗng).
4. Hiển thị: "Đã tạo. Chạy `/okr` để lập plan."

---

## Mode UPDATE-OBJECTIVE: sửa objective/KR

### Phase 1: Hiển thị state hiện tại

Đọc `objective.md`, hiển thị bảng tóm tắt như Phase 4 mode `new`.

### Phase 2: Hỏi user muốn sửa gì

Menu:
1. Sửa Objective text / WHY
2. Sửa period (start/end)
3. Thêm/sửa/xoá KR
4. Đổi status (active/paused/completed/cancelled)

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn, hỏi chi tiết.

### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr` để check tác động sang plan."

---

## Mode UPDATE-RESOURCE: sửa resource

### Phase 1: Hiển thị state hiện tại

Đọc `resources.md` + frontmatter `actions/*.md` (nếu có). Hiển thị:

```
Resource hiện tại
| Mục              | Số lượng | Chi tiết                          |
|------------------|----------|-----------------------------------|
| Người            | 3        | An (PM), Bình (Dev), Chi (Design) |
| Công cụ          | 4        | Notion, GitHub, Figma, Slack      |
| Ngân sách        | 50M VND  | đã chi: 12M                       |
| Actions chưa PIC | 2        | A007, A011                        |
| Cảnh báo         | 1        | Bình khả dụng <50%, có 5 actions  |
```

### Phase 2: Hỏi user muốn update gì

Menu:
1. Thêm/sửa/xoá người
2. Thêm/sửa/xoá công cụ, tài liệu
3. Update ngân sách
4. Mapping PIC vào actions
5. Update rủi ro/thiếu hụt
6. Re-check xung đột

User chọn nhiều cùng lúc được (vd "1, 4").

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn user. Vd nếu chọn 4:
- Liệt kê actions chưa PIC.
- Đề xuất PIC dựa trên skill match + khả dụng.
- User confirm từng cái.

### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng
| Loại          | Trước              | Sau                  |
|---------------|--------------------|----------------------|
| Thêm người    | -                  | Dũng (QA, 80%)       |
| Sửa PIC       | A007: unassigned   | A007: Dũng           |
| Sửa khả dụng  | Bình: 100%         | Bình: 50%            |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Update field `pic` trong frontmatter `actions/*.md` (nếu có mapping mới).
3. Check xung đột (xem section "Phát hiện xung đột" trong `references/data-format.md`):
   - PIC khả dụng <50% nhưng có >N actions cùng deadline → cảnh báo
   - Action có deadline mà PIC chưa khả dụng → cảnh báo
4. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."

---

## Schema

Đọc `references/data-format.md` cho schema `objective.md` + `resources.md` + rule phát hiện xung đột.
Đọc `references/okr-guide.md` cho check SMART KR.

## Quy tắc chung

- Hỏi 1 câu/lần.
- KHÔNG ghi file trước phase confirm.
- Mode `new`: ghi cả 2 file (objective + resources). Resource trống vẫn ghi `resources.md` rỗng có header.
- Mode `update-*`: ghi đè SOT. Update PIC sync cả `resources.md` + frontmatter `actions/*.md`.
- KR không SMART → chỉ rõ thiếu tiêu chí nào + gợi ý sửa.
- Không tạo `plan.md` hay action files. Plan thuộc `okr-plan`.
- Không sửa action status. Đó là việc của `okr-track`.
