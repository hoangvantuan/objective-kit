# Data Format: Lessons (okr-retro)

Schema cho layer bài học. SOT của `okr-retro`. Sinh tại project đích, dưới `.okr/`.

## Cây thư mục

```
.okr/lessons/
├── index.md              # Overview + link. AUTO-LOAD mỗi phiên OKR.
├── skill/                # Loại A: công cụ (tool) - năng lực harness, luôn shared
│   └── Sxxx-slug.md
├── workflow/             # Loại C: bản đồ điều phối - lưỡng tính shared/local
│   └── Wxxx-slug.md
└── project/              # Loại B: tri thức tĩnh về project, luôn local
    └── Pxxx-slug.md
```

- Loại A (`skill/`): công cụ ("how"). Năng lực nguyên tử của harness. Luôn `scope=shared`. Hàng đợi cải tiến, user port thủ công về repo gốc `objective-kit`.
- Loại C (`workflow/`): bản đồ ("khi nào/thứ tự nào"). **Lưỡng tính**: `shared` = điều phối tổng quát, port về harness thành flow; `local` = playbook riêng project, sống tại chỗ.
- Loại B (`project/`): tri thức tĩnh về project đích (dữ kiện, định nghĩa). Luôn `scope=local`, sống tại chỗ.

## File bài học (template unified)

Tên file: `Sxxx-slug.md` (skill), `Wxxx-slug.md` (workflow), hoặc `Pxxx-slug.md` (project). `xxx` = số 3 chữ số tăng dần, đếm riêng từng loại.

```yaml
---
id: W001                 # S### skill · W### workflow · P### project (đếm riêng từng loại)
type: workflow           # skill | workflow | project
mode: create             # improve | create. CHỈ skill/workflow. project bỏ trống.
scope: local             # shared | local. workflow lưỡng tính; skill luôn shared; project luôn local.
status: active           # xem bảng "Status values"
date: "YYYY-MM-DD"        # ngày trích xuất
target:                  # skill/workflow: file cần sửa (mode=improve) HOẶC vị trí đề xuất (mode=create). project bỏ trống.
area:                    # CHỈ project: mảng tri thức (vd "định nghĩa KR", "capacity"). skill/workflow bỏ trống.
essence: "Câu lõi hành động 1 dòng. Đây là dòng đưa lên index."
---

## Bối cảnh
[Phiên này xảy ra gì dẫn tới bài học. 1-3 câu.]

## Bài học
[Lần sau làm khác đi thế nào. Rule cụ thể, actionable.]

## Bằng chứng
[Link action/log/file hoặc trích dẫn. mode=create + scope=shared: BẮT BUỘC liệt kê ≥3 mốc thực tế (mỗi mốc 1 dòng: ngày + nơi xảy ra). Đây là cái gác cổng minh bạch cho ngưỡng ≥3 lần. Các loại khác: 1 artifact hoặc "phiên YYYY-MM-DD".]
```

### Ràng buộc đóng (agent tự kiểm trước khi ghi)

| type | scope | mode | target | area |
| --- | --- | --- | --- | --- |
| `skill` | luôn `shared` | `improve` \| `create` | file cần sửa / vị trí đề xuất | trống |
| `workflow` | `shared` \| `local` (BẮT BUỘC chọn) | `improve` \| `create` | file cần sửa / vị trí đề xuất | trống |
| `project` | luôn `local` | trống | trống | mảng tri thức |

- `type=skill` mà `scope=local` là SAI: công cụ nguyên tử gần như luôn tổng quát. Nếu thật sự riêng project, nó là `workflow` local hoặc `project`, không phải skill.
- `mode=create` + chưa có file thật: `target` ghi chuỗi vị trí đề xuất, vd `"flow mới: okr-track/flow-weekly-sync"` hoặc `"skill mới: okr-xxx"`.
- `mode=create` + `scope=shared`: chỉ hợp lệ khi `## Bằng chứng` có ≥3 mốc, HOẶC bài sinh do user chủ động yêu cầu đưa vào harness (ghi rõ "user yêu cầu" ở Bằng chứng).

## Index (`index.md`)

Auto-load mỗi phiên. CHỈ chứa dòng tóm tắt, KHÔNG chứa body bài học. Ba bảng theo loại (A skill, C workflow, B project).

```markdown
# Lessons Index

> Auto-load mỗi phiên OKR. Bài học chi tiết: file trong `skill/`, `workflow/` và `project/`.
> Hàng đợi port về repo gốc objective-kit: loại A (skill) + workflow `scope=shared`.

## Cải tiến bộ skill OKR (loại A)

| ID | Essence | Target | Status | Link |
|----|---------|--------|--------|------|
| S001 | Câu lõi 1 dòng | okr-track/flow-light.md | pending | [S001](skill/S001-slug.md) |

## Workflow (loại C)

| ID | Essence | Scope | Target/Vị trí | Status | Link |
|----|---------|-------|---------------|--------|------|
| W001 | Câu lõi 1 dòng | local | playbook tuần | active | [W001](workflow/W001-slug.md) |

## Project: <tên project> (loại B)

| ID | Essence | Area | Status | Link |
|----|---------|------|--------|------|
| P001 | Câu lõi 1 dòng | định nghĩa KR | active | [P001](project/P001-slug.md) |

## Lưu trữ (ported / obsolete)

> Bài đã port hoặc lỗi thời. Giữ để trace, không lái hành vi.

| ID | Essence | Status | Link |
|----|---------|--------|------|
```

- Cột `Target` (skill/workflow) = file skill cần sửa (improve) hoặc vị trí đề xuất (create). Cột `Area` (project) = mảng tri thức (tự do). Cột `Scope` (workflow) = shared | local.
- Ba bảng A/C/B chỉ liệt kê bài còn hiệu lực (`pending` / `active`). Bài `ported` / `obsolete` chuyển xuống "Lưu trữ".

## Status values

| type | scope | status | Nghĩa |
| --- | --- | --- | --- |
| skill | shared | `pending` | Chưa port về repo gốc |
| skill | shared | `ported` | User đã port + cải tiến ở source |
| workflow | shared | `pending` | Bản đồ tổng quát, chờ port thành flow ở harness |
| workflow | shared | `ported` | Đã port về harness |
| workflow | local | `active` | Playbook riêng project, còn áp dụng |
| workflow | local | `obsolete` | Playbook không còn đúng |
| project | local | `active` | Tri thức còn đúng |
| project | local | `obsolete` | Tri thức hết đúng (objective đổi, đã khắc phục) |

> **Vòng đời theo scope, không theo type:** `shared` đi `pending → ported` (hàng đợi port). `local` đi `active → obsolete` (sống tại chỗ). Workflow là loại duy nhất có thể đổi scope: xem "Thăng scope" ở `SKILL.md`.

## Quy ước

- Mỗi bài = 1 file (dễ port/xoá từng cái).
- `id` tăng dần riêng từng loại (`S001`... skill · `W001`... workflow · `P001`... project). Tìm max id trong thư mục tương ứng rồi +1.
- Slug: tiếng Việt không dấu hoặc tiếng Anh, ≤5 từ, gạch ngang.
- `essence` là SOT của dòng index: sửa essence trong file thì cập nhật lại dòng index.
- Trùng bài: cập nhật file cũ (tinh chỉnh essence, có thể thêm ngày gặp lại vào Bối cảnh), KHÔNG tạo file mới.
