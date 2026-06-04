# Data Flows chi tiết (skill-only inline)

Toàn bộ luồng do **một agent** thực thi: orchestrator đọc state, rồi đọc tiếp SKILL.md phù hợp và chạy theo flow. "→ skill X" nghĩa là cùng agent đọc và thực thi skill X, không spawn sub-agent.

## 1. Dashboard flow

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: /okr-harness
    H->>H: Preload state (.okr/ frontmatter)
    H->>H: → okr-analyze: đọc song song objective/plan/resources/actions/inbox/log
    H->>H: → okr-analyze: tính metrics (KR%, trend, action health)
    H->>H: → okr-analyze: phát hiện issues + xếp priority
    H->>U: Render dashboard + nhắc review nếu cần
```

**okr-analyze output**: dashboard, metrics, issues, priority top 2, recommendations.

## 2. Track light flow

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "track" / "cập nhật"
    H->>H: → okr-analyze (focus: progress, overdue, blocked)
    H->>H: → okr-track light, dùng analysis làm input
    H->>H: Sync pull (nếu external_ids)
    H->>U: Hỏi: KR.current? Action status?
    U->>H: Updates
    H->>U: CONFIRM bảng thay đổi
    U->>H: OK
    H->>H: Ghi SOT + sync push + archive + inbox
    H->>U: Tóm tắt + next action
```

## 3. Deep review flow (chuỗi tuần tự inline, KHÔNG agent team)

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "review sâu"
    H->>H: → okr-analyze deep: đọc toàn bộ .okr/ + log history
    H->>H: → okr-analyze deep: root cause (≥3 "tại sao?")
    H->>H: → okr-track deep: nhận analysis
    H->>U: Root cause + đề xuất điều chỉnh
    U->>H: Feedback / chọn
    H->>U: All-changes confirm (gom theo skill đích)
    U->>H: Approve
    H->>H: → okr-track: ghi progress fields + log review
    opt Có thay đổi cấu trúc
        H->>H: → okr-init / okr-plan (pre_confirmed): hiển thị reason + diff, ghi ngay
    end
    H->>U: Tóm tắt toàn bộ thay đổi
```

**Thay đổi cấu trúc** (mỗi item, từ track deep sang init/plan):
- `apply_via`: "okr-init" / "okr-plan"
- `mode`: "update-objective" / "update" / "update-resource"
- `changes`: danh sách (field, from, to)
- `context.reason`: root cause text
- `pre_confirmed`: true (đã confirm tại all-changes preview)

> Trước đây bước này là agent team (analyst + tracker qua SendMessage, builder spawn riêng). Skill-only gộp thành chuỗi tuần tự một agent.

## 4. Init/Plan flow

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "tạo mục tiêu" / "tạo plan"
    H->>H: → okr-init / okr-plan (mode tương ứng)
    H->>U: Thu thập yêu cầu (tương tác nhiều bước)
    U->>H: Trả lời
    H->>U: CONFIRM bảng tóm tắt
    U->>H: OK
    H->>H: Ghi files
    H->>U: Tóm tắt
```

## 5. Capture flow

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "ghi nhanh: ý tưởng ABC"
    H->>H: → okr-capture: phân loại type
    H->>H: → okr-capture: ghi inbox/YYYY-MM-DD-HHmm-slug.md
    H->>U: "Đã ghi vào inbox"
```

## 6. Inbox flow

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "xử lý inbox"
    H->>H: → okr-track inbox-only: đọc inbox pending + staleness
    H->>U: Cảnh báo cũ > 30 ngày
    H->>U: Gợi ý xử lý per item
    U->>H: Quyết định per item
    H->>H: Xử lý: blocker → tự ghi, thought → action/log/context/giữ
    opt action / resource item
        H->>H: → okr-init / okr-plan: tạo action / update resource
    end
    H->>U: Báo cáo inbox đã xử lý
```

`thought` có 4 nhánh xử lý canonical ở `okr-track/references/flow-inbox.md`: tạo action, append log, nâng thành `context/<slug>.md` + `context/index.md`, hoặc giữ inbox.

## 7. Retro flow (rút bài học)

```mermaid
sequenceDiagram
    participant U as User
    participant H as Harness (1 agent)

    U->>H: "rút bài học" / đồng ý gợi ý cuối flow
    H->>H: → okr-retro: đọc lessons/index.md (dedup)
    H->>H: → okr-retro: quét hội thoại phiên → ứng viên bài học
    H->>H: → okr-retro: phân loại A/B + soạn essence + dedup
    H->>U: Bảng ứng viên + bài nghi lỗi thời
    U->>H: Tick/cắt/sửa
    H->>H: Ghi file lessons/{skill,project}/ + cập nhật index.md
    H->>U: Báo: ghi mới/cập nhật/lỗi thời + nhắc port loại A
```

Record-only: `okr-retro` KHÔNG sửa file skill. Loại A là hàng đợi port thủ công về repo gốc. Chỉ user chủ động (hoặc đồng ý gợi ý cuối flow) mới chạy.
