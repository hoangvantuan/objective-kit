# Delegate Protocol: Pre-confirmed + Reason Display

Quy tắc chung khi `okr-init` hoặc `okr-plan` nhận delegate payload từ `okr-track` deep.

## Reason Display

Nếu vào từ track delegate (có `context.reason`), HIỂN THỊ block lý do **trước** bảng diff:

```
Lý do điều chỉnh (từ track deep)
  [context.reason text, 1-3 câu]
  Source: [context.source_review]
```

Quy tắc:

- `context.reason` rỗng hoặc không có (user vào trực tiếp) → KHÔNG render block.
- Reason là plain text 1-3 câu, KHÔNG markdown đặc biệt.
- `source_review` luôn đi kèm reason (cùng block).

## Pre-confirmed Flow

Khi payload có `context.pre_confirmed: true` (user đã confirm tại track Bước 4 với full all-changes diff preview + reply "y"):

1. **SKIP** ask "Xác nhận? (y/sửa/huỷ)" → đi thẳng ghi file.
2. **VẪN HIỂN THỊ**: block "Lý do điều chỉnh" + bảng diff + cảnh báo (nếu có) để user trace.
3. Kèm 1 dòng cuối: `(Đã được confirm tại track Bước 4. Ghi file ngay.)`

Điều kiện `pre_confirmed: true` hợp lệ:

- Track Bước 4 đã hiển thị **all-changes diff** (gom theo skill đích).
- User reply "y" cho all-changes preview.
- Thiếu 1 trong 2 → skill nhận vẫn chạy default flow (hỏi confirm).

Pre-confirmed bypass CHỈ skip ask "y/sửa/huỷ". Vẫn ghi log + hiển thị báo cáo bình thường.
