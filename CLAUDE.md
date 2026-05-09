# Objective Kit

Bộ skill quản lý mục tiêu & dự án theo OKR.

## Triết lý

User chỉ làm việc với `**/okr**` (entry point duy nhất). Skill này tự đánh giá trạng thái hiện tại và chủ động kích hoạt skill con phù hợp với context user cung cấp. Không cần nhớ tên skill con.

## Cấu trúc (3 skill con + 1 orchestrator)

| Skill        | Vai trò                                   | Sub-mode                                     |
| ------------ | ----------------------------------------- | -------------------------------------------- |
| `/okr`       | Orchestrator điều phối                    | n/a                                          |
| `/okr-init`  | Quản lý SOT objective + resource          | `new`, `update-objective`, `update-resource` |
| `/okr-plan`  | Quản lý plan + actions                    | `new`, `update`                              |
| `/okr-track` | Đánh giá + cập nhật + đề xuất next action | `light`, `deep`, `closure`                   |


## Phân nhóm logic

- **Source of Truth (tạo + cập nhật cấu trúc)**: `init` (objective + resource) và `plan` (kế hoạch + actions). Mỗi skill BẮT BUỘC có phase confirm bảng tóm tắt trước khi ghi/sửa file.
- **Đánh giá + cập nhật progress + delegate điều chỉnh cấu trúc**: `track` (gộp tracking thường ngày + review sâu). Track tự ghi đè progress fields (KR.current, action.status); thay đổi cấu trúc (KR target, action mới, dời deadline, đổi PIC) → ĐỀ XUẤT rồi delegate sang `init`/`plan` mode `update-*` để apply.
- **Điều phối**: `okr`.

## Phân vai SOT fields

| Field | Skill được phép sửa |
|-------|---------------------|
| Objective text, KR target/baseline, period, status | `okr-init` `update-objective` |
| Người, tool, ngân sách, PIC, khả dụng | `okr-init` `update-resource` |
| Milestones, action structure (title, deadline, deps, deliverable, thêm/xoá) | `okr-plan` `update` |
| KR.current, action.status, plan counters | `okr-track` `light` hoặc `deep` |

## Quy ước dữ liệu

Mọi skill đọc/ghi trong thư mục `.okr/` của dự án hiện tại:

```
.okr/
├── objective.md       # SOT - mục tiêu + KR              (do okr-init quản)
├── resources.md       # SOT - người + tool + ngân sách   (do okr-init quản)
├── plan.md            # SOT - milestones + counters      (do okr-plan quản)
├── actions/           # SOT - 1 file/action              (do okr-plan + okr-track quản)
│   └── AXXX-*.md
└── log/               # Append-only                       (do okr-track ghi)
    ├── YYYY-MM-DD.md          # tracking light
    └── reviews/
        └── YYYY-MM-DD.md      # tracking deep / review
```

Mỗi skill chứa schema chi tiết trong `references/data-format.md` của riêng nó.

## Nguyên tắc thiết kế

1. **Single entry point**: User chỉ gọi `/okr`. Orchestrator route + truyền context.
2. **Confirm trước khi ghi**: `init` và `plan` BẮT BUỘC có bảng tóm tắt + xác nhận trước khi tạo/sửa file.
3. **SOT vs Log**: SOT ghi đè (luôn phản ánh trạng thái mới). Log append-only (lịch sử thay đổi theo thời gian).
4. **Resource gộp vào init**: Khởi tạo lần đầu (`new`) thu thập cả objective lẫn resource. Update sau qua mode `update-resource` của cùng skill `okr-init`.
5. **Track gộp Review**: 1 skill 2 mode chính (light/deep) thay vì 2 skill riêng. Tự chọn mode dựa trên context.
