# Skill Improver Report: OKR System (5 skills)

Ngày: 2026-05-11

## Điểm trước/sau

| Tiêu chí            | okr         | okr-init    | okr-plan    | okr-track   | okr-capture |
| ------------------- | ----------- | ----------- | ----------- | ----------- | ----------- |
|                     | T → S       | T → S       | T → S       | T → S       | T → S       |
| Clarity             | 4→4         | 4→4         | 4→4         | 4→4         | 5→5         |
| Specificity         | 5→5         | 5→5         | 4→4         | 4→4         | 4→4         |
| Coverage            | 4→4         | 5→5         | 4→4         | 5→5         | 4→4         |
| Structure           | 5→5         | 2→4         | 4→4         | 4→4         | 4→4         |
| Cognitive Load      | 4→5         | 2→4         | 3→4         | 3→3         | 5→5         |
| Bloat Score         | 3→4         | 2→4         | 3→4         | 3→3         | 4→5         |
| Anti-patterns       | 4→4         | 3→4         | 4→4         | 3→3         | 4→4         |
| Description         | 4→4         | 4→4         | 4→4         | 4→4         | 4→4         |
| **Tổng (/40)**      | **33→35**   | **27→34**   | **30→32**   | **30→30**   | **34→35**   |
| **SKILL.md (dòng)** | **143→132** | **481→315** | **234→213** | **265→263** | **113→113** |


Tổng SKILL.md: 1236 → 1036 dòng (-200, giảm 16%)
Tổng toàn hệ thống (SKILL.md + references): 2676 → 2593 dòng (-83)

## Cải tiến đã thực hiện

### 1. okr-init: Tách update modes ra references

- Tạo `references/flow-update-objective.md` (92 dòng)
- Tạo `references/flow-update-resource.md` (60 dòng)
- SKILL.md: 481 → 315 dòng (-166)
- Hiệu quả: Mode NEW (primary flow) giữ attention cao nhất trong SKILL.md. Update modes ít dùng hơn → load khi cần.

### 2. Tạo delegate-protocol.md (shared)

- Tạo `okr/references/delegate-protocol.md` (35 dòng)
- Cắt ~35 dòng trùng từ okr-init, ~21 dòng từ okr-plan
- Hiệu quả: Pre-confirmed + reason display rules define 1 nơi. Sửa 1 chỗ, tất cả skill nhận.

### 3. Loại SOT table trùng từ okr/SKILL.md

- Thay 13 dòng bảng bằng 1 dòng pointer tới sot-ownership.md
- Hiệu quả: SOT table đã có trong sot-ownership.md (load cùng skill). Không cần copy trong SKILL.md body.

### 4. Consolidate Roadmap format + Inbox Aging

- okr-plan/data-format.md: thay 40 dòng Roadmap format bằng pointer tới shared-schemas.md
- okr-capture/data-format.md: thay 10 dòng Inbox Aging bằng pointer tới shared-schemas.md
- Hiệu quả: shared-schemas.md là canonical, không cần bản copy.

## Bài học rút ra (tổng quát)

1. **Multi-mode skill > 300 dòng cần tách mode phụ**: Khi skill có nhiều mode (new/update-X/update-Y), giữ mode chính (dùng nhiều nhất) trong SKILL.md, tách mode phụ ra references. Agent load đúng mode cần thiết.
2. **Cross-skill duplicate phát hiện bằng grep keyword**: Grep các keyword đặc trưng (pre_confirmed, reason display, Roadmap format) cross toàn bộ skills → thấy trùng nhanh. Tốn 1 phút, tiết kiệm attention model lâu dài.
3. **"Canonical" claim cần enforce**: Khi 1 file claim canonical nhưng bản copy vẫn tồn tại ở nơi khác, model có thể đọc bản copy thay vì canonical → mâu thuẫn khi sửa. Enforce bằng cách thay copy bằng pointer.
4. **Delegate protocol là candidate tốt cho shared reference**: Khi nhiều skill nhận delegate từ 1 skill (init + plan nhận từ track), behavior khi nhận nên define 1 nơi. Tránh "nói khác nhau" giữa các skill nhận.
