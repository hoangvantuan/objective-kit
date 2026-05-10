---
name: okr-plan
description: "Sub-skill của /okr. Tạo hoặc điều chỉnh kế hoạch (plan.md) và actions. 2 mode: new, update. Được kích hoạt từ orchestrator /okr hoặc khi okr-track delegate thay đổi cấu trúc. KHÔNG gọi trực tiếp trừ khi user gõ /okr-plan."
---

# okr-plan: Tạo + cập nhật plan & actions

Skill quản lý 2 SOT: `plan.md` và `actions/`. Hỗ trợ tạo mới và điều chỉnh sau (khi `okr-track` deep phát hiện cần thay đổi cấu trúc).

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại. Thiếu → quay lại `/okr` để init.
- `.okr/resources.md` tồn tại. Thiếu → hỏi user có bổ sung trước (`/okr` → `okr-init` mode `update-resource`) hay tiếp tục với PIC trống.

## Quality Gate: đánh giá câu trả lời trước khi đi tiếp

Mỗi khi user trả lời (chọn initiative, confirm action, assign PIC...), agent tự kiểm tra 3 câu (KHÔNG hiển thị cho user):

1. **Đủ cụ thể?** Có thể chuyển thành action có deliverable đo được không? Vd: "Nghiên cứu thêm" → FAIL (output là gì? ai verify?).
2. **Giả định ẩn?** User bỏ qua constraint quan trọng không? Vd: thêm 5 actions mới nhưng không nói ai làm.
3. **Mâu thuẫn?** Có xung đột với resource/timeline đã có không? Vd: PIC 50% available nhưng gán 6 actions cùng deadline.

**Hành vi theo kết quả:**

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp |
| Bất kỳ fail | Follow-up ngay, dùng kỹ thuật phù hợp (xem Deepening Techniques) |
| User nói "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm PHẢI nhắc lại |
| User sốt ruột | Giảm độ sâu, giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn |

## Phase 0: Detect mode

| State | Mode |
|-------|------|
| `plan.md` chưa có | `new` |
| Có `plan.md` + user/track nhắc thay đổi cấu trúc | `update` |
| User chọn explicit (vd `/okr plan update`) | theo lựa chọn |

---

## Mode NEW: tạo plan từ đầu

### Phase 1: Đọc context

- `.okr/objective.md`: Objective + Key Results.
- `.okr/resources.md`: PIC available + công cụ + ngân sách.

### Phase 2: Đề xuất phân rã (KHÔNG ghi file, có đào sâu)

Với mỗi KR:
1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Đề xuất PIC từ resources.md cho mỗi action.

**Deepening Techniques (agent tự challenge trước khi trình user):**

| Khía cạnh | Kỹ thuật | Ví dụ |
|-----------|---------|-------|
| **Initiative** | Hỏi "nếu chỉ được làm 1, chọn cái nào?" | Buộc user ưu tiên, tránh plan phình to |
| **Action** | Yêu cầu mô tả output cụ thể (file, event, metric) | "Action 'Nghiên cứu thị trường': output là gì? Report? Slide? Data raw?" |
| **PIC** | Challenge khả dụng thực tế | "Bình 50% available, đang có 3 actions. Thêm action này thì Bình có kịp không?" |
| **Dependencies** | Vẽ critical path, hỏi user confirm | "A003 chờ A001 + A002. Nếu A002 trễ 1 tuần, plan có chịu được không?" |
| **Timeline** | So sánh tổng effort vs. available time | "12 actions, 3 tháng, 2 người = 6 person-months. Mỗi action ước 2 tuần = 6 months. Vừa khít, không buffer. OK?" |

Không cần challenge tất cả. Chỉ challenge khía cạnh nào Quality Gate chưa pass.

### Phase 3: CONFIRM Plan (BẮT BUỘC, kèm đánh giá)

Bảng confirm PHẢI kèm phần "Đánh giá nhanh" do agent tự phân tích. Không chỉ echo data.

```
Tóm tắt Plan
| Milestone     | Deadline    | # Actions | KR liên quan |
|---------------|-------------|-----------|--------------|
| M1: Research  | 2026-10-15  | 3         | KR1          |
| M2: Build MVP | 2026-11-15  | 5         | KR1, KR2     |
| M3: Launch    | 2026-12-15  | 4         | KR2, KR3     |

Actions (12 total)
| ID    | Title              | Milestone | PIC  | Deps     | Output            |
|-------|--------------------|-----------|------|----------|-------------------|
| A001  | Khảo sát user      | M1        | An   | -        | survey-report.md  |
| A002  | Phân tích đối thủ  | M1        | Bình | -        | competitor.xlsx   |
| A003  | Spec MVP           | M2        | An   | A001,A002| spec.md           |

Đánh giá nhanh
  Resource fit:
  - Tổng capacity: [X] person-months. Tổng actions: [Y]. Fit: [✅/⚠️]
  - PIC tải nhiều nhất: [tên], [N] actions, [%] available
  Timeline:
  - Critical path: [A001 → A003 → A007]. Độ dài: [X tuần]. Buffer: [Y tuần]
  Rủi ro:
  - [vd: A005 có dependency ngoài team, chưa có backup plan]
  - [vd: ⚠️ 3 actions chưa có PIC (TBD)]

Xác nhận? (y / sửa <ID>: <field>=<value> / xoá <ID> / thêm)
```

Quy tắc đánh giá:
- **Resource fit**: tính tổng capacity vs. số actions. PIC nào gánh nhiều nhất?
- **Timeline**: xác định critical path dài nhất, còn buffer không?
- **Rủi ro**: liệt kê field TBD, dependency ngoài, bottleneck tiềm ẩn.
- Nếu có rủi ro nghiêm trọng → đề xuất giải pháp cụ thể (thêm người, dời deadline, cắt scope).

### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`.
4. Cập nhật counters trong `plan.md`.
5. Cập nhật cột `Actions` trong `resources.md` (mapping PIC → action IDs) + `last_updated`.

### Phase 5: Hậu xử lý

Hiển thị: "Đã tạo plan với X actions. Chạy `/okr` để bắt đầu thực thi."

---

## Mode UPDATE: điều chỉnh plan/actions

Trigger phổ biến: `okr-track` mode `deep` phát hiện KR at-risk hoặc blocker dài → đề xuất điều chỉnh cấu trúc → kích hoạt `okr-plan` mode `update` với context đề xuất.

### Phase 1: Hiển thị state hiện tại

Đọc `plan.md` + frontmatter `actions/*.md` (**không đệ quy**, bỏ qua `actions/archive/`) + `objective.md` + `resources.md`. Hiển thị:

```
Plan hiện tại
Milestones (3)
| Milestone     | Deadline    | Actions  | Status                |
|---------------|-------------|----------|-----------------------|
| M1: Research  | 2026-10-15  | 3/3      | done                  |
| M2: Build MVP | 2026-11-15  | 2/5      | in-progress, trễ 5 ngày|
| M3: Launch    | 2026-12-15  | 0/4      | pending               |

Cảnh báo
- KR2 at-risk → cần thêm action marketing
- A005 blocked 5 ngày, dependency A002 đã done nhưng quality không đạt
```

### Phase 2: Hỏi user/nhận đề xuất từ track

Nếu vào từ `okr-track`: hiển thị đề xuất track đã đưa, user chọn áp dụng cái nào.

Menu nếu user vào trực tiếp:
1. Thêm action mới
2. Sửa action (title, PIC, due_date, status, deps, deliverable)
3. Xoá action
4. Dời deadline milestone
5. Thêm/xoá milestone
6. Đổi PIC hàng loạt

### Phase 3: Thu thập thay đổi

Hỏi chi tiết từng cái user chọn.

### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |
| Đổi PIC A007| An                 | Dũng                      |

Tác động
- A013 mới → resources.md cập nhật cột Actions của Marketing
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`.
3. Sync `resources.md` (cột Actions của PIC) + `last_updated`.
4. Đề xuất chạy `okr-track` mode `light` để confirm state mới.

---

## Schema

- `references/data-format.md`: schema `plan.md` + frontmatter `actions/*.md`.
- `references/task-format.md`: template body action file.
- `references/action-guide.md`: hướng dẫn viết action chất lượng (5 tiêu chí bắt buộc, effort, priority, verifier, anti-patterns). Đọc trước khi tạo/review actions.

## Quy tắc

- KHÔNG ghi file trước phase confirm.
- Mỗi action BẮT BUỘC có:
  - Definition of Done rõ ràng
  - Output/Deliverable cụ thể (file, số liệu, sự kiện)
  - PIC (hoặc `unassigned` nếu resource thiếu)
  - **Ai verify output?** (người hoặc cơ chế kiểm tra kết quả)
  - **Tiêu chí chất lượng** (output đạt khi nào? đo bằng gì?)
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output đo được → CẤM. Agent phải follow-up: "Output cụ thể là gì? Ai đọc/dùng output này?"
- Nếu user không trả lời được "ai verify" hoặc "output đo bằng gì" → action chưa đủ rõ, cần refine trước khi ghi.
- Dependencies hợp lệ: ID tồn tại, không vòng tròn.
- Mode `update` không sửa KR target. Đó là việc của `okr-init` mode `update-objective`.
- Mode `update` không sửa action.status hay KR.current. Đó là việc của `okr-track` mode `light`.
- Ongoing type: plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI (vd: "Mua đồ tập gym", "Đặt lịch khám sức khoẻ"). Actions này vẫn tuân quy tắc action bình thường (DoD, output, PIC, effort). Milestones không bắt buộc với Ongoing.
