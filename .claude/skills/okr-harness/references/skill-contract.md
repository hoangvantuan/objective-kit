# Skill Contract: Input/Output Specification

Mô tả input/output từng skill khi orchestrator chạy inline. Đây là "hợp đồng" giữa các bước trong cùng một agent: bước trước sản xuất gì, bước sau cần gì. Không phải dispatch sang sub-agent.

## okr-analyze

### Input

| Param | Bắt buộc | Mô tả |
|-------|---------|-------|
| okr_path | Có | Đường dẫn tới `.okr/` |
| focus | Không | Lĩnh vực focus: "full" (default), "progress", "issues", "priority" |
| max_items | Không | Số priority items (default 2 cho dashboard, 3 cho track) |
| horizon_days | Không | Cửa sổ deadline (default 3 cho dashboard, 7 cho track) |
| mode | Không | "default" hoặc "deep" (deep: thêm root cause + log history) |

### Output (markdown structured)

```markdown
## Dashboard
[Objective summary 1 dòng]
[KR/KI progress bảng: ID | Tên | Current | Target | % | Status | Trend]
[Action summary: X done / Y total, Z blocked, W overdue]

## Issues (nếu có)
[Severity] [Type]: [description]
  Evidence: [file:line hoặc giá trị]
  Recommendation: [hành động, áp dụng qua: okr-init/okr-plan/okr-track]

## Root cause (chỉ mode deep)
[Mỗi issue cần cải thiện: nhân gốc vs duyên, tách triệu chứng]

## Priority hôm nay
→ AXXX: Title (lý do ≤10 từ)

## Recommendations (nếu có issues)
[Đề xuất, chỉ rõ skill đích + mode]
```

## okr-init

### Input

| Param | Bắt buộc | Mô tả |
|-------|---------|-------|
| okr_path | Có | Đường dẫn `.okr/` |
| mode | Có | new, update-objective, update-resource, pre-confirmed |
| context | Không | User request text hoặc analysis insights |
| changes | Khi pre-confirmed | Danh sách thay đổi cụ thể (field, from, to) |
| reason | Khi pre-confirmed | Root cause text từ deep review |

### Output

```yaml
status: success | cancelled     # cancelled = user từ chối confirm
files_changed:
  - path: string
    action: created | modified
summary: string                 # tóm tắt 1-2 câu
```

## okr-plan

Input/Output giống okr-init (mode: new, update, pre-confirmed). SOT: `plan.md` + `actions/`.

## okr-track

### Input

| Param | Bắt buộc | Mô tả |
|-------|---------|-------|
| okr_path | Có | Đường dẫn `.okr/` |
| mode | Có | light, deep, inbox-only, closure, trace |
| analysis | Không | Output từ okr-analyze (khi track light hoặc deep) |

### Output

```yaml
status: success | partial       # partial = một số thao tác skip
changes:
  - file: string
    field: string
    old_value: any
    new_value: any
structural_changes:             # chỉ khi deep hoặc inbox: thay đổi cần okr-init/okr-plan áp dụng
  - apply_via: okr-init | okr-plan
    mode: string
    changes: [...]
    context:
      reason: string
      pre_confirmed: boolean
archived_actions: [AXXX, ...]
inbox_processed: number
next_actions:                   # top 3
  - id: AXXX
    title: string
    reason: string
```

## Chuyển skill inline (thay cho dispatch)

Khi một flow cần kết quả của skill khác, cùng agent đọc tiếp SKILL.md kia và thực thi:

- Track deep phát hiện cần sửa cấu trúc → đọc `okr-init`/`okr-plan` mode pre-confirmed, mang theo `context.reason` + `pre_confirmed: true`. Xem `okr-shared/references/delegate-protocol.md`.
- Dashboard/track cần metrics → đọc `okr-analyze` trước, dùng output (`analysis`) làm input. Track/dashboard KHÔNG tự tính lại metrics hay root cause; chỉ tự tính khi chạy KHÔNG qua analyze (vd inbox-only độc lập).

Quy tắc:
- Truyền `okr_path` tuyệt đối, không relative.
- Mang theo context đã có giữa các bước (analysis, user confirm) để không hỏi lại.
- Tôn trọng SOT ownership: mỗi field chỉ skill được phép mới ghi (xem `okr-shared/references/sot-ownership.md`).
</content>
