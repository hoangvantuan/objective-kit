# Changelog

Lịch sử **phát triển** OKR Harness (source code, kiến trúc, skill definitions).

> Lịch sử **cải tiến** harness tại project đích: xem `.claude/CHANGELOG.md` của project đó.

| Ngày       | Thay đổi                                                                      | Đối tượng                       | Lý do                                        |
| ---------- | ----------------------------------------------------------------------------- | ------------------------------- | -------------------------------------------- |
| 2026-05-29 | Đợt 10: gom logic phân tích về 1 nơi. Dời `metrics.md` okr-track→okr-shared (canonical đọc dùng chung, bỏ "Cập nhật SOT"+"Log format" trùng data-format). okr-analyze hết chép công thức + sửa priority lệch (4 bước→trỏ canonical 5 bước). track dùng `analysis`, không tự tính metrics/root cause. Gom 5 tín hiệu capacity vào metrics.md. Dọn breadcrumb "Nguồn gốc" + section rỗng. Thêm checkpoint slip vào analyze/deep/closure. Đồng bộ bảng SOT trong CLAUDE.md với canonical. | okr-shared (metrics.md mới + SKILL), okr-analyze, okr-track (flow-shared/deep/light/closure/inbox, data-format, SKILL), okr-init/data-format, okr-harness/skill-contract, CLAUDE.md | Review prompt-master: gỡ trùng/mâu thuẫn analyze↔track, đưa shared logic về okr-shared đúng hub-and-spoke, chống drift công thức. Verify 6 agent: 5 pass, 0 fail |
| 2026-05-29 | Skill-only: xoá `.claude/agents/` (3 agent) + agent team; gộp analyst/builder/tracker vào skill; thêm `okr-analyze`; orchestrator chạy inline; agent-contract → skill-contract; flows.md bỏ team | .claude/agents/ (xoá), okr-harness, okr-analyze, okr-init/plan/track/capture/shared, CLAUDE.md | Không lệ thuộc runtime agent team, deploy được mọi project Claude Code (theo yêu cầu user) |
| 2026-05-19 | Đóng gói harness: hấp thụ AGENTS.example.md vào okr-harness, tạo CHANGELOG.md | okr-harness SKILL.md, CLAUDE.md | Self-contained, deploy chỉ cần copy .claude/ |
| 2026-05-18 | Tách CLAUDE.md / AGENTS.example.md                                            | CLAUDE.md, AGENTS.example.md    | Tách dev guide vs runtime template           |
| 2026-05-18 | Migrate skills/ → harness                                                     | 6 skills, 26 references         | Self-contained, tách skill nhỏ dễ maintain   |
| 2026-05-18 | Khởi tạo harness                                                              | 3 agents + orchestrator         | Tăng tốc phân tích + tracking song song      |

