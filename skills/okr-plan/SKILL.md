---
name: okr-plan
description: "Quản lý kế hoạch + actions: tạo mới hoặc điều chỉnh dựa trên feedback từ `okr-track`. Skill được kích hoạt từ `/okr` khi cần khởi tạo plan, hoặc khi `okr-track` mode `deep` đề xuất thay đổi cấu trúc plan (thêm/bớt actions, dời milestone, đổi PIC). 2 sub-mode: `new` (tạo mới khi chưa có plan.md), `update` (điều chỉnh plan/actions hiện có). KHÔNG gọi trực tiếp trừ khi user gõ `/okr plan` hoặc `/okr-plan`."
---

# okr-plan: Tạo + cập nhật plan & actions

Skill quản lý 2 SOT: `plan.md` và `actions/`. Hỗ trợ tạo mới và điều chỉnh sau (khi `okr-track` deep phát hiện cần thay đổi cấu trúc).

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại. Thiếu → quay lại `/okr` để init.
- `.okr/resources.md` tồn tại. Thiếu → hỏi user có bổ sung trước (`/okr` → `okr-init` mode `update-resource`) hay tiếp tục với PIC trống.

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

### Phase 2: Đề xuất phân rã (KHÔNG ghi file)

Với mỗi KR:
1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Đề xuất PIC từ resources.md cho mỗi action.

### Phase 3: CONFIRM Plan (BẮT BUỘC)

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

Xác nhận? (y / sửa <ID>: <field>=<value> / xoá <ID> / thêm)
```

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

Đọc `plan.md` + frontmatter `actions/*.md` + `objective.md`. Hiển thị:

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
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng.
3. Sync `resources.md` (cột Actions của PIC) + `last_updated`.
4. Đề xuất chạy `okr-track` mode `light` để confirm state mới.

---

## Schema

- `references/data-format.md`: schema `plan.md` + frontmatter `actions/*.md`.
- `references/task-format.md`: template body action file.

## Quy tắc

- KHÔNG ghi file trước phase confirm.
- Mỗi action BẮT BUỘC có:
  - Definition of Done rõ ràng
  - Output/Deliverable cụ thể (file, số liệu, sự kiện)
  - PIC (hoặc `unassigned` nếu resource thiếu)
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output đo được → CẤM.
- Dependencies hợp lệ: ID tồn tại, không vòng tròn.
- Mode `update` không sửa KR target. Đó là việc của `okr-init` mode `update-objective`.
- Mode `update` không sửa action.status hay KR.current. Đó là việc của `okr-track` mode `light`.
- Habit type: plan.md chứa recurring tasks thay vì milestones, không tạo action files.
