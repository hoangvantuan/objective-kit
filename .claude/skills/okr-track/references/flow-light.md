# Mode LIGHT (cập nhật progress nhanh)

Phạm vi: CHỈ progress fields. Không sửa cấu trúc.

**Project type:**

1. **Sync pull** (nếu có action với `external_ids`): chạy External Sync pull flow (xem `references/data-format.md` section "External Sync"). Diff status hiển thị trước bảng hỏi update. User confirm sync → merge vào danh sách thay đổi step 1. Không có `external_ids` → skip.
2. Hỏi user có thay đổi: KR current, action status (pending/doing/done/blocked), blocker mới, output (nếu user chủ động cung cấp).
  - User cung cấp output cho action đang `doing` hoặc `done` → ghi đè section `## Output/Deliverable` trong action file. Không cần chờ mark done.

1b. **Checkpoint update** (nếu có action status=doing với `effort: xl` và section `## Checkpoints`): hiển thị checklist, hỏi user tick mốc đã đạt. Mốc quá hạn chưa tick → cảnh báo "Action AXXX trượt checkpoint N".
2. CONFIRM trước khi ghi (đếm số field thay đổi rồi chọn UI):

- **Nếu ≤2 field thay đổi** → confirm 1 dòng (giảm friction quick check-in):
  ```
  KR1: 40>50, A003 done. (y/sửa/huỷ)
  ```
  hoặc:
  ```
  A005 blocked (chờ approve). (y/sửa/huỷ)
  ```
- **Nếu ≥3 field thay đổi** → confirm bảng đầy đủ:
  ```
  Thay đổi sắp ghi
  - KR1.current: 40 > 50
  - A003.status: doing > done
  - A003.output: [report.md](docs/report.md), https://docs.google.com/...
  - A005.status: doing > blocked, lý do: chờ approve
  Xác nhận? (y/sửa/huỷ)
  ```
- Cả 2 nhánh: user trả lời `sửa` → hỏi field nào cần sửa, sửa xong CONFIRM lại. `huỷ` → bỏ toàn bộ thay đổi, kết thúc light mode.

3. Áp dụng:
  - Ghi đè progress: `objective.md` (KR.current, KR status), `plan.md` (counters + `last_track_date: today`), `actions/*.md` (frontmatter status).
  - Append log: `.okr/log/YYYY-MM-DD.md`. File ngày đã có → append section mới.

3b. **Sync push** (nếu có action vừa thay đổi status + có `external_ids`): push status mới lên tool ngoài theo External Sync push flow. Báo kết quả.
4. **Nhắc output** (nếu có action chuyển sang `done` trong lần này): với mỗi action vừa done, kiểm tra body section `## Output/Deliverable`. Nếu section vẫn là mô tả dự kiến (do okr-plan tạo), hỏi:

  ```
  A003 done. Output là gì? (file path, link, text... hoặc "skip")
  ```

  - User cung cấp output → ghi đè section `## Output/Deliverable` trong action file bằng output thực tế (free-form markdown). Append output vào log entry (format: `A003.output:` + danh sách items).
  - User nói "skip" → giữ nguyên section dự kiến, không ghi output vào log.
  - Nếu section đã được ghi đè trước đó (user đã ghi output giữa chừng) → skip, không hỏi lại.

5. **Archive actions done**: chạy Archive Rules (xem `okr-shared/references/schemas.md` section "Archive Rules"). Sau đó:

  - **Re-render bảng Roadmap**: đọc lại tất cả `actions/*.md` (không archive), sinh lại toàn bộ bảng per-milestone trong `plan.md` body. Format xem `okr-shared/references/schemas.md` section "Roadmap format".

6. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
7. Đề xuất next action: chọn theo thuật toán [action-priority.md](../../okr-shared/references/action-priority.md) với `max_items=3`, `horizon_days=7`. Mỗi gợi ý kèm 1 lý do ≤10 từ.

**Ongoing type:**

1. Hiển thị KI hiện tại (tên, ngưỡng, current, status).
2. Hỏi user update từng KI: "KI1 (Tập thể dục, hiện tại: 2, ngưỡng: ≥3). Tuần này bao nhiêu?"
3. Tính status mới theo logic: healthy (≥ ngưỡng), warning (< ngưỡng, chênh < 20%), critical (< 80% ngưỡng). Xem `../../okr-shared/references/metrics.md`.
4. **Update practice streak**: đọc `plan.md` body section `## Practices`.
  - **Fallback (Minor2):** nếu `plan.md` không tồn tại HOẶC body không có heading `## Practices` HOẶC section rỗng (không có `### PN:`), in 1 dòng `(Ongoing chưa có practices. Skip step 4. Tạo practices qua /okr plan để track streak.)` rồi đi tiếp step 5. KHÔNG báo lỗi.
  - Với mỗi practice (`### PN: <Tên>` kèm field `frequency`, `target_count`, `current_streak`, `ki_link`):
    - Tính chu kỳ vừa qua dựa trên `frequency` (vd `weekly` = tuần vừa rồi, `daily` = hôm qua). Hỏi: "Practice **P1: Tập thể dục** (target ≥3 lần/tuần, streak hiện tại: 2). Tuần vừa rồi đạt target chưa? (y = đạt → streak +1 / n = không đạt → reset streak về 0 / skip = bỏ qua không update)".
    - User trả lời `y` → `current_streak += 1`. `n` → `current_streak = 0`. `skip` → giữ nguyên.
    - Nếu practice `ki_link` đã có KI tương ứng vừa update ở step 2-3, hiển thị inline để user thấy liên kết: `(P1 → KI1 healthy)`.
5. Nếu có action files (task cải thiện KI) → hỏi update status (pending/doing/done/blocked) như Project.
6. CONFIRM trước khi ghi (tương tự Project: ≤2 field → 1 dòng, ≥3 field → bảng). Bao gồm cả `current_streak` thay đổi vào diff list.
7. Áp dụng: ghi đè `objective.md` (KI current, status), ghi đè `plan.md` body section `## Practices` (chỉ field `current_streak` cho practice nào thay đổi) + frontmatter `last_track_date: today`. Nếu có actions → cập nhật `plan.md` counters + `actions/*.md`.
8. Append log: ngoài KI/action, ghi cả practice streak thay đổi (vd `## Thay đổi → P1.current_streak: 2 > 3`).
9. **Xử lý inbox** (nếu có items pending): chạy Inbox Processing Flow (xem Phase 5).
10. Đề xuất:
  - Nếu KI warning/critical → gợi ý tạo action cải thiện qua `/okr plan`.
  - Nếu practice streak vừa reset về 0 → cảnh báo "Practice P1 đã reset streak, KI1 có thể giảm tuần tới. Cân nhắc điều chỉnh practice (giảm target_count, đổi thời gian) qua `/okr plan update`".
  - Nếu có actions đang mở → chọn theo [action-priority.md](../../okr-shared/references/action-priority.md) (`max_items=3`, `horizon_days=7`). Hiển thị SAU gợi ý KI/practice.
