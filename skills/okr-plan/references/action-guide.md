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
| `xl` | > 1 tuần | Nên tách thành nhiều actions nhỏ hơn |

Nếu effort = `xl`, hỏi user: "Task này khá lớn. Có thể tách thành 2-3 tasks nhỏ hơn không?"

## Priority (ưu tiên)

| Value | Khi nào dùng | Ví dụ |
|-------|-------------|-------|
| `critical` | Chặn toàn bộ dự án nếu không xong | Setup CI/CD trước khi team bắt đầu code |
| `high` | Cần xong sớm, nhiều task phụ thuộc | Thiết kế database schema |
| `medium` | Quan trọng nhưng không chặn ai | Viết docs, test coverage |
| `low` | Nice-to-have, làm khi rảnh | Refactor code cũ |

## Verify output

Bộ skill OKR mặc định **user tự verify qua DoD checklist**: mỗi tiêu chí trong `## Definition of Done` phải đo được, check được. Action xong khi mọi item DoD tick xanh. Không cần field reviewer riêng trong frontmatter.

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
