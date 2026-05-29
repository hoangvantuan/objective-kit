# Data Format: Lessons (okr-retro)

Schema cho layer bài học. SOT của `okr-retro`. Sinh tại project đích, dưới `.okr/`.

## Cây thư mục

```
.okr/lessons/
├── index.md              # Overview + link. AUTO-LOAD mỗi phiên OKR.
├── skill/                # Loại A: bài học cải tiến bộ skill OKR
│   └── Sxxx-slug.md
└── project/              # Loại B: bài học về project đang làm
    └── Pxxx-slug.md
```

- Loại A (`skill/`): hàng đợi cải tiến harness. Record-only, user thủ công port về repo gốc `objective-kit`.
- Loại B (`project/`): tri thức về project đích, sống tại chỗ.

## File bài học (template unified)

Tên file: `Sxxx-slug.md` (skill) hoặc `Pxxx-slug.md` (project). `xxx` = số 3 chữ số tăng dần, đếm riêng từng loại.

```yaml
---
id: P001                 # S### cho skill, P### cho project
type: project            # skill | project
status: active           # skill: pending | ported  ·  project: active | obsolete
date: "YYYY-MM-DD"        # ngày trích xuất
target:                  # CHỈ khi type=skill: file/skill cần sửa (vd okr-track/flow-deep.md). type=project bỏ trống.
area:                    # CHỈ khi type=project: mảng tri thức (vd "định nghĩa KR", "capacity"). type=skill bỏ trống.
essence: "Câu lõi hành động 1 dòng. Đây là dòng đưa lên index."
---

## Bối cảnh
[Phiên này xảy ra gì dẫn tới bài học. 1-3 câu.]

## Bài học
[Lần sau làm khác đi thế nào. Rule cụ thể, actionable.]

## Bằng chứng
[Link action/log/file hoặc trích dẫn chứng minh. Nếu không có artifact, ghi "phiên YYYY-MM-DD".]
```

## Index (`index.md`)

Auto-load mỗi phiên. CHỈ chứa dòng tóm tắt, KHÔNG chứa body bài học. Hai bảng theo loại.

```markdown
# Lessons Index

> Auto-load mỗi phiên OKR. Bài học chi tiết: file trong `skill/` và `project/`.
> Loại A (skill) là hàng đợi cải tiến harness, port thủ công về repo gốc objective-kit.

## Cải tiến bộ skill OKR (loại A)

| ID | Essence | Target | Status | Link |
|----|---------|--------|--------|------|
| S001 | Câu lõi 1 dòng | okr-track/flow-light.md | pending | [S001](skill/S001-slug.md) |

## Project: <tên project> (loại B)

| ID | Essence | Area | Status | Link |
|----|---------|------|--------|------|
| P001 | Câu lõi 1 dòng | định nghĩa KR | active | [P001](project/P001-slug.md) |

## Lưu trữ (ported / obsolete)

> Bài đã port hoặc lỗi thời. Giữ để trace, không lái hành vi.

| ID | Essence | Status | Link |
|----|---------|--------|------|
```

- Cột `Target` (loại A) = file skill cần sửa. Cột `Area` (loại B) = mảng tri thức (tự do).
- Bảng A và B chỉ liệt kê bài còn hiệu lực (`pending` / `active`).
- Bài `ported` / `obsolete` chuyển xuống mục "Lưu trữ".

## Status values

| Loại | Status | Nghĩa |
|------|--------|-------|
| skill | `pending` | Chưa port về repo gốc |
| skill | `ported` | User đã port + cải tiến ở source |
| project | `active` | Còn đúng, còn áp dụng |
| project | `obsolete` | Không còn đúng (objective đổi, đã khắc phục) |

## Quy ước

- Mỗi bài = 1 file (dễ port/xoá từng cái).
- `id` tăng dần riêng từng loại (S001, S002... · P001, P002...). Tìm max id hiện có trong thư mục tương ứng rồi +1.
- Slug: tiếng Việt không dấu hoặc tiếng Anh, ≤5 từ, gạch ngang.
- `essence` là SOT của dòng index: sửa essence trong file thì cập nhật lại dòng index.
- Trùng bài: cập nhật file cũ (tinh chỉnh essence, có thể thêm ngày gặp lại vào Bối cảnh), KHÔNG tạo file mới.
