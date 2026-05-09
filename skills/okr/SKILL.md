---
name: okr
description: "Entry point chính cho mọi tương tác OKR. LUÔN dùng skill này đầu tiên khi user nhắc đến mục tiêu, dự án, OKR, tracking, kế hoạch, tài nguyên, PIC, hoặc gọi `/okr`. Skill tự đánh giá trạng thái `.okr/` hiện tại và chủ động kích hoạt skill phù hợp (init/plan/track) với context user cung cấp. User KHÔNG cần biết tên skill con, chỉ làm việc với `/okr`."
---

# okr: Orchestrator quản lý OKR

Skill điều phối trung tâm. User mặc định luôn vào skill này. Phân tích state + intent → kích hoạt skill phù hợp + truyền context.

## Nguyên tắc cốt lõi

1. **Single entry point**: User chỉ gọi `/okr` (kèm hoặc không kèm câu lệnh tự nhiên). Không cần nhớ tên skill con.
2. **Tự kích hoạt**: Sau khi xác định phase, gọi thẳng skill con (đọc SKILL.md tương ứng và thực thi), KHÔNG bắt user gõ thêm lệnh.
3. **Truyền context**: Mọi câu chữ user nói khi gọi `/okr` phải forward sang skill con (vd: "/okr tôi muốn làm app X" → kích hoạt `okr-init` mode `new` với context dự án app X).
4. **Không tự ghi file**: Skill này CHỈ phân tích, route, gọi skill con. Mọi thay đổi file do skill con xử lý (kèm phase confirm với user).

## Skill con (3)

| Skill | Vai trò | Sub-mode |
|-------|---------|----------|
| `okr-init` | Quản lý SOT objective + resource | `new`, `update-objective`, `update-resource` |
| `okr-plan` | Quản lý SOT plan + actions | `new`, `update` |
| `okr-track` | Đánh giá state + cập nhật progress + đề xuất + delegate điều chỉnh cấu trúc | `light`, `deep`, `closure` |

## Phân vai SOT (quan trọng)

| Field | Skill được phép sửa |
|-------|---------------------|
| Objective text, KR target/baseline, period, status | `okr-init` `update-objective` |
| Người, tool, ngân sách, PIC, khả dụng | `okr-init` `update-resource` |
| Milestones, action structure (title, deadline, deps, deliverable) | `okr-plan` `update` |
| KR.current, action.status, plan counters | `okr-track` `light`/`deep` |

`okr-track` mode `deep` chỉ ĐỀ XUẤT điều chỉnh cấu trúc, KHÔNG tự sửa. Nó delegate sang `okr-init`/`okr-plan` mode `update-*` để apply.

## Flow xử lý

### Bước 1: Đọc trạng thái `.okr/`

Đọc song song nếu file tồn tại:
- `.okr/objective.md` (frontmatter + KR)
- `.okr/plan.md` (frontmatter)
- `.okr/resources.md` (frontmatter)
- glob `.okr/actions/*.md` (chỉ frontmatter)
- Latest file trong `.okr/log/` (nếu có)

### Bước 2: Hiển thị status ngắn (luôn luôn)

```
Trạng thái OKR
  Objective : [tên hoặc "(chưa có)"]
  Period    : [start > end hoặc "n/a"]
  KR        : X/Y đạt | Y total
  Actions   : a done / b doing / c blocked / d pending
  Resource  : [đã có / thiếu PIC: tên action]
  Log gần   : YYYY-MM-DD ([X ngày trước])
```

### Bước 3: Suy luận next action

| State / Intent | Skill + Mode |
|----------------|--------------|
| `.okr/` chưa có HOẶC objective.md thiếu | `okr-init` mode `new` |
| User nhắc objective/KR/period/mục tiêu | `okr-init` mode `update-objective` |
| User nhắc người/tài nguyên/PIC/ngân sách/tool | `okr-init` mode `update-resource` |
| Có objective + resource, thiếu plan.md | `okr-plan` mode `new` |
| Có plan.md + user nhắc thêm/sửa action, dời deadline, đổi milestone | `okr-plan` mode `update` |
| User nhắc kế hoạch/lộ trình/milestone (mới) | `okr-plan` mode `new` |
| Đủ SOT + actions còn mở + user nhắc tiến độ/cập nhật/xong/blocked | `okr-track` mode `light` |
| Đủ SOT + user nhắc review/tổng kết/lookback/đánh giá sâu | `okr-track` mode `deep` |
| Mọi action `done` | `okr-track` mode `closure` |

Keyword routing khi user cung cấp context:
- "tài liệu / tài nguyên / nhân sự / công cụ / PIC / ngân sách" → `okr-init` `update-resource`
- "đổi target / sửa KR / đổi deadline objective" → `okr-init` `update-objective`
- "kế hoạch / plan / lộ trình / milestone / action mới" → `okr-plan` `new` (nếu chưa có) hoặc `update`
- "thêm action / sửa action / dời deadline / xoá action" → `okr-plan` `update`
- "khởi tạo / mục tiêu mới / dự án mới" → `okr-init` `new`
- "tiến độ / cập nhật / xong rồi / blocked" → `okr-track` `light`
- "review / tổng kết / lookback / đánh giá" → `okr-track` `deep`

### Bước 4: Xác nhận với user (1 câu)

```
> Đề xuất: chạy [skill] mode [mode] vì [lý do ngắn]. Tiếp tục? (y/sửa)
```

User reply `y` → kích hoạt skill con. User reply context khác → re-route.

### Bước 5: Kích hoạt skill con

Đọc `skills/<skill-con>/SKILL.md` rồi thực thi theo hướng dẫn của skill đó. Truyền context user đã nói khi gọi `/okr` vào prompt khởi đầu của skill con.

## Khi user gọi `/okr <subcommand>` rõ tên

| Lệnh | Hành động |
|------|-----------|
| `/okr init` | Bỏ qua state-check, gọi `okr-init` (mode tự detect) |
| `/okr init new` | `okr-init` mode `new` (cảnh báo nếu sẽ ghi đè) |
| `/okr init update-objective` | `okr-init` mode `update-objective` |
| `/okr init update-resource` | `okr-init` mode `update-resource` |
| `/okr plan` | Gọi `okr-plan` (mode tự detect) |
| `/okr plan new` | `okr-plan` mode `new` (cảnh báo nếu sẽ ghi đè) |
| `/okr plan update` | `okr-plan` mode `update` |
| `/okr resource` | Alias cho `okr-init` mode `update-resource` |
| `/okr track` | Gọi `okr-track` (mode tự detect) |
| `/okr track light\|deep\|closure` | Gọi `okr-track` mode tương ứng |
| `/okr status` | Chỉ hiển thị status (Bước 2), không kích hoạt skill |

## Quy tắc

- KHÔNG tạo/sửa file. Chỉ đọc + route.
- Đọc tối thiểu cần thiết: frontmatter trước, body chỉ khi cần.
- Status hiển thị bắt buộc trước mọi đề xuất.
- State mơ hồ → hỏi user thay vì đoán.
- Sau khi skill con xong, quay về `okr` để hiển thị status mới + đề xuất bước tiếp theo.
