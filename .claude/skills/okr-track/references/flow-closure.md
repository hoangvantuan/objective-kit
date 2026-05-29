# Mode CLOSURE (mọi action done)

Đọc mở rộng: **đọc cả `actions/archive/**` (tổng kết toàn bộ actions đã hoàn thành) + **đọc tất cả `log/**` (tổng kết toàn period).

Như deep + thêm:

1. Tính tổng kết toàn period: KR đạt vs target, % thời gian, lessons learned. Soi actions `effort: xl` đã done: mốc `## Checkpoints` nào từng trượt deadline → rút bài học ước lượng cho `## Lessons`.
2. Hỏi user: chuyển status objective → `completed`, hay tạo follow-up project?
3. Nếu user đồng ý → delegate sang `okr-init` mode `update-objective` để đổi `status: completed`.
4. Xử lý inbox còn lại (nếu có).
5. Ghi log vào `log/YYYY-MM-DD.md` với `type: [closure]` (hoặc union nếu file ngày đã có). Kèm section `## Lessons`.
6. Update `plan.md` frontmatter: `last_review_date: today`, `last_track_date: today`.
