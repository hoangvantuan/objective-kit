# Changelog

Lịch sử **phát triển** OKR Harness (source code, kiến trúc, skill definitions).

> Lịch sử **cải tiến** harness tại project đích: xem `.claude/CHANGELOG.md` của project đó.

| Ngày       | Thay đổi                                                                      | Đối tượng                       | Lý do                                        |
| ---------- | ----------------------------------------------------------------------------- | ------------------------------- | -------------------------------------------- |
| 2026-05-29 | Skill-only: xoá `.claude/agents/` (3 agent) + agent team; gộp analyst/builder/tracker vào skill; thêm `okr-analyze`; orchestrator chạy inline; agent-contract → skill-contract; flows.md bỏ team | .claude/agents/ (xoá), okr-harness, okr-analyze, okr-init/plan/track/capture/shared, CLAUDE.md | Không lệ thuộc runtime agent team, deploy được mọi project Claude Code (theo yêu cầu user) |
| 2026-05-19 | Đóng gói harness: hấp thụ AGENTS.example.md vào okr-harness, tạo CHANGELOG.md | okr-harness SKILL.md, CLAUDE.md | Self-contained, deploy chỉ cần copy .claude/ |
| 2026-05-18 | Tách CLAUDE.md / AGENTS.example.md                                            | CLAUDE.md, AGENTS.example.md    | Tách dev guide vs runtime template           |
| 2026-05-18 | Migrate skills/ → harness                                                     | 6 skills, 26 references         | Self-contained, tách skill nhỏ dễ maintain   |
| 2026-05-18 | Khởi tạo harness                                                              | 3 agents + orchestrator         | Tăng tốc phân tích + tracking song song      |

