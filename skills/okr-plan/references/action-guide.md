# Hướng dẫn viết Action

Action trả lời câu hỏi: "Cần làm gì cụ thể để đạt Key Result?"

## 5 tiêu chí bắt buộc

Mỗi action phải trả lời đủ 5 câu:

| # | Câu hỏi | Field tương ứng | Ví dụ tốt | Ví dụ xấu |
|---|---------|-----------------|-----------|-----------|
| 1 | **Ai làm?** | `pic` | "An (Product)" | "" (trống) |
| 2 | **Làm cái gì?** | `title` + `description` + `## Output/Deliverable` | "Viết report phân tích 5 đối thủ → file competitor-analysis.xlsx" | "Nghiên cứu thị trường" |
| 3 | **Deadline khi nào?** | `due_date` | "2026-11-15" | "càng sớm càng tốt" |
| 4 | **Xong trông thế nào?** | `## Definition of Done` + `## Tiêu chí chất lượng` | "Report có ≥5 đối thủ, mỗi đối thủ ≥3 chiều so sánh, data source ghi rõ" | "Done khi xong" |
| 5 | **Ước lượng bao lớn?** | `effort` | "m (1-2 ngày)" | không ghi |

## Effort (ước lượng độ lớn)

| Value | Thời gian | Khi nào dùng |
|-------|-----------|-------------|
| `xs` | < 1 giờ | Config, fix nhỏ, gửi email |
| `s` | 1-4 giờ | Viết 1 function, 1 trang doc |
| `m` | 1-2 ngày | 1 feature nhỏ, 1 report |
| `l` | 3-5 ngày | 1 module, design hoàn chỉnh |
| `xl` | > 1 tuần | Cảnh báo overrun cao, ưu tiên tách. Nếu giữ → BẮT BUỘC Checkpoints. |

### Quy tắc cho effort = xl

Solo user thường underestimate task xl: thực tế overrun gấp 1.5-2× ước lượng ban đầu (không có review giữa kỳ → không phát hiện trượt sớm). Quy trình bắt buộc khi user định ghi `effort: xl`:

1. **Hỏi tách trước**: "Task này khá lớn (>1 tuần). Có thể tách thành 2-3 tasks `m` hoặc `l` không?". Đề xuất ngay split candidate dựa trên DoD nếu thấy được.
2. **Nếu user giữ xl** → BẮT BUỘC tạo body section `## Checkpoints` với ≥2 mục, mỗi mục:
   - 1 dòng mô tả mốc đạt được (vd "Phase 1: Spec hoàn chỉnh")
   - Ngày deadline cụ thể (vd "by 2026-11-15")
   - Format: `- [ ] [Mô tả mốc] (by YYYY-MM-DD)`
3. **Nếu user từ chối tạo Checkpoints** → action không hợp lệ. Quay lại bước 1 (yêu cầu tách).

Track đọc Checkpoints khi chạy `light` / `deep`: mốc nào quá hạn mà chưa tick → cảnh báo "Action AXXX trượt checkpoint N". User có thể update tiến độ checkpoint riêng (tick checkbox) trước khi đánh action `done`.

Ví dụ Checkpoints hợp lệ:

```markdown
## Checkpoints
- [ ] Phase 1: Khảo sát user xong (by 2026-11-08)
- [ ] Phase 2: Spec MVP draft (by 2026-11-15)
- [ ] Phase 3: Review + finalize (by 2026-11-22)
```

## Priority (ưu tiên)

| Value | Khi nào dùng | Ví dụ |
|-------|-------------|-------|
| `critical` | Chặn toàn bộ dự án nếu không xong | Setup CI/CD trước khi team bắt đầu code |
| `high` | Cần xong sớm, nhiều task phụ thuộc | Thiết kế database schema |
| `medium` | Quan trọng nhưng không chặn ai | Viết docs, test coverage |
| `low` | Nice-to-have, làm khi rảnh | Refactor code cũ |

## Verify output

Bộ skill OKR mặc định **user tự verify qua DoD checklist**: mỗi tiêu chí trong `## Definition of Done` phải đo được, check được. Action xong khi mọi item DoD tick xanh. Không cần field `verifier` riêng (đã bỏ khỏi schema).

Hệ quả khi viết DoD:
- Mỗi item DoD phải verify được không cần ai khác (vd: "Test suite chạy ≥80% pass", "File xuất hiện tại path X", "Số liệu trong dashboard đạt ngưỡng Y").
- Tránh DoD mơ hồ kiểu "Code đẹp", "Doc tốt" (không đo được).
- Nếu output thực sự cần người khác review (rare cho solo) → ghi rõ trong `## Definition of Done` bằng item dạng "Reviewed by [tên]".

## Anti-patterns (action xấu)

| Pattern xấu | Vấn đề | Cách sửa |
|------------|--------|---------|
| "Nghiên cứu thêm" | Không có output đo được | Hỏi: "Output cụ thể là gì? Ai đọc/dùng output?" |
| "Cải thiện X" | Không rõ mức nào là đủ | Hỏi: "Cải thiện từ bao nhiêu đến bao nhiêu?" |
| "Hỗ trợ team Y" | Không rõ scope, ai measure | Hỏi: "Hỗ trợ bằng cách nào cụ thể? Deliverable?" |
| Action trùng DoD | DoD lặp lại title | DoD phải cụ thể hơn title: checklist verify được |
| Effort xl không tách | Task quá lớn, khó track | Tách thành 2-3 tasks m hoặc l |

## Ongoing type: khi nào tạo action?

Ongoing type chủ yếu dùng `## Practices` (hành động lặp lại) trong plan.md. Nhưng CÓ THỂ tạo action file khi:

- Cần task cải thiện KI đang warning/critical (vd: "Mua đồ tập gym" để cải thiện KI tập thể dục)
- Task one-off hỗ trợ duy trì (vd: "Đặt lịch khám sức khoẻ định kỳ", "Setup app theo dõi giấc ngủ")
- Task setup hệ thống tracking (vd: "Mua cân thông minh kết nối Health app")

Actions cho Ongoing tuân quy tắc bình thường. Milestones không bắt buộc (có thể gán milestone = "Cải thiện" hoặc bỏ trống).
