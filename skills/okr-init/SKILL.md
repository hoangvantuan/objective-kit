---
name: okr-init
description: "Khởi tạo mục tiêu dự án theo OKR. Dùng khi bắt đầu dự án mới hoặc khi chưa có thư mục .okr/ trong folder hiện tại. Trigger: /okr-init, hoặc tự động khi /okr phát hiện chưa có .okr/"
---

# okr-init: Khởi tạo mục tiêu

Tạo `.okr/objective.md` cho dự án hiện tại. Hỗ trợ 2 loại: Project (có kỳ hạn) và Habit (thói quen).

## Checklist

1. Kiểm tra `.okr/` đã tồn tại chưa. Nếu rồi, hỏi user muốn tạo mới hay cập nhật
2. Hỏi loại mục tiêu: Project hay Habit
3. Thu thập thông tin:
   - **WHY**: Tại sao mục tiêu này quan trọng?
   - **HOW**: Cách tiếp cận tổng quan
   - **WHAT**: Kết quả cụ thể mong đợi
4. Viết Objective (hỏi 1 câu tại 1 thời điểm, không hỏi nhiều câu cùng lúc)
5. Viết Key Results (Project) hoặc Key Indicators (Habit). Đọc `references/okr-guide.md` để kiểm tra chất lượng
6. Kiểm tra SMART cho mỗi KR/KI
7. Tạo thư mục `.okr/` và file `objective.md`

## Format output

Đọc `references/data-format.md` để lấy schema chính xác.

### Project

```yaml
---
type: project
objective: "[tóm tắt ngắn]"
quarter: "[Q?-YYYY]"
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
status: active
---
```

Body: `## Objective` (mô tả WHY chi tiết), `## Key Results` (bảng), `## Context` (bối cảnh).

### Habit

```yaml
---
type: habit
objective: "[tóm tắt ngắn]"
start_date: YYYY-MM-DD
frequency: daily|weekly|monthly
status: active
---
```

Body: `## Objective` (mô tả WHY), `## Key Indicators` (bảng), `## Checklist lặp lại`.

## Quy tắc

- Hỏi từng câu một, không hỏi hàng loạt
- Đề xuất Objective và KR, nhưng luôn để user quyết
- Nếu KR không đạt SMART, chỉ ra cụ thể thiếu tiêu chí nào và gợi ý sửa
- Tạo `.okr/` folder nếu chưa có
- Chỉ tạo `objective.md`. Không tạo plan.md hay file khác
