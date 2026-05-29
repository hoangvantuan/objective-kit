# Quality Gate (shared cho okr-init + okr-plan)

Mỗi khi user trả lời 1 câu hỏi từ skill (init: tạo objective/KR/resource; plan: chọn initiative/action/effort), agent tự kiểm tra 3 câu **không hiển thị cho user**:

1. **Đủ cụ thể?** Câu trả lời có thể chuyển thành KR/KI đo được (init) hoặc action có deliverable rõ (plan) không?
2. **Giả định ẩn?** User có bỏ qua constraint quan trọng nào (capacity, deadline, dependency, skill)?
3. **Mâu thuẫn?** Câu trả lời có xung đột với context trước (capacity, timeline, objective, KR đã chốt)?

## Hành vi theo kết quả

| Kết quả | Hành vi |
|---------|--------|
| Cả 3 pass | Đi tiếp câu hỏi kế |
| Bất kỳ fail | **TRƯỚC** follow-up, in 1 dòng `(Mình đào sâu thêm vì <lý do cụ thể>)` để user hiểu vì sao bị hỏi sâu. Sau đó dùng Deepening Techniques phù hợp (mỗi skill có bảng kỹ thuật riêng). |
| User trả lời "chưa biết" / "để sau" | Ghi nhận, đánh dấu `⚠️ TBD`. Phase confirm BẮT BUỘC nhắc lại các field TBD trước khi ghi. |
| User tỏ ra sốt ruột | Giảm độ sâu, chỉ giữ câu 1 (đủ cụ thể?). Không skip hoàn toàn. |
| User paste từ doc có sẵn (brief, PRD, spec) | Tóm tắt lại bằng lời mình, hỏi "tôi hiểu đúng chưa?" thay vì nhận nguyên xi. |

## Nguyên tắc

- Quality Gate là **internal check**, không hiển thị 3 câu cho user.
- Dòng `(Mình đào sâu thêm vì <lý do>)` chỉ in khi có fail, đặt **trước** câu hỏi follow-up trên cùng turn.
- Lý do trong ngoặc phải **cụ thể**, không chung chung. Vd:
  - ✅ `(Mình đào sâu thêm vì "tăng doanh thu" chưa nói kênh nào, sản phẩm nào.)`
  - ❌ `(Mình cần thêm thông tin)` (quá chung)
- Mỗi skill có thể bổ sung thêm rule riêng (vd plan check capacity vs effort), nhưng vẫn dựa trên 3 câu core ở đây.
