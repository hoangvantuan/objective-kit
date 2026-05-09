---
name: okr-track
description: "Đánh giá trạng thái dự án + nhận update progress từ user + đề xuất next action. Skill này gộp cả tracking thường ngày lẫn review sâu (lookback). Tự chọn mode `light` (cập nhật progress: KR.current, action.status), `deep` (phân tích root cause + ĐỀ XUẤT điều chỉnh cấu trúc, delegate sang `okr-init`/`okr-plan` để áp dụng), hoặc `closure` (chốt khi mọi action done). Skill được kích hoạt từ `/okr` khi đã có SOT đầy đủ và actions còn mở. KHÔNG gọi trực tiếp trừ khi user gõ `/okr track`."
---

# okr-track: Track + Review hợp nhất

Một skill duy nhất cho 2 use case "đánh giá trạng thái": cập nhật progress nhanh (daily) và review sâu (milestone/period-end). Tự chọn mode dựa trên context.

**Phân vai rõ ràng**:
- `okr-track` ghi đè **progress fields** (KR.current, action.status) và ghi log.
- Thay đổi **cấu trúc** (KR target, action mới, dời deadline, đổi PIC) → delegate sang `okr-init` hoặc `okr-plan` mode `update-*`. Track đề xuất, init/plan áp dụng.

## Điều kiện tiên quyết

- `.okr/objective.md` + `.okr/plan.md` + `.okr/actions/` tồn tại.
- Thiếu → quay lại `/okr` để init/plan.

## Flow

### Phase 1: Đọc state + tính toán

Đọc song song:
- `.okr/objective.md` (KR hiện tại)
- `.okr/plan.md` (milestones)
- frontmatter `.okr/actions/*.md` (status, due_date, pic)
- `.okr/resources.md` (PIC khả dụng)
- File log gần nhất trong `.okr/log/` và `.okr/log/reviews/` (nếu có)

Tính metrics (xem `references/metrics.md`):
- KR: % đạt, trend so với log trước
- Timeline: % thời gian đã dùng vs % tiến độ trung bình
- Actions: tổng / done / doing / blocked / pending
- Tốc độ: actions hoàn thành/tuần
- Highlight: action quá hạn, blocked >X ngày

### Phase 2: Hiển thị dashboard

```
Dashboard: [Tên Objective]
Period: 2026-10-01 > 2026-12-31 (60% thời gian đã dùng)

Key Results
  KR1: ████░░░░░░ 40/100 (40%) > on-track
  KR2: ██░░░░░░░░ 10/50  (20%) ! at-risk (trễ 15%)
  KR3: ██████░░░░ 3/5    (60%) > ahead

Actions: 12 tổng | 4 done | 3 doing | 1 blocked | 4 pending
Tốc độ: 1.2 done/tuần (kế hoạch: 1.5)

Cần chú ý
  - A005 blocked 5 ngày: chờ approve từ stakeholder
  - A007 quá hạn 3 ngày (PIC: An)
  - KR2 cần +8 đơn vị/tháng để kịp target
```

### Phase 3: Tự xác định mode

Heuristic, sau đó hỏi user xác nhận:

| Tín hiệu | Mode đề xuất |
|----------|--------------|
| Daily update, không blocker mới, KR on-track | `light` |
| Đến milestone deadline, hoặc KR at-risk, hoặc nhiều blocked | `deep` |
| User nói "review", "tổng kết", "đánh giá", "lookback" | `deep` |
| User nói "update", "xong rồi", "cập nhật" | `light` |
| Mọi action `done` → đề xuất | `closure` (deep + chốt) |

```
Đề xuất mode: light (cập nhật nhanh, không phân tích sâu).
Đổi mode? (light/deep/closure)
```

---

### Phase 4a: Mode LIGHT (cập nhật progress nhanh)

Phạm vi: CHỈ progress fields. Không sửa cấu trúc.

1. Hỏi user có thay đổi: KR current, action status (done/doing/blocked), blocker mới.
2. CONFIRM trước khi ghi:
   ```
   Thay đổi sắp ghi
   - KR1.current: 40 > 50
   - A003.status: doing > done
   - A005.status: doing > blocked, lý do: chờ approve
   Xác nhận? (y/sửa/huỷ)
   ```
3. Áp dụng:
   - Ghi đè progress: `objective.md` (KR.current, KR.status), `plan.md` (counters), `actions/*.md` (frontmatter status).
   - Append log: `.okr/log/YYYY-MM-DD.md`. File ngày đã có → append section mới.
4. Đề xuất next action: highlight việc cần làm trong 1-7 ngày tới.

---

### Phase 4b: Mode DEEP (review sâu + delegate điều chỉnh cấu trúc)

Phạm vi: phân tích root cause + ĐỀ XUẤT điều chỉnh. KHÔNG tự sửa cấu trúc. Việc apply đẩy sang `okr-init` hoặc `okr-plan`.

#### Bước 1: Update progress nếu cần

Hỏi user có update progress nào trước phân tích (giống light). Nếu có → áp dụng + log như light.

#### Bước 2: Phân tích root cause

Cho mỗi vấn đề (KR at-risk, blocker, quá hạn):
- Hỏi "tại sao?" tối thiểu 3 lần.
- Phân biệt nhân (gốc) vs duyên (điều kiện).
- Tách triệu chứng vs nguyên nhân.

Hiển thị:
```
Phân tích
## Đạt tốt
  - KR3 vượt kế hoạch vì [lý do]
## Cần cải thiện
  - KR2 trễ 15%: root cause là [...] (không phải triệu chứng [...])
  - A005 blocked 5 ngày: nguyên nhân là dependency [...]
```

#### Bước 3: Đề xuất điều chỉnh cấu trúc (kèm phân loại delegate)

Mỗi đề xuất gắn nhãn skill sẽ áp dụng:

```
Đề xuất điều chỉnh
| # | Đề xuất                            | Lý do            | Áp dụng qua            |
|---|------------------------------------|------------------|------------------------|
| 1 | Giảm KR2 target: 50 > 35           | Market shift     | okr-init update-objective |
| 2 | Thêm action A013 "Tăng marketing"  | Thiếu đẩy KR2    | okr-plan update        |
| 3 | Dời M2: 2026-11-15 > 2026-11-30    | A005 chậm        | okr-plan update        |
| 4 | Đổi PIC A007: An > Dũng            | An quá tải       | okr-init update-resource|
| 5 | Tăng khả dụng Bình: 50% > 80%      | Cần đẩy build    | okr-init update-resource|
```

#### Bước 4: User chọn áp dụng cái nào

```
Đồng ý đề xuất nào? (vd: 1,3,4 / không / sửa N: <new value>)
```

#### Bước 5: Delegate sang skill phù hợp

Gom đề xuất user đồng ý theo skill target:
- Đề xuất gắn `okr-init update-objective` → kích hoạt `okr-init` mode `update-objective` với danh sách thay đổi KR/period.
- Đề xuất gắn `okr-init update-resource` → kích hoạt `okr-init` mode `update-resource` với danh sách thay đổi PIC/khả dụng/tool.
- Đề xuất gắn `okr-plan update` → kích hoạt `okr-plan` mode `update` với danh sách thay đổi action/milestone.

Skill được delegate sẽ tự chạy phase confirm + ghi file (theo flow của riêng nó). Track CHỈ truyền context (lý do + giá trị mới), KHÔNG tự ghi SOT objective/plan.

#### Bước 6: Ghi log review

Sau khi tất cả delegate hoàn tất, append `.okr/log/reviews/YYYY-MM-DD.md`:
- Tổng kết KR
- Phân tích root cause
- Đề xuất + cái nào đã apply (kèm skill nào áp dụng)

Đồng thời append `log/YYYY-MM-DD.md` link sang file review.

#### Bước 7: Đề xuất next action

---

### Phase 4c: Mode CLOSURE (mọi action done)

Như deep + thêm:
1. Tính tổng kết toàn period: KR đạt vs target, % thời gian, lessons learned.
2. Hỏi user: chuyển status objective → `completed`, hay tạo follow-up project?
3. Nếu user đồng ý → delegate sang `okr-init` mode `update-objective` để đổi `status: completed`.
4. Ghi log review closure (kèm section `## Lessons`).

## Schema

Đọc `references/data-format.md` cho schema:
- `log/YYYY-MM-DD.md` (tracking entries)
- `log/reviews/YYYY-MM-DD.md` (review entries)
- Progress fields được phép ghi đè
- Cấu trúc fields delegate sang skill khác

Đọc `references/metrics.md` cho cách tính KR %, trend, timeline.

## Quy tắc

- Dashboard luôn hiển thị TRƯỚC khi hỏi update.
- Mọi thay đổi qua phase confirm.
- Mode `light`: chỉ sửa progress fields (KR.current, action.status, plan counters). Cấm sửa cấu trúc.
- Mode `deep`: KHÔNG tự sửa KR target / action mới / dời deadline / đổi PIC. Delegate sang `okr-init` / `okr-plan`.
- Phân tích root cause BẮT BUỘC ≥3 lần "tại sao", không dừng ở triệu chứng.
- Ghi đè SOT progress fields, append log. Không ngược lại.
- Log review (deep) ghi vào `log/reviews/`. Log thường ghi vào `log/`.
- Cuối flow LUÔN đề xuất next action cụ thể (file/người/thời điểm).
- Habit type: dashboard hiển thị streak + adherence rate thay vì %.
