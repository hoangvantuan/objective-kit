# Mode CLOSURE (mọi action done)

Đọc mở rộng: **đọc cả `actions/archive/**` (tổng kết toàn bộ actions đã hoàn thành) + **đọc tất cả `log/**` (tổng kết toàn period).

Như deep + thêm:

1. Tính tổng kết toàn period: KR đạt vs target, % thời gian, **tốc độ hoàn thành** (từ `completed_date` trong `actions/archive/`). Soi actions `effort: xl` đã done: mốc `## Checkpoints` nào từng trượt deadline → ghi vào `## Tổng kết period` (phần rút kinh nghiệm ước lượng cho period này).
2. Hỏi user: chuyển status objective → `completed`, hay tạo follow-up project?
3. Nếu user đồng ý → delegate sang `okr-init` mode `update-objective` để đổi `status: completed`.
4. Xử lý inbox còn lại (nếu có).
5. Ghi log vào `log/YYYY-MM-DD.md` với `type: [closure]` (hoặc union nếu file ngày đã có). Kèm section `## Tổng kết period` (KR cuối, % thời gian, tốc độ, điểm ước lượng từng trượt). Đây là tổng kết **cho period này trong log**, KHÔNG phải kho bài học tái dùng.
6. Update `plan.md` frontmatter: `last_review_date: today`, `last_track_date: today`.
7. **Gợi ý rút bài học**: in 1 dòng "Muốn rút bài học tái dùng từ period này? (chạy okr-retro)". Bài học sống lâu dài (cải tiến skill / tri thức project) thuộc `okr-retro` → `.okr/lessons/`, KHÔNG ghi vào log closure. User đồng ý → chạy `okr-retro` inline.
