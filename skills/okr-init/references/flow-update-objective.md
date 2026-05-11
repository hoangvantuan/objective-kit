# Mode UPDATE-OBJECTIVE: sửa objective/KR/KI

### Phase 1: Hiển thị state hiện tại

Đọc `objective.md`, hiển thị bảng tóm tắt như Phase 4 mode `new`.

### Phase 2: Hỏi user muốn sửa gì

Menu:

1. Sửa Objective text / WHY
2. Sửa period (start/end) hoặc review_cycle
3. Thêm/sửa/xoá KR (Project) hoặc KI (Ongoing)
4. Đổi status:
   - Project: active / paused / completed / cancelled
   - Ongoing: active / paused / archived

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

| Field thay đổi               | Cách quét tác động                                                                                                                  |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `end_date` rút ngắn          | Liệt kê actions có `due_date > end_date_mới`. Liệt kê milestones có `target_date > end_date_mới`.                                   |
| `end_date` extend            | Note "Period mở rộng N ngày, có thể thêm milestone/actions mới qua `/okr plan update`".                                             |
| `start_date` lùi             | Liệt kê actions/milestones có ngày sớm hơn `start_date_mới`.                                                                        |
| `KR<N>.target` đổi           | Liệt kê actions có `key_result: KR<N>` (count + IDs). User cần cân nhắc thêm/bỏ actions nếu target thay đổi nhiều (>20% so với cũ). |
| `KR<N>.baseline` đổi         | Cảnh báo recompute % progress. Action không bị ảnh hưởng trực tiếp.                                                                 |
| Xoá KR                       | Liệt kê actions có `key_result: KR<N>`. Đề xuất reassign sang KR khác hoặc xoá.                                                     |
| Đổi `objective` text / WHY   | Note "Cân nhắc review lại practices và actions xem có còn align không".                                                             |
| Đổi `review_cycle` (Ongoing) | Note "Track sẽ chuyển sang chu kỳ mới từ lần check-in tới".                                                                         |
| Đổi `status` → `completed/cancelled/archived` | Liệt kê actions chưa done + đề xuất archive cùng.                                  |
| Đổi `status` → `paused`      | Liệt kê actions status=doing (N actions). Đề xuất: giữ doing (resume sau) hay chuyển pending. Track sẽ không gợi ý update progress khi paused. Inbox vẫn capture được. |
| Đổi `status` → `active` (resume từ paused) | Recompute timeline: `remaining_days = end_date - today`. Nếu remaining < 0 → đề xuất extend deadline. Nếu dương → hiển thị capacity fit mới (effort còn lại vs remaining capacity). |


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

Delegate protocol (pre-confirmed flow + reason display): xem skill `okr/references/delegate-protocol.md`.

Luôn hiển thị bảng diff + cảnh báo:

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
