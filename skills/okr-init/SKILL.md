---
name: okr-init
description: "Sub-skill của /okr. Khởi tạo hoặc cập nhật mục tiêu OKR (objective.md) và resource (resources.md). 3 mode: new, update-objective, update-resource. Được kích hoạt từ orchestrator /okr, KHÔNG gọi trực tiếp trừ khi user gõ /okr-init."
---

# okr-init: Khởi tạo + cập nhật objective & resource

Skill duy nhất quản lý 2 SOT khởi tạo: `objective.md` và `resources.md`. Hỗ trợ tạo mới và cập nhật.

## Nguyên tắc

- Hỏi từng câu một, không hỏi hàng loạt.
- BẮT BUỘC có phase confirm bảng trước khi ghi/sửa file.
- Tận dụng context user truyền từ `/okr`. Vd: "/okr thêm Dũng vào dự án QA 80%" → vào `update-resource` ngay với context Dũng.
- Đề xuất + lý do, để user quyết. Không tự áp đặt.

## Quality Gate: đánh giá câu trả lời trước khi đi tiếp

Mỗi khi user trả lời, agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành KR/KI đo được hoặc action có deliverable không? Vd: "tăng doanh thu" → FAIL (chưa nói kênh nào, sản phẩm nào, bao nhiêu %).
2. **Giả định ẩn?** User bỏ qua điều kiện quan trọng nào không? Vd: "launch Q4" nhưng chưa nói team có ai.
3. **Mâu thuẫn?** Câu trả lời có xung đột với context trước không? Vd: muốn 12 bài/tháng nhưng chỉ có 1 writer part-time.

**Hành vi theo kết quả:**

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp câu hỏi kế |
| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |
| User trả lời "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm PHẢI nhắc lại các field TBD |
| User tỏ ra sốt ruột | Giảm độ sâu, chỉ giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn |
| User paste từ doc có sẵn (brief, PRD) | Tóm tắt lại bằng lời mình, hỏi "tôi hiểu đúng chưa?" thay vì nhận nguyên xi |

## Flow

### Phase 0: Detect mode

| State | Mode |
|-------|------|
| `.okr/` chưa có HOẶC `objective.md` thiếu | `new` |
| Có objective.md + user nhắc tới objective/KR/KI/period | `update-objective` |
| Có objective.md + user nhắc tới capacity/skill/tool/ngân sách | `update-resource` |
| User chọn explicit (vd "/okr init update-objective") | theo lựa chọn |
| Mơ hồ | hỏi user: "Tạo mới (ghi đè) / Sửa mục tiêu / Sửa resource?" |

---

## Mode NEW: tạo `.okr/` từ đầu

### Phase 1: Loại mục tiêu

Hỏi: **Project** (có deadline, đạt target rồi kết thúc) hay **Ongoing** (duy trì liên tục, không có điểm kết thúc, giống lĩnh vực cần chăm sóc).

Ví dụ giúp user phân biệt:
- Project: "Ra mắt MVP trước 30/7", "Viết xong sách trước Q4"
- Ongoing: "Duy trì sức khoẻ thể chất", "Chất lượng code", "Tài chính cá nhân"

### Phase 2: Thu thập Objective (hỏi từng câu + đào sâu)

Hỏi từng câu. Sau mỗi câu trả lời, chạy Quality Gate. Nếu fail, dùng kỹ thuật đào sâu tương ứng.

| Câu hỏi | Tiêu chí "đạt" | Kỹ thuật đào sâu khi chưa đạt |
|---------|----------------|-------------------------------|
| **WHY**: Tại sao mục tiêu này quan trọng? | Chạm được động lực gốc (không còn hỏi "tại sao?" tiếp được) | Dùng "5 Whys": hỏi "tại sao?" lặp lại cho đến khi chạm lý do thật. Vd: "Tăng doanh thu" → "Tại sao?" → "Để có lãi" → "Đang lỗ à? Lỗ bao nhiêu?" |
| **WHAT**: Kết quả cụ thể mong đợi? | User mô tả được "done trông thế nào" (Project) hoặc "khoẻ trông thế nào" (Ongoing) bằng 1 câu cụ thể | Hỏi: "Khi xong, chuyện gì khác đi so với bây giờ?" (Project) hoặc "Lĩnh vực này đang khoẻ khi nào? Tiêu chí nào cho biết?" (Ongoing) |
| **HOW**: Cách tiếp cận tổng quan? | User chọn được hướng đi, không nói chung chung | Đưa 2-3 hướng kèm tradeoff. Vd: "Có 3 cách: A (nhanh, rủi ro cao), B (chắc, tốn resource), C (thử nghiệm nhỏ trước). Nghiêng hướng nào?" |
| **Khung thời gian** (Project): start_date, end_date | Có mốc cụ thể, không "càng sớm càng tốt" | Hỏi: "Có event nào buộc phải xong trước không?" (deadline cứng) hoặc "Nếu không có deadline, bao lâu là hợp lý?" |
| **Chu kỳ review** (Ongoing): review_cycle | User chọn weekly/biweekly/monthly | Hỏi: "Bạn muốn review lĩnh vực này bao lâu 1 lần? Tuần / 2 tuần / tháng?" |

### Phase 3: Đề xuất Key Results (Project) hoặc Key Indicators (Ongoing)

Đọc `references/okr-guide.md` để check tiêu chí.

**Với Project (Key Results)**:
- Đo được (số liệu, không mơ hồ)
- Có baseline + target
- Khả thi nhưng thách thức

Đề xuất 2-4 KR, hỏi feedback từng cái.

**Với Ongoing (Key Indicators)**:
- Đo được (số hoặc tần suất)
- Có ngưỡng tối thiểu rõ ràng
- Phản ánh trực tiếp sức khoẻ lĩnh vực

Đề xuất 2-4 KI, hỏi feedback từng cái.

**Challenge KR/KI (bắt buộc với mỗi cái):**

Sau khi user đồng ý, agent phải challenge:

Với KR (Project):
1. **Đo bằng gì?** Data source ở đâu? Ai đo? Bao lâu đo 1 lần?
2. **Kiểm soát được không?** KR phụ thuộc hoàn toàn vào team hay cần bên ngoài? Nếu phụ thuộc ngoài → cảnh báo rủi ro + hỏi có backup plan.
3. **Baseline chính xác chưa?** Số hiện tại là bao nhiêu? Lấy từ đâu? Đã verify chưa?
4. **Target có stretch đủ?** So sánh với tốc độ tăng trưởng hiện tại. Vd: "Hiện 1k DAU, target 5k trong 3 tháng = tăng 5x. Growth rate hiện tại bao nhiêu?"

Với KI (Ongoing):
1. **Đo bằng gì?** Data thu thập thế nào? Tự ghi hay tự động?
2. **Ngưỡng có thực tế?** So với thực tế hiện tại. Vd: "Hiện tập 1 lần/tuần, đặt ngưỡng 5 lần/tuần có khả thi không?"
3. **Warning/Critical phân biệt rõ?** Khi nào chỉ cần lưu ý, khi nào cần hành động gấp?

Không cần hỏi tất cả nếu đã rõ (Quality Gate pass). Chỉ hỏi câu nào agent chưa tự trả lời được.

### Phase 4: CONFIRM Objective (BẮT BUỘC, kèm đánh giá)

Bảng confirm PHẢI kèm phần "Đánh giá nhanh" do agent tự phân tích. Không chỉ echo data.

**Với Project:**

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

Đánh giá nhanh
  Điểm mạnh:
  - [vd: KR1 đo được rõ, data source có sẵn (GA4)]
  - [vd: Timeline 3 tháng khớp với chu kỳ kinh doanh]
  Cần lưu ý:
  - [vd: KR2 phụ thuộc đội sales, ngoài kiểm soát trực tiếp]
  - [vd: ⚠️ WHY chưa rõ động lực gốc (đánh dấu TBD)]

Xác nhận? (y / sửa <field>: <giá trị mới> / huỷ)
```

**Với Ongoing:**

```
Tóm tắt Objective
| Field         | Value                                |
|---------------|--------------------------------------|
| Type          | ongoing                              |
| Objective     | Duy trì sức khoẻ thể chất            |
| Review cycle  | weekly                               |
| WHY           | [tóm tắt 1 câu]                      |
| KI1           | Tập thể dục ≥3 lần/tuần (current: 1) |
| KI2           | Ngủ ≥7 giờ/đêm (current: 5.5)        |

Đánh giá nhanh
  Điểm mạnh:
  - [vd: KI đo được rõ, dễ tự track hàng ngày]
  Cần lưu ý:
  - [vd: KI1 ngưỡng 3 nhưng current chỉ 1, gap lớn → nên bắt đầu từ 2?]
  - [vd: KI2 phụ thuộc nhiều yếu tố ngoài kiểm soát (công việc, con nhỏ)]

Xác nhận? (y / sửa <field>: <giá trị mới> / huỷ)
```

Quy tắc đánh giá:
- Liệt kê 1-3 điểm mạnh (để user yên tâm phần đã tốt).
- Liệt kê mọi điểm cần lưu ý: field TBD, KI/KR phụ thuộc ngoài, mâu thuẫn tiềm ẩn.
- Nếu có field TBD → liệt kê rõ, hỏi user muốn bổ sung ngay hay ghi nhận rủi ro.

Lặp đến khi user `y`.

### Phase 5: Thu thập Resource (hỏi tuần tự + cross-check)

Skill phục vụ persona **solo only** (1 user, 1 objective). KHÔNG còn nhánh team. Hỏi tuần tự:

1. **Solo Profile**:
   - **Capacity**: Bạn dành bao nhiêu giờ/tuần cho mục tiêu này? (vd: 10 giờ/tuần, mỗi tối 1 tiếng)
   - **Skills liên quan**: Bạn đã có sẵn kỹ năng nào phục vụ mục tiêu này? (vd: Python, viết content, design Figma). Mục đích: agent biết tự tin đề xuất action loại nào, và cảnh báo khi action cần skill chưa có.
2. **Công cụ**: Danh sách công cụ sẽ sử dụng, khi nào dùng, dùng để làm gì, resource liên quan (URL, account).
3. **Tài liệu / Knowledge Base**: Các tài liệu, hệ thống lưu trữ hiện có hoặc cần thiết (Link, folder, file), status (có sẵn/cần tạo/đang thiếu).
4. **Ngân sách** (nếu Project hoặc cần đầu tư): Có ngân sách không? Bao nhiêu?
5. **Thiếu hụt**: Có rủi ro/thiếu hụt nào nhận biết được không? (vd: skill còn thiếu, tool chưa mua, deadline ngoài tầm kiểm soát)

User skip được (`không có` / `để sau`), nhưng field skip đánh dấu `⚠️ TBD`.

**Cross-check capacity vs. scope (bắt buộc sau khi thu thập xong):**

Agent tự tính toán sơ bộ trước khi sang Phase 6:
- Tổng available capacity = `capacity h/tuần × số tuần đến end_date` (Project) hoặc `capacity h/tuần × 4` (Ongoing, dùng 1 tháng làm horizon đánh giá)
- Ước tính scope từ KR/KI đã define (Phase 3): mỗi KR cần khoảng bao nhiêu giờ?
- So sánh: capacity có đủ cho scope không?

Nếu lệch rõ rệt → cảnh báo user ngay, hỏi:
- "Scope cần khoảng X giờ, capacity hiện có Y giờ. Cần thu hẹp scope, extend deadline, hay tăng capacity?"

Không cần chính xác tuyệt đối. Mục tiêu là phát hiện bất hợp lý lớn (vd: 10 giờ/tuần làm việc của 40 giờ/tuần).

### Phase 6: CONFIRM Resource (BẮT BUỘC, kèm đánh giá)

```
Tóm tắt Resource
| Mục            | Giá trị                                  |
|----------------|------------------------------------------|
| Solo Profile   | Tuấn, 10h/tuần, skills: Python, viết     |
| Công cụ        | Notion (Task), GitHub, Figma             |
| Tài liệu/KB    | brief.pdf, thư mục Drive                 |
| Ngân sách      | 5M VND                                   |
| Rủi ro         | Skill design Figma còn yếu               |

Đánh giá nhanh
  Capacity: 10h/tuần × 12 tuần = 120 giờ
  Scope ước tính: [dựa trên KR/KI] ≈ 150 giờ
  Fit: ⚠️ hơi thiếu, cần buffer hoặc thu hẹp scope
  Rủi ro chính:
  - Skill Figma còn yếu, cân nhắc dùng template/outsource phần design
  - ⚠️ Ngân sách chưa rõ (TBD)

Xác nhận? (y / sửa / huỷ)
```

### Phase 7: Ghi file

1. Tạo `.okr/`.
2. Ghi `.okr/objective.md` theo schema.
3. Ghi `.okr/resources.md` theo schema (giữ section header dù rỗng).
4. Hiển thị: "Đã tạo. Chạy `/okr` để lập plan."

---

## Mode UPDATE-OBJECTIVE: sửa objective/KR/KI

### Phase 1: Hiển thị state hiện tại

Đọc `objective.md`, hiển thị bảng tóm tắt như Phase 4 mode `new`.

### Phase 2: Hỏi user muốn sửa gì

Menu:
1. Sửa Objective text / WHY
2. Sửa period (start/end) hoặc review_cycle
3. Thêm/sửa/xoá KR (Project) hoặc KI (Ongoing)
4. Đổi status (active/paused/completed/cancelled/archived)

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

Đọc `resources.md`. Hiển thị:

```
Resource hiện tại
| Mục            | Giá trị                                |
|----------------|----------------------------------------|
| Solo Profile   | Tuấn, 10h/tuần, skills: Python, viết   |
| Công cụ        | 4 (Notion, GitHub, Figma, Slack)       |
| Tài liệu/KB    | 2 (brief.pdf, Drive folder)            |
| Ngân sách      | 5M VND (đã chi: 1.2M)                  |
| Rủi ro         | 1 (Skill Figma còn yếu)                |
```

> **Legacy migrate**: Nếu file vẫn còn section `## Nhân sự (Vai trò & Trách nhiệm)` schema cũ, agent tự convert: lấy dòng đầu (hoặc dòng có khả dụng cao nhất) thành Solo Profile, drop các cột "Liên lạc", "Người quản lý", "Khả dụng %", "Actions". Thông báo user "Đã migrate schema cũ sang Solo Profile."

### Phase 2: Hỏi user muốn update gì

Menu:
1. Sửa Solo Profile (capacity h/tuần, skills)
2. Thêm/sửa/xoá công cụ
3. Thêm/sửa/xoá tài liệu / knowledge base
4. Update ngân sách
5. Update rủi ro/thiếu hụt

User chọn nhiều cùng lúc được (vd "1, 4").

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn user. Vd nếu chọn 1:
- Hỏi capacity mới: "Tuần này bạn còn dành được X h/tuần không?"
- Hỏi có thêm skill mới sau quá trình thực thi (vd học được kỹ năng mới)?

### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng
| Loại                  | Trước              | Sau                  |
|-----------------------|--------------------|----------------------|
| Sửa capacity          | 10h/tuần           | 6h/tuần              |
| Thêm skill            | Python, viết       | Python, viết, SQL    |
| Sửa ngân sách         | 5M VND             | 8M VND               |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Cảnh báo cross-check:
   - Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done → cảnh báo "Capacity giảm, scope có thể không kịp deadline. Cân nhắc dời actions sang sau hoặc giảm scope qua `/okr plan update`."
   - Skill mới thêm có thể unlock action TBD trước đó (vd "Skill SQL mới: action A007 trước đây skip vì thiếu skill có thể làm lại").
3. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."

---

## Schema

Đọc `references/data-format.md` cho schema `objective.md` + `resources.md` + rule phát hiện xung đột.
Đọc `references/okr-guide.md` cho check SMART KR và KI guidelines.

## Quy tắc chung

- Hỏi 1 câu/lần.
- KHÔNG ghi file trước phase confirm.
- Mode `new`: ghi cả 2 file (objective + resources). Resource trống vẫn ghi `resources.md` rỗng có header.
- Mode `update-*`: ghi đè SOT. Solo Profile chỉ 1 user, không cần sync `pic` vào `actions/*.md` (mặc định `self`).
- KR không SMART → chỉ rõ thiếu tiêu chí nào + gợi ý sửa.
- KI không đo được hoặc thiếu ngưỡng → chỉ rõ + gợi ý sửa.
- Không tạo `plan.md` hay action files. Plan thuộc `okr-plan`.
- Không sửa action status. Đó là việc của `okr-track`.
