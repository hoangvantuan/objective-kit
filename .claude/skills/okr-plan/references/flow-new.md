# Flow: Plan New (tạo plan từ đầu)

> Self-contained. Orchestrator chạy inline, mang theo context đã có.

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại. Thiếu → cần chạy init trước.
- `.okr/resources.md` tồn tại. Thiếu → hỏi user bổ sung trước hay tiếp tục với Solo Profile mặc định (capacity 0h/tuần, skills rỗng, cảnh báo cross-check fit ở Phase 3).

## Quality Gate

3 câu check trước mỗi follow-up (đọc `quality-gate.md` nếu cần chi tiết):

- "Đủ cụ thể?" → action "Nghiên cứu thêm" FAIL (output là gì? đo bằng gì?).
- "Giả định ẩn?" → thêm 5 actions mới nhưng không nói effort tổng bao nhiêu giờ.
- "Mâu thuẫn?" → capacity 10h/tuần nhưng gán 6 actions effort `m` cùng deadline tuần sau.

## Phase 0: Detect mode

| State | Mode |
|-------|------|
| `plan.md` chưa có | `new` (file này) |
| Có `plan.md` + user/track nhắc thay đổi cấu trúc | `update` (xem `flow-update.md`) |

**Cảnh báo ghi đè**: Nếu mode `new` nhưng `plan.md` đã tồn tại:
1. Hiển thị: "Plan đã tồn tại (X milestones, Y actions active, Z done/archive). Mode `new` sẽ ghi đè plan + xoá action files active."
2. Hỏi: "(ghi đè / chuyển sang update / huỷ)"

## Phase 1: Đọc context

- `.okr/objective.md`: Objective + Key Results/Indicators.
- `.okr/resources.md`: Solo Profile (capacity, skills) + công cụ + ngân sách.

## Phase 2: Đề xuất phân rã

Với mỗi KR:

1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Mặc định `pic: self`. Cảnh báo nếu action cần skill chưa có trong Solo Profile.
6. Ghi chú ngắn → field `notes` (≤50 ký tự). Không có thì để trống.
7. Nếu action liên kết tool có `Cách dùng` là `skill:` hoặc `mcp:`, hỏi external ID. Nếu có → `external_ids` frontmatter.

**Deepening Techniques:**

| Khía cạnh | Kỹ thuật | Ví dụ |
|-----------|---------|-------|
| Initiative | "Nếu chỉ được làm 1, chọn cái nào?" | Buộc ưu tiên, tránh plan phình to |
| Action | Mô tả output cụ thể (file, event, metric) | "Output 'Nghiên cứu' là gì? Report? Slide?" |
| Skill match | Challenge action có skill chưa có | "Action 'Design Figma' nhưng chưa có skill design" |
| Dependencies | Vẽ critical path, hỏi confirm | "A003 chờ A001+A002. A002 trễ 1 tuần có sao?" |
| Timeline | Tổng effort vs. capacity | "12 actions, 120h available. Mỗi action ~8h = 96h. Buffer OK?" |

Chỉ challenge khía cạnh nào Quality Gate chưa pass.

## Phase 3: CONFIRM Plan (BẮT BUỘC, kèm đánh giá)

```
Tóm tắt Plan
| Milestone    | Deadline   | # Actions | KR liên quan |
|-------------|------------|-----------|--------------|
| M1: Research | 2026-10-15 | 3         | KR1          |

Actions (N total, pic mặc định self)
| ID   | Title           | Milestone | Effort | Deps | Output          | Notes    | Ext IDs      |
|------|-----------------|-----------|--------|------|-----------------|----------|--------------|
| A001 | Khảo sát user   | M1        | m      | -    | survey-report.md|          |              |

Đánh giá nhanh
  Capacity fit:
  - Tổng effort: [X] giờ. Capacity: [Y] giờ. Fit: [✅/⚠️]
  - Tuần dồn việc nhất: [tuần W]
  Timeline:
  - Critical path: [A001 → A003 → A007]. Buffer: [N tuần]
  Rủi ro:
  - [dependency ngoài, skill thiếu, field TBD]

Xác nhận? (y / sửa <ID>: <field>=<value> / xoá <ID> / thêm)
```

Quy tắc đánh giá:
- Capacity fit: tổng effort vs. capacity còn lại. Tuần dồn nhất?
- Timeline: critical path dài nhất, buffer?
- Rủi ro: field TBD, dependency ngoài, skill chưa có.
- Rủi ro nghiêm trọng → đề xuất giải pháp (cắt scope, dời deadline, học skill, outsource).

## Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`. Schema xem `data-format.md`. Mặc định `pic: self`.
4. Cập nhật counters trong `plan.md`.
5. **Render bảng Roadmap** trong `plan.md` body `## Roadmap`. Format xem `schemas.md` section "Roadmap format". Sắp: Priority → Deadline. Ongoing không có milestone → heading `### Chưa phân milestone`.

## Phase 5: Hậu xử lý

"Đã tạo plan với X actions."

## Ongoing type

Ongoing plan có thể chứa `## Practices` thay vì milestones. Chi tiết xem `action-guide.md` section "Ongoing type: khi nào tạo action?".

## Schema

- `data-format.md`: schema plan.md + frontmatter actions/*.md
- `task-format.md`: template body action file
- `action-guide.md`: viết action chất lượng (5 tiêu chí, effort, priority, DoD, anti-patterns)

## Quy tắc

- KHÔNG ghi file trước phase confirm.
- Mỗi action BẮT BUỘC: DoD rõ ràng, Output/Deliverable cụ thể, `pic`, tiêu chí chất lượng.
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output → CẤM. Follow-up: "Output cụ thể là gì?"
- Dependencies hợp lệ: ID tồn tại, không vòng tròn.
- Không sửa KR target (việc `okr-init` mode update-objective).
- Không sửa KR.current (việc `okr-track`).
