# Mode UPDATE-RESOURCE: sửa resource

### Phase 1: Hiển thị state hiện tại

Đọc `resources.md`. Hiển thị:

```
Resource hiện tại
| Mục            | Giá trị                                |
|----------------|----------------------------------------|
| Solo Profile   | Tuấn, 10h/tuần, skills: Python, viết   |
| Công cụ        | 4 (Notion, GitHub, Figma, Slack)       |
| Tài liệu/KB    | 2 (brief.pdf, Drive folder)            |
| Ngân sách      | 5M VND (đã chi: 1.2M)                  |
| Rủi ro         | 1 (Skill Figma còn yếu)                |
```

> **Legacy migrate**: Nếu file vẫn còn section `## Nhân sự (Vai trò & Trách nhiệm)` schema cũ, agent tự convert: lấy dòng đầu (hoặc dòng có khả dụng cao nhất) thành Solo Profile, drop các cột "Liên lạc", "Người quản lý", "Khả dụng %", "Actions". Thông báo user "Đã migrate schema cũ sang Solo Profile."

> **Legacy migrate 6 cột**: Nếu bảng `## Công cụ` có 4 cột (Tên công cụ, Khi nào dùng, Mục đích, Resource) → auto-migrate sang 6 cột (thêm Cách dùng + Ghi chú trống). Nếu bảng `## Tài liệu & Knowledge Base` có 4 cột (Tên/Loại, Vị trí, Mục đích, Status) → map: Tên/Loại → Tên, Vị trí → Resource, Mục đích giữ, Status → Ghi chú. Thêm Khi nào dùng + Cách dùng trống. Thông báo user "Đã migrate bảng Công cụ/KB sang schema 6 cột. Vui lòng điền Cách dùng và Ghi chú."

### Phase 2: Hỏi user muốn update gì

Menu:

1. Sửa Solo Profile (capacity h/tuần, skills)
2. Thêm/sửa/xoá công cụ
3. Thêm/sửa/xoá tài liệu / knowledge base
4. Update ngân sách
5. Update rủi ro/thiếu hụt

User chọn nhiều cùng lúc được (vd "1, 4").

### Phase 3: Thu thập thay đổi

Tuỳ lựa chọn user. Vd nếu chọn 1:

- Hỏi capacity mới: "Tuần này bạn còn dành được X h/tuần không?"
- Hỏi có thêm skill mới sau quá trình thực thi (vd học được kỹ năng mới)?

### Phase 4: CONFIRM diff (BẮT BUỘC)

```
Thay đổi sắp áp dụng
| Loại                  | Trước              | Sau                  |
|-----------------------|--------------------|----------------------|
| Sửa capacity          | 10h/tuần           | 6h/tuần              |
| Thêm skill            | Python, viết       | Python, viết, SQL    |
| Sửa ngân sách         | 5M VND             | 8M VND               |

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Cảnh báo cross-check:
  - Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done → cảnh báo "Capacity giảm, scope có thể không kịp deadline. Cân nhắc dời actions sang sau hoặc giảm scope qua `/okr plan update`."
  - Skill mới thêm có thể unlock action TBD trước đó (vd "Skill SQL mới: action A007 trước đây skip vì thiếu skill có thể làm lại").
3. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."
