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

## Quality Gate

Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.

Ví dụ áp dụng cho mode `new` / `update-objective`:
- "Đủ cụ thể?" → "tăng doanh thu" FAIL vì chưa nói kênh nào, sản phẩm nào, bao nhiêu %.
- "Giả định ẩn?" → "launch Q4" nhưng chưa nói capacity có đủ không.
- "Mâu thuẫn?" → muốn 12 bài/tháng nhưng capacity Solo Profile chỉ 8h/tuần.

Mode `update-resource` áp dụng câu 1 chính (tài nguyên có cụ thể đo được không: số giờ/tuần, skill rõ ràng, tool đã tồn tại).

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

### Phase 0a: Đọc inbox làm context (nếu có)

Trước khi hỏi loại mục tiêu, kiểm tra `.okr/inbox/*.md`:

- Nếu thư mục `.okr/inbox/` chưa có hoặc rỗng → skip Phase 0a, sang Phase 1.
- Nếu có items → đọc tất cả file (frontmatter + body), nhóm theo `type`:
  - `action` items: gợi ý cho user đây có thể là việc cần làm cho objective sắp tạo.
  - `resource` items: tool/tài liệu sẵn có, dùng làm input cho Phase 5 Resource.
  - `blocker` items: rủi ro/thiếu hụt, dùng làm input cho Phase 5 mục Rủi ro.
  - `thought` items: ý tưởng nền, có thể chuyển thành KR hoặc actions.

Hiển thị block context:

```
Inbox sẵn có (5 items, captured trước khi tạo objective)
- 2 action: "Viết blog post về X", "Setup analytics tracking"
- 1 resource: "Library Y hỗ trợ chart"
- 1 blocker: "Server staging không stable"
- 1 thought: "Có thể test pricing model 2 tier?"

Mình sẽ dùng các items này làm context khi đề xuất KR (Phase 3),
actions (qua /okr plan sau), và resources (Phase 5). Sau khi
objective tạo xong, mình sẽ hỏi map từng item vào KR phù hợp.
Tiếp tục Phase 1? (y/sửa context/skip context)
```

User trả lời:
- `y` → đi tiếp Phase 1, agent giữ items làm context nội bộ.
- `sửa context` → user bổ sung/loại trừ items nào.
- `skip context` → đi tiếp Phase 1 nhưng không dùng items (vẫn giữ inbox cho Phase 8).

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
4. Hiển thị: "Đã tạo objective + resources. Tiếp tục Phase 8 nếu có inbox sẵn, hoặc chạy `/okr` để lập plan."

### Phase 8: Post-init — gợi ý map inbox vào KR (chỉ chạy nếu Phase 0a đã đọc inbox)

Skip nếu Phase 0 trả về rỗng. Nếu có items:

Với mỗi inbox item có `related_kr: null`, agent đề xuất KR phù hợp dựa trên KR vừa tạo + nội dung item:

```
Map inbox items vào KR (5 items pending)
| # | Type     | Title                          | Đề xuất related_kr | Lý do                          |
|---|----------|--------------------------------|--------------------|--------------------------------|
| 1 | action   | Viết blog post về X            | KR1                | KR1 về content marketing       |
| 2 | action   | Setup analytics tracking       | KR2                | KR2 cần đo conversion          |
| 3 | resource | Library Y hỗ trợ chart         | (không cần map)    | Resource, không link KR        |
| 4 | blocker  | Server staging không stable    | KR2                | Blocker A007 thuộc KR2         |
| 5 | thought  | Có thể test pricing 2 tier?    | KR2                | Liên quan revenue              |

Apply gợi ý? (vd: "1,2,4,5 apply / 3 skip" / all / sửa N: KR<X> / huỷ)
```

User trả lời:
- `<N> apply`: ghi `related_kr: KR<X>` vào frontmatter inbox item.
- `<N> skip`: giữ `related_kr: null`.
- `all`: apply hết.
- `sửa N: KR<X>`: override gợi ý.
- `huỷ`: bỏ hết, không apply.

Sau khi apply: hiển thị "Đã map N items. Chạy `/okr` để lập plan và xử lý inbox khi track."

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

### Phase 4: Re-validate SMART (BẮT BUỘC trước CONFIRM)

Trước khi sang Impact Check, agent re-validate SMART cho mọi KR/KI có thay đổi (target, baseline, period). Tiêu chí xem `references/okr-guide.md`:

- **S**pecific: KR mô tả rõ chỉ số nào, sản phẩm/kênh nào.
- **M**easurable: có baseline + target số (Project) hoặc ngưỡng (Ongoing).
- **A**chievable: target stretch nhưng khả thi với capacity hiện tại.
- **R**elevant: KR/KI vẫn phục vụ Objective.
- **T**ime-bound: có deadline (Project) hoặc review_cycle (Ongoing).

Nếu KR/KI thay đổi fail tiêu chí nào → ghi nhớ để liệt kê trong bảng confirm Phase 6. KHÔNG block, user vẫn quyết.

### Phase 5: Impact Check (BẮT BUỘC trước CONFIRM)

Quét `.okr/plan.md` (frontmatter milestones + body Practices nếu Ongoing) và `.okr/actions/*.md` (frontmatter; **không đệ quy** archive). Với mỗi field thay đổi trong Phase 3, xác định tác động cụ thể:

| Field thay đổi | Cách quét tác động |
|----------------|--------------------|
| `end_date` rút ngắn | Liệt kê actions có `due_date > end_date_mới`. Liệt kê milestones có `target_date > end_date_mới`. |
| `end_date` extend | Note "Period mở rộng N ngày, có thể thêm milestone/actions mới qua `/okr plan update`". |
| `start_date` lùi | Liệt kê actions/milestones có ngày sớm hơn `start_date_mới`. |
| `KR<N>.target` đổi | Liệt kê actions có `key_result: KR<N>` (count + IDs). User cần cân nhắc thêm/bỏ actions nếu target thay đổi nhiều (>20% so với cũ). |
| `KR<N>.baseline` đổi | Cảnh báo recompute % progress. Action không bị ảnh hưởng trực tiếp. |
| Xoá KR<N> | Liệt kê actions có `key_result: KR<N>`. Đề xuất reassign sang KR khác hoặc xoá. |
| Đổi `objective` text / WHY | Note "Cân nhắc review lại practices và actions xem có còn align không". |
| Đổi `review_cycle` (Ongoing) | Note "Track sẽ chuyển sang chu kỳ mới từ lần check-in tới". |
| Đổi `status` | Nếu chuyển sang `completed/cancelled/archived` → liệt kê actions chưa done + đề xuất archive cùng. |

Hiển thị block tác động:

```
Tác động sang plan + actions
- end_date 2026-12-31 → 2026-11-30 (rút ngắn 31 ngày):
  ⚠️ 3 actions due_date sau 2026-11-30: A007 (2026-12-15), A009 (2026-12-20), A011 (2026-12-28)
  ⚠️ 1 milestone target_date sau: M3 (2026-12-15)
  Đề xuất: chạy `/okr plan update` để dời actions/milestone, hoặc giảm scope.
- KR2.target 200M → 250M (+25%):
  3 actions thuộc KR2: A005, A006, A013. Cân nhắc thêm 1-2 actions marketing.
```

KHÔNG block. User vẫn có thể confirm và xử lý plan sau qua `/okr plan update`. Mục tiêu là surface tác động để user không quên.

### Phase 6: CONFIRM diff

**Pre-confirmed flow (T4b):** Nếu payload có `context.pre_confirmed: true` (user đã confirm tại track Bước 4 với full all-changes preview), SKIP ask "Xác nhận? (y/sửa/huỷ)" và đi thẳng Phase 7 ghi file. Vẫn HIỂN THỊ block "Lý do điều chỉnh" + bảng diff + cảnh báo SMART + tác động (Phase 5) để user trace, kèm 1 dòng cuối: `(Đã được confirm tại track Bước 4. Ghi file ngay.)`.

Default flow (không pre_confirmed): nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:

```
Lý do điều chỉnh (từ track deep)
  Market shift Q4 (root cause Bước 2): tăng trưởng ngành chậm 30%,
  target 50 không khả thi.
  Source: log/reviews/2026-12-01.md
```

Quy tắc reason display:
- Reason rỗng hoặc không có → KHÔNG render block.
- Plain text 1-3 câu, không markdown.
- `source_review` cùng block với reason.

Quy tắc pre_confirmed:
- `pre_confirmed: true` chỉ áp dụng khi track Bước 4 đã hiển thị **all-changes diff** + user reply "y" (xem `okr-track/SKILL.md` Phase 4b Bước 4-5).
- Pre-confirmed bypass CHỈ skip ask "y/sửa/huỷ". Vẫn hiển thị Impact Check (Phase 5) + ghi log + báo cáo bình thường.

Sau đó luôn hiển thị bảng:

```
Thay đổi sắp áp dụng (objective.md)
| Field   | Trước              | Sau                |
|---------|--------------------|--------------------|
| KR2     | MRR 200M           | MRR 250M           |
| End     | 2026-12-31         | 2027-01-15         |
| Status  | active             | active (giữ)       |

Cảnh báo SMART (KR/KI thay đổi)
- KR2 (MRR 200M > 250M): ⚠️ Achievable? Tốc độ MRR hiện tại
  ~10M/tháng × 4 tháng = 40M, gap target mới 50M. Hơi stretch.
- End 2027-01-15: Time-bound OK, nhưng cross năm tài chính. Cân
  nhắc đặt thêm milestone Q4 closure trước.

Tác động sang plan + actions (từ Phase 5)
- KR2.target 200M → 250M: 3 actions thuộc KR2. Cân nhắc thêm
  1-2 actions marketing qua `/okr plan update` sau.

Xác nhận? (y / sửa / huỷ / bỏ qua cảnh báo)
```

### Phase 7: Ghi file

Ghi đè `objective.md`. Hiển thị: "Đã update. Chạy `/okr plan update` để áp dụng tác động sang actions/milestones (Phase 5 Impact Check đã liệt kê)."

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
