# Flow: Init New (tạo objective + resource từ đầu)

> Self-contained. Orchestrator chạy inline, mang theo context đã có.

## Nguyên tắc

- Hỏi từng câu một, không hỏi hàng loạt.
- BẮT BUỘC có phase confirm bảng trước khi ghi/sửa file.
- Tận dụng context user truyền từ orchestrator. Vd: "thêm Dũng vào dự án QA 80%" → vào `update-resource` ngay.
- Đề xuất + lý do, để user quyết. Không tự áp đặt.

## Quality Gate

Áp dụng 3 câu check trước mỗi follow-up (đọc `quality-gate.md` nếu cần chi tiết):

- "Đủ cụ thể?" → "tăng doanh thu" FAIL vì chưa nói kênh nào, sản phẩm nào, bao nhiêu %.
- "Giả định ẩn?" → "launch Q4" nhưng chưa nói capacity có đủ không.
- "Mâu thuẫn?" → muốn 12 bài/tháng nhưng capacity Solo Profile chỉ 8h/tuần.

## Phase 0: Detect mode

| State | Mode |
|-------|------|
| `.okr/` chưa có HOẶC `objective.md` thiếu | `new` |
| Có objective.md + user nhắc tới objective/KR/KI/period | `update-objective` (xem `flow-init-update-objective.md`) |
| Có objective.md + user nhắc tới capacity/skill/tool/ngân sách | `update-resource` (xem `flow-init-update-resource.md`) |
| User chọn explicit | theo lựa chọn |
| Mơ hồ | hỏi user: "Tạo mới (ghi đè) / Sửa mục tiêu / Sửa resource?" |

**Cảnh báo ghi đè**: Nếu mode `new` nhưng `objective.md` đã tồn tại:
1. Hiển thị: "Objective đã tồn tại: [tên]. Ghi đè sẽ mất KR/KI hiện tại + plan liên kết có thể bất khớp."
2. Hỏi: "(ghi đè / chuyển sang update-objective / huỷ)"

## Phase 0a: Đọc inbox làm context (nếu có)

Kiểm tra `.okr/inbox/*.md`:

- Rỗng → skip, sang Phase 1.
- Có items → đọc tất cả (frontmatter + body), nhóm theo `type`:
  - `action`: việc cần làm cho objective sắp tạo.
  - `resource`: tool/tài liệu sẵn có, input cho Phase 5.
  - `blocker`: rủi ro/thiếu hụt, input cho Phase 5 mục Rủi ro.
  - `thought`: ý tưởng nền, có thể chuyển thành KR hoặc actions.

Hiển thị block context, hỏi user: `y` / `sửa context` / `skip context`.

## Phase 1: Loại mục tiêu

Hỏi: **Project** (có deadline, đạt target rồi kết thúc) hay **Ongoing** (duy trì liên tục, không điểm kết thúc).

Ví dụ:
- Project: "Ra mắt MVP trước 30/7", "Viết xong sách trước Q4"
- Ongoing: "Duy trì sức khoẻ thể chất", "Chất lượng code"

## Phase 2: Thu thập Objective (hỏi từng câu + đào sâu)

Hỏi từng câu. Sau mỗi câu, chạy Quality Gate. Fail → dùng kỹ thuật đào sâu.

| Câu hỏi | Tiêu chí "đạt" | Kỹ thuật đào sâu |
|----------|----------------|-------------------|
| **WHY**: Tại sao quan trọng? | Chạm động lực gốc (không hỏi "tại sao?" tiếp được) | 5 Whys |
| **WHAT**: Kết quả cụ thể? | Mô tả "done trông thế nào" bằng 1 câu cụ thể | "Khi xong, chuyện gì khác đi?" |
| **HOW**: Cách tiếp cận? | Chọn được hướng đi, không chung chung | Đưa 2-3 hướng kèm tradeoff |
| **Khung thời gian** (Project) | Mốc cụ thể | "Có event nào buộc phải xong trước?" |
| **Chu kỳ review** (Ongoing) | weekly/biweekly/monthly | "Review bao lâu 1 lần?" |

## Phase 3: Đề xuất Key Results / Key Indicators

Đọc `okr-guide.md` để check tiêu chí.

**Project (KR)**: đo được, có baseline + target, khả thi nhưng thách thức. Đề xuất 2-4 KR.

**Ongoing (KI)**: đo được, có ngưỡng tối thiểu, phản ánh sức khoẻ lĩnh vực. Đề xuất 2-4 KI.

**Challenge KR/KI (bắt buộc mỗi cái):**

KR: Đo bằng gì? Kiểm soát được không? Baseline chính xác? Target stretch đủ?
KI: Đo bằng gì? Ngưỡng thực tế? Warning/Critical phân biệt rõ?

Không cần hỏi tất cả nếu Quality Gate pass. Chỉ hỏi câu agent chưa tự trả lời được.

## Phase 4: CONFIRM Objective (BẮT BUỘC, kèm đánh giá)

Bảng confirm PHẢI kèm "Đánh giá nhanh" do agent tự phân tích.

**Project:**

```
Tóm tắt Objective
| Field   | Value                                |
|---------|--------------------------------------|
| Type    | project                              |
| Objective | [tên]                              |
| Period  | [start] > [end]                      |
| WHY     | [tóm tắt 1 câu]                     |
| KR1     | [metric]: [baseline] > [target]      |

Đánh giá nhanh
  Điểm mạnh: [1-3 điểm]
  Cần lưu ý: [field TBD, phụ thuộc ngoài, mâu thuẫn]

Xác nhận? (y / sửa <field>: <giá trị mới> / huỷ)
```

**Ongoing:** tương tự, thay KR bằng KI, thay Period bằng Review cycle.

Liệt kê mọi điểm cần lưu ý. Lặp đến khi user `y`.

## Phase 5: Thu thập Resource

Solo only (1 user, 1 objective). Hỏi tuần tự:

1. **Solo Profile**: Capacity (giờ/tuần), Skills liên quan
2. **Công cụ**: 6 cột (Tên, Mục đích, Khi nào dùng, Cách dùng, Resource, Ghi chú). Schema xem `data-format-init.md`.
3. **Tài liệu / KB**: 6 cột tương tự.
4. **Ngân sách** (nếu Project hoặc cần đầu tư)
5. **Thiếu hụt**: rủi ro/thiếu hụt nhận biết được

User skip được, field skip đánh dấu `⚠️ TBD`.

**Cross-check capacity vs. scope (bắt buộc):**

- Tổng capacity = `h/tuần × số tuần đến end_date` (Project) hoặc `h/tuần × 4` (Ongoing)
- Ước tính scope từ KR/KI
- Lệch rõ → cảnh báo: "Scope cần X giờ, capacity Y giờ. Thu hẹp scope, extend deadline, hay tăng capacity?"

## Phase 6: CONFIRM Resource (BẮT BUỘC, kèm đánh giá)

```
Tóm tắt Resource
| Mục        | Giá trị                              |
|------------|--------------------------------------|
| Solo Profile | [tên], [capacity], skills: [...]   |
| Công cụ    | [list]                               |
| Ngân sách  | [amount]                             |
| Rủi ro     | [list]                               |

Đánh giá nhanh
  Capacity: [h/tuần × N tuần] = [total]
  Scope ước tính: [X] giờ
  Fit: [✅/⚠️]
  Rủi ro chính: [list]

Xác nhận? (y / sửa / huỷ)
```

## Phase 7: Ghi file

1. Tạo `.okr/`
2. Ghi `.okr/objective.md` theo schema (`data-format-init.md`)
3. Ghi `.okr/resources.md` theo schema (giữ section header dù rỗng)

## Phase 8: Map inbox vào KR (chỉ nếu Phase 0a đọc inbox)

Skip nếu inbox rỗng. Nếu có:

Với mỗi item `related_kr: null`, đề xuất KR phù hợp:

```
Map inbox items vào KR
| # | Type   | Title              | Đề xuất KR | Lý do              |
|---|--------|--------------------|-----------|---------------------|
| 1 | action | Viết blog post X   | KR1       | KR1 về content      |

Apply gợi ý? (N apply / all / sửa N: KR<X> / huỷ)
```

## Schema

- `data-format-init.md`: schema objective.md + resources.md
- `okr-guide.md`: check SMART KR và KI guidelines

## Quy tắc

- Hỏi 1 câu/lần.
- KHÔNG ghi file trước phase confirm.
- Mode `new`: ghi cả 2 file. Resource trống vẫn ghi `resources.md` rỗng có header.
- KR không SMART → chỉ rõ thiếu tiêu chí nào + gợi ý sửa.
- KI không đo được hoặc thiếu ngưỡng → chỉ rõ + gợi ý sửa.
- Không tạo `plan.md` hay action files. Plan thuộc `okr-plan` mode new.
- Không sửa action status. Đó là việc của `okr-track`.
