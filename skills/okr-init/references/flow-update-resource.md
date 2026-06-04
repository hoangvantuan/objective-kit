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

> **Legacy migrate**: Nếu gặp schema cũ (Nhân sự, bảng 4 cột Công cụ/KB), chạy migration theo `data-format.md` section "Migration schema cũ → 6 cột" và "Lưu ý dữ liệu cũ". Thông báo user kết quả migrate.

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

[Nếu thay đổi gồm tạo file mới ở vị trí lạ, vd nâng nội dung thành context/ hoặc tạo file ad-hoc thuộc update-resource, in thêm dòng neo TRƯỚC "Xác nhận?":]
> Sẽ tạo `.okr/context/<slug>.md`, neo vào `context/index.md` entry, vai trò `<1 câu>`.

Xác nhận? (y / sửa / huỷ)
```

### Phase 5: Áp dụng + cảnh báo

1. Ghi đè `resources.md`, update `last_updated`.
2. Cảnh báo cross-check:
  - Capacity giảm rõ rệt (vd <60% so với cũ) mà còn nhiều actions chưa done → cảnh báo "Capacity giảm, scope có thể không kịp deadline. Cân nhắc dời actions sang sau hoặc giảm scope qua `/okr plan update`."
  - Skill mới thêm có thể unlock action TBD trước đó (vd "Skill SQL mới: action A007 trước đây skip vì thiếu skill có thể làm lại").
- **Phân loại đúng nhà** (ranh giới `schemas.md` "Context layer"): nếu user mô tả một *con trỏ tới nguồn DÙNG* (link, file cần tra) → vào `## Tài liệu & Knowledge Base`. Nếu là *nội dung cross-cutting do dự án tạo* (glossary, playbook) → KHÔNG nhét vào resources; tạo `context/<slug>.md` + entry `context/index.md` (owner = okr-init) kèm dòng confirm gate ở Phase 4.
3. Hiển thị: "Đã update resource. Cảnh báo: [...]. Chạy `/okr` để xem tiến độ."
