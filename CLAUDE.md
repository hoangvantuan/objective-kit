# Objective Kit

Bộ skill quản lý mục tiêu & dự án theo OKR.

## Cấu trúc

- `/okr` : Orchestrator, tự phát hiện phase phù hợp
- `/okr-init` : Khởi tạo mục tiêu (Project hoặc Habit)
- `/okr-plan` : Lập kế hoạch hành động
- `/okr-resource` : Quản lý tài nguyên & vai trò
- `/okr-track` : Theo dõi tiến độ
- `/okr-review` : Lookback & điều chỉnh

## Quy ước dữ liệu

Mọi skill đọc/ghi trong thư mục `.okr/` của dự án hiện tại. Mỗi skill chứa schema chi tiết trong `references/data-format.md` của riêng nó.
