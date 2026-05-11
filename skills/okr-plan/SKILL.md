---
name: okr-plan
description: "Sub-skill của /okr. Tạo hoặc điều chỉnh kế hoạch (plan.md) và actions. 2 mode: new, update. Được kích hoạt từ orchestrator /okr hoặc khi okr-track delegate thay đổi cấu trúc. KHÔNG gọi trực tiếp trừ khi user gõ /okr-plan."
---

# okr-plan: Tạo + cập nhật plan & actions

Skill quản lý 2 SOT: `plan.md` và `actions/`. Hỗ trợ tạo mới và điều chỉnh sau (khi `okr-track` deep phát hiện cần thay đổi cấu trúc).

## Điều kiện tiên quyết

- `.okr/objective.md` tồn tại. Thiếu → quay lại `/okr` để init.
- `.okr/resources.md` tồn tại. Thiếu → hỏi user có bổ sung trước (`/okr` → `okr-init` mode `update-resource`) hay tiếp tục với Solo Profile mặc định (capacity 0h/tuần, skills rỗng — sẽ cảnh báo cross-check fit ở Phase 3 confirm).

## Quality Gate

Áp dụng 3 câu kiểm tra core + bảng hành vi từ `skills/okr/references/quality-gate.md`. Đọc file đó trước khi tiến hành các phase hỏi user.

Ví dụ áp dụng cho mode `new` / `update`:
- "Đủ cụ thể?" → action "Nghiên cứu thêm" FAIL (output là gì? đo bằng gì?).
- "Giả định ẩn?" → thêm 5 actions mới nhưng không nói effort tổng bao nhiêu giờ.
- "Mâu thuẫn?" → capacity 10h/tuần nhưng gán 6 actions effort `m` (1-2 ngày mỗi cái) cùng deadline tuần sau.

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
- `.okr/resources.md`: Solo Profile (capacity h/tuần, skills) + công cụ + ngân sách.

### Phase 2: Đề xuất phân rã (KHÔNG ghi file, có đào sâu)

Với mỗi KR:
1. Đề xuất 1-3 **Initiatives** (sáng kiến lớn).
2. Mỗi Initiative → 2-5 **Actions** cụ thể.
3. Nhóm actions thành **Milestones** theo thời gian.
4. Đề xuất dependencies (action B chờ A).
5. Mặc định mỗi action `pic: self`. Cảnh báo nếu action cần skill chưa có trong Solo Profile (đề xuất thêm action học/outsource trước).

**Deepening Techniques (agent tự challenge trước khi trình user):**

| Khía cạnh | Kỹ thuật | Ví dụ |
|-----------|---------|-------|
| **Initiative** | Hỏi "nếu chỉ được làm 1, chọn cái nào?" | Buộc user ưu tiên, tránh plan phình to |
| **Action** | Yêu cầu mô tả output cụ thể (file, event, metric) | "Action 'Nghiên cứu thị trường': output là gì? Report? Slide? Data raw?" |
| **Skill match** | Challenge action có skill chưa có | "Action 'Design Figma' nhưng Solo Profile chưa có skill design. Có học trước, outsource, hay đổi approach?" |
| **Dependencies** | Vẽ critical path, hỏi user confirm | "A003 chờ A001 + A002. Nếu A002 trễ 1 tuần, plan có chịu được không?" |
| **Timeline** | So sánh tổng effort vs. capacity | "12 actions, 12 tuần, capacity 10h/tuần = 120h. Mỗi action ước 8h = 96h. Buffer 24h. OK?" |

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

Actions (12 total, mọi action `pic: self`)
| ID    | Title              | Milestone | Effort | Deps      | Output            |
|-------|--------------------|-----------|--------|-----------|-------------------|
| A001  | Khảo sát user      | M1        | m      | -         | survey-report.md  |
| A002  | Phân tích đối thủ  | M1        | l      | -         | competitor.xlsx   |
| A003  | Spec MVP           | M2        | l      | A001,A002 | spec.md           |

Đánh giá nhanh
  Capacity fit:
  - Tổng effort ước tính: [X] giờ. Capacity: [10h/tuần × N tuần] = [Y] giờ. Fit: [✅/⚠️]
  - Tuần dồn việc nhất: [tuần W], [N] actions cùng deadline.
  Timeline:
  - Critical path: [A001 → A003 → A007]. Độ dài: [X tuần]. Buffer: [Y tuần]
  Rủi ro:
  - [vd: A005 dependency ngoài (chờ stakeholder), chưa có backup plan]
  - [vd: ⚠️ 2 actions cần skill chưa có trong Solo Profile (Figma, SQL)]

Xác nhận? (y / sửa <ID>: <field>=<value> / xoá <ID> / thêm)
```

Quy tắc đánh giá:
- **Capacity fit**: tính tổng effort ước tính (giờ) vs. capacity còn lại. Tuần nào dồn việc nhất?
- **Timeline**: xác định critical path dài nhất, còn buffer không?
- **Rủi ro**: liệt kê field TBD, dependency ngoài, skill chưa có trong Solo Profile.
- Nếu có rủi ro nghiêm trọng → đề xuất giải pháp cụ thể (cắt scope, dời deadline, học skill trước, outsource).

### Phase 4: Ghi file

1. Tạo `.okr/plan.md`.
2. Tạo thư mục `.okr/actions/`.
3. Ghi mỗi action thành `AXXX-slug.md`. Mọi action `pic: self`.
4. Cập nhật counters trong `plan.md`.

### Phase 5: Hậu xử lý

Hiển thị: "Đã tạo plan với X actions. Chạy `/okr` để bắt đầu thực thi."

---

## Mode UPDATE: điều chỉnh plan/actions

Trigger phổ biến: `okr-track` mode `deep` phát hiện KR at-risk hoặc blocker dài → đề xuất điều chỉnh cấu trúc → kích hoạt `okr-plan` mode `update` với context đề xuất.

### Phase 1: Hiển thị state hiện tại

**Bypass detect (M7):** Nếu mode `update` được kích hoạt qua delegate từ `okr-track` Phase 5 (inbox processing) **kèm `context.items[]` đã pre-processed** (mỗi item có `title`, `description`, `related_kr`/`related_action` đã validate, `effort` ước, `milestone` gợi ý), SKIP Phase 1 state display + Phase 2 menu, đi thẳng Phase 3 (gom item thành changes) → Phase 4 confirm với data sẵn. State display chỉ chạy khi user vào trực tiếp `/okr plan update`.

Mode bình thường (state display): đọc `plan.md` + frontmatter `actions/*.md` (**không đệ quy**, bỏ qua `actions/archive/`) + `objective.md` + `resources.md`. Hiển thị:

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

3 nguồn trigger:

1. **Từ track deep delegate** (`context.changes` + `context.reason` ở payload): hiển thị đề xuất track đã đưa, user chọn áp dụng cái nào (nếu `pre_confirmed: false` hoặc không có), hoặc đi thẳng Phase 4 confirm (nếu `pre_confirmed: true`).
2. **Từ track inbox delegate** (`context.items[]` pre-processed): gom items thành Phase 3 changes, đi thẳng Phase 4 confirm. Mỗi `action` item → 1 "Thêm action mới" với data có sẵn (title, description, related_kr, effort gợi ý, milestone gợi ý). Mỗi `blocker` đụng action → 1 "Sửa action" status=blocked + lý do. Mỗi `resource` → delegate tiếp sang `okr-init update-resource` (không xử lý ở đây).
3. **User vào trực tiếp** (`/okr plan update`): hiển thị menu:
   1. Thêm action mới
   2. Sửa action (title, due_date, status, deps, deliverable, effort, priority)
   3. Xoá action
   4. Dời deadline milestone
   5. Thêm/xoá milestone

### Phase 3: Thu thập thay đổi

Hỏi chi tiết từng cái user chọn.

### Phase 4: CONFIRM diff

**Pre-confirmed flow (T4c):** Nếu payload có `context.pre_confirmed: true` (user đã confirm tại track Bước 4 với full all-changes preview), SKIP ask "Xác nhận? (y/sửa/huỷ)" và đi thẳng Phase 5 ghi file. Vẫn HIỂN THỊ block "Lý do điều chỉnh" + bảng diff để user trace, kèm 1 dòng cuối: `(Đã được confirm tại track Bước 4. Ghi file ngay.)`.

Default flow (không pre_confirmed): nếu vào từ track delegate (có `context.reason`), HIỂN THỊ trước bảng diff:

```
Lý do điều chỉnh (từ track deep)
  Market shift Q4 (root cause Bước 2): tăng trưởng ngành chậm 30%,
  cần thêm marketing để đẩy KR2.
  Source: log/reviews/2026-12-01.md
```

Sau đó luôn hiển thị bảng diff:

```
Thay đổi sắp áp dụng (plan.md + actions/)
| Loại        | Trước              | Sau                       |
|-------------|--------------------|---------------------------|
| Thêm action | -                  | A013 "Marketing campaign" |
| Sửa A005    | due_date 2026-11-10| due_date 2026-11-20       |
| Sửa M2      | deadline 2026-11-15| deadline 2026-11-25       |

Tác động
- A013 mới → 1 action mới gắn KR2
- Dời M2 → 2 actions con tự dời theo? (y/giữ nguyên)

Xác nhận? (y / sửa / huỷ)
```

Quy tắc reason display:
- Nếu `context.reason` rỗng hoặc không có (user vào trực tiếp `/okr plan update`) → KHÔNG render block "Lý do điều chỉnh".
- `source_review` luôn đi kèm reason (cùng block).
- Reason là plain text 1-3 câu, KHÔNG markdown đặc biệt (giữ readable trong terminal).

Quy tắc pre_confirmed:
- `pre_confirmed: true` chỉ áp dụng khi track Bước 4 đã hiển thị **all-changes diff** + user reply "y" (xem `okr-track/SKILL.md` Phase 4b Bước 4-5).
- Nếu thiếu 1 trong 2 (track Bước 4 không show full diff, hoặc payload không set `pre_confirmed`), plan vẫn chạy default flow (hỏi confirm).
- Pre-confirmed bypass CHỈ skip ask "y/sửa/huỷ". Vẫn ghi log + báo cáo bình thường.

### Phase 5: Áp dụng

1. Ghi đè `plan.md` (counters, milestones).
2. Ghi/sửa file `actions/AXXX-*.md` tương ứng. **Không** tạo hoặc sửa file trong `actions/archive/`. Mọi action mới `pic: self`.
3. Đề xuất chạy `okr-track` mode `light` để confirm state mới.

---

## Schema

- `references/data-format.md`: schema `plan.md` + frontmatter `actions/*.md`.
- `references/task-format.md`: template body action file.
- `references/action-guide.md`: hướng dẫn viết action chất lượng (5 tiêu chí bắt buộc, effort, priority, DoD-based verify, anti-patterns). Đọc trước khi tạo/review actions.

## Quy tắc

- KHÔNG ghi file trước phase confirm.
- Mỗi action BẮT BUỘC có:
  - Definition of Done rõ ràng (mỗi item đo được, user tự check được)
  - Output/Deliverable cụ thể (file, số liệu, sự kiện)
  - `pic: self` (solo default, không thay đổi)
  - **Tiêu chí chất lượng** (output đạt khi nào? đo bằng gì?)
- Action mơ hồ kiểu "Nghiên cứu thêm" không có output đo được → CẤM. Agent phải follow-up: "Output cụ thể là gì?"
- Nếu user không trả lời được "output đo bằng gì" → action chưa đủ rõ, cần refine trước khi ghi.
- Dependencies hợp lệ: ID tồn tại, không vòng tròn.
- Mode `update` không sửa KR target. Đó là việc của `okr-init` mode `update-objective`.
- Mode `update` không sửa action.status hay KR.current. Đó là việc của `okr-track` mode `light`.
- Ongoing type: plan.md body chứa `## Practices` (hành động lặp lại để duy trì KI). Ngoài ra, Ongoing CÓ THỂ tạo action files khi cần task cải thiện KI (vd: "Mua đồ tập gym", "Đặt lịch khám sức khoẻ"). Actions này vẫn tuân quy tắc action bình thường (DoD, output, effort, `pic: self`). Milestones không bắt buộc với Ongoing.
