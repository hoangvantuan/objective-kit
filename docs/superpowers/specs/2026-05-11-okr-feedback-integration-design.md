# Design: Tích hợp 4 Feedback vào hệ thống OKR Skill

**Ngày**: 2026-05-11
**Scope**: Mở rộng schema plan.md, action frontmatter, resources.md
**Approach**: Mở rộng tối thiểu, không breaking change

## Bối cảnh

4 feedback từ user về cải thiện hệ thống OKR skill hiện tại:

1. **Roadmap bảng**: plan.md cần bảng tổng quan task thay vì chỉ list
2. **External tool sync**: Action cần ID bên thứ 3 để sync 2 chiều
3. **Resource mở rộng**: Thêm công cụ quản lý task, KB với hướng dẫn truy cập
4. **Hướng dẫn sử dụng resource**: Mọi resource cần đủ 4 field: mục đích, khi nào dùng, cách dùng, ghi chú

## Thiết kế chi tiết

### 1. Bảng Roadmap trong plan.md

**Vấn đề**: Roadmap hiện tại chỉ liệt kê action ID dưới milestone heading. Thiếu tổng quan nhanh về deadline, priority.

**Giải pháp**: Bảng auto-generated từ action files, nằm dưới mỗi milestone heading.

**Vị trí**: Section `## Roadmap`, mỗi milestone heading chứa bảng action bên dưới.

**Format**:

```markdown
## Roadmap

### M1: Research (target: 2026-05-20)

| ID | Task | Deadline | Priority | Notes |
|----|------|----------|----------|-------|
| [A001](actions/A001-research-market.md) | Nghiên cứu thị trường | 2026-05-15 | high | Cần xong trước A003 |
| [A002](actions/A002-competitor-analysis.md) | Phân tích đối thủ | 2026-05-18 | medium | |

### M2: Development (target: 2026-06-15)

| ID | Task | Deadline | Priority | Notes |
|----|------|----------|----------|-------|
| [A003](actions/A003-build-mvp.md) | Xây MVP | 2026-06-01 | critical | Depends: A001 |
| [A004](actions/A004-testing.md) | Viết test | 2026-06-10 | high | |
```

**Cột bảng (5 cột, solo mode)**:

| Cột | Nguồn dữ liệu | Ghi chú |
|-----|---------------|---------|
| ID | `id` frontmatter, link tới file action | Relative link: `actions/AXXX-slug.md` |
| Task | `title` frontmatter | Hiển thị nguyên văn |
| Deadline | `due_date` frontmatter | Format YYYY-MM-DD |
| Priority | `priority` frontmatter | critical, high, medium, low |
| Notes | `notes` frontmatter (field mới, optional) | Tối đa 50 ký tự |

**Cột Assignee (ẩn khi solo)**: Khi resources.md Solo Profile có >1 người (mở rộng tương lai), thêm cột Assignee giữa Task và Deadline. Nguồn: `pic` frontmatter.

**Quy tắc sinh bảng**:

- **Ai render**: `okr-plan` (khi tạo/update action) và `okr-track` (khi update status).
- **Sắp xếp**: Trong mỗi milestone, sắp theo Priority (critical > high > medium > low), rồi Deadline (sớm trước).
- **Lọc**: Chỉ hiển thị action chưa done. Action done đã archive, không xuất hiện trong bảng.
- **SOT**: Action files là SOT. Bảng là view được sinh lại mỗi lần update. Không sửa bảng trực tiếp.
- **Ongoing type**: Ongoing có thể có action không thuộc milestone nào. Nhóm dưới heading `### Chưa phân milestone`. Nếu Ongoing không có action nào, section `## Roadmap` chỉ chứa `## Practices` (không có bảng).

**Thay đổi action frontmatter** (field mới):

```yaml
notes: "string (optional, ≤50 ký tự)"
```

Field `notes` dùng cho ghi chú ngắn hiển thị trong bảng Roadmap. Guideline: giữ ngắn (khuyến nghị ≤50 ký tự, không hard limit). Ví dụ: "Cần xong trước A003", "Blocked by API access". Khác với `## Ghi chú` body (chi tiết dài). Không có notes thì cell trống trong bảng.

### 2. External IDs + Sync 2 chiều

**Vấn đề**: Action không có cách liên kết với task trên tool bên thứ 3 (Things 3, Notion, Jira...).

**Giải pháp**: Thêm field `external_ids` vào action frontmatter + logic sync trong okr-track.

#### 2a. Schema action frontmatter

```yaml
---
id: A001
title: "Nghiên cứu thị trường"
# ... các field hiện tại ...
depends_on: []
notes: ""
external_ids:              # MỚI - optional
  things3: "ABC-123"
  notion: "page-xyz-456"
---
```

**Quy tắc field `external_ids`**:

| Quy tắc | Chi tiết |
|---------|---------|
| Bắt buộc? | Không. Optional. Không có thì bỏ qua |
| Format | YAML map: `tool_key: "external_id_string"` |
| Key | Lowercase, match tên tool trong resources.md (vd: `things3`, `notion`) |
| Value | ID hoặc URL string từ tool bên thứ 3 |
| Nhiều tool | Hỗ trợ, mỗi key 1 tool |
| Ai tạo | `okr-plan` khi tạo action (user cung cấp), hoặc okr-track sau sync |
| Ai sửa | `okr-plan` (thêm/xóa tool), `okr-track` (cập nhật ID sau sync đầu tiên) |

#### 2b. Hành vi sync trong okr-track

**Pull (trước tracking)**:

1. Đọc tất cả active action có `external_ids`
2. Với mỗi tool trong `external_ids`:
   a. Tra resources.md `## Công cụ` → tìm cột "Cách dùng" để biết integration method (`skill: things-mac`, `mcp: notion-server`)
   b. Gọi skill/MCP tương ứng → lấy status task bên ngoài
3. So sánh status:
   - Nếu khác biệt: hiển thị "Tool X: A001 = completed, OKR: A001 = doing. Đồng bộ? (y/n)"
   - User confirm → update action status trong OKR
4. Nếu tool không có skill/MCP: skip, log cảnh báo

**Push (sau tracking)**:

1. Với mỗi action vừa thay đổi status + có `external_ids`:
   a. Tra integration method từ resources.md
   b. Gọi skill/MCP → push status mới lên tool ngoài
2. Báo kết quả: "Đã sync A001 → Things 3 (completed), A002 → Notion (doing)"
3. Nếu push thất bại: log lỗi, không retry tự động

**Mapping status**:

| OKR Status | Hướng map chung |
|------------|----------------|
| pending | Tool-specific "todo" / "not started" |
| doing | Tool-specific "in progress" |
| done | Tool-specific "completed" / "done" |
| blocked | Tool-specific "on hold" / "blocked" (nếu tool hỗ trợ) |

Mapping cụ thể theo từng tool nằm trong config của skill/MCP tương ứng. OKR system không hardcode.

**Nguyên tắc sync**:

- Mọi thay đổi từ tool ngoài cần user confirm trước khi ghi vào OKR
- Mọi push ra tool ngoài thực hiện tự động sau confirm tracking (không hỏi thêm)
- Tool không có integration → skip, không lỗi
- Sync là optional step, không block tracking flow nếu thất bại

### 3. Mở rộng resources.md

**Vấn đề**: Bảng Công cụ và Tài liệu & KB có cột khác nhau, thiếu "Cách dùng" và "Ghi chú".

**Giải pháp**: Thống nhất 6 cột cho cả 2 section.

#### Schema cũ vs mới

**`## Công cụ` (cũ → mới)**:

| Cũ | Mới |
|----|-----|
| Tên công cụ | **Tên** |
| Khi nào dùng | **Khi nào dùng** |
| Mục đích | **Mục đích** |
| Resource (URL/Account) | **Resource** |
| (không có) | **Cách dùng** (MỚI) |
| (không có) | **Ghi chú** (MỚI) |

**`## Tài liệu & Knowledge Base` (cũ → mới)**:

| Cũ | Mới |
|----|-----|
| Tên/Loại | **Tên** |
| Mục đích | **Mục đích** |
| Vị trí (Link/Folder/File) | **Resource** (hợp nhất) |
| Status | → gộp vào **Ghi chú** |
| (không có) | **Khi nào dùng** (MỚI) |
| (không có) | **Cách dùng** (MỚI) |

#### Format thống nhất (6 cột)

```markdown
## Công cụ

| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| Things 3 | Quản lý task hàng ngày | Daily planning, tracking | skill: things-mac | macOS app | Sync 2 chiều với OKR |
| Cursor | Code editor AI | Khi code, debug | Mở project → chat | cursor.sh | License Pro |
| Figma | Design UI | Khi thiết kế giao diện | mcp: figma-server | figma.com/team/xxx | Free plan |

## Tài liệu & Knowledge Base

| Tên | Mục đích | Khi nào dùng | Cách dùng | Resource | Ghi chú |
|-----|----------|-------------|-----------|----------|---------|
| API Docs v2 | Reference backend endpoints | Khi tích hợp API | Mở link → search endpoint | https://docs.example.com | Cần VPN |
| Meeting Notes | Ghi nhận quyết định | Sau mỗi meeting | GDrive → Folder meetings | gdrive://folder-id | Tự động từ Otter.ai |
| Design System | UI components reference | Khi build frontend | Figma → Library → Components | figma.com/file/xxx | v2.1, cập nhật monthly |
```

#### Quy tắc cột "Cách dùng"

| Loại resource | Nội dung cột "Cách dùng" |
|--------------|-------------------------|
| Tool có tích hợp OKR | `skill: <tên-skill>` hoặc `mcp: <tên-mcp-server>` |
| Tool không tích hợp | Quy trình ngắn 1 dòng (vd: "Mở project → chat") |
| Tài liệu online | Cách truy cập (vd: "Mở link → search endpoint") |
| Tài liệu local | Path hoặc cách mở (vd: "GDrive → Folder meetings") |
| Knowledge base | Cách query/search (vd: "skill: deepwiki → search topic") |

#### Migration từ schema cũ

`okr-init` mode `update-resource` lần đầu gặp schema cũ:

1. Detect: bảng Công cụ có 4 cột (thiếu Cách dùng, Ghi chú) → schema cũ
2. Auto-migrate: thêm 2 cột trống, báo user điền
3. Detect: bảng KB có cột "Status" riêng → gộp vào Ghi chú
4. Không mất dữ liệu, chỉ restructure cột

## Tổng hợp thay đổi

### Files schema cần sửa

| File | Thay đổi |
|------|---------|
| `skills/okr-plan/references/data-format.md` | Thêm field `notes`, `external_ids` vào action schema. Cập nhật plan.md Roadmap format thành bảng per milestone |
| `skills/okr-init/references/data-format.md` | Cập nhật resources.md schema: 6 cột thống nhất cho Công cụ và KB |
| `skills/okr-track/references/data-format.md` | Thêm sync flow description, reference external_ids |

### Files SKILL.md cần sửa

| File | Thay đổi |
|------|---------|
| `skills/okr-plan/SKILL.md` | Logic render bảng Roadmap khi tạo/update action |
| `skills/okr-track/SKILL.md` | Thêm phase sync (pull trước tracking, push sau tracking). Logic re-render bảng Roadmap khi update status |
| `skills/okr-init/SKILL.md` | Migration logic cho resources.md schema cũ. Thêm cột Cách dùng, Ghi chú khi tạo mới |

### File CLAUDE.md

| File | Thay đổi |
|------|---------|
| `CLAUDE.md` (project root) | Cập nhật bảng SOT: thêm `notes`, `external_ids` thuộc okr-plan. Ghi chú sync behavior thuộc okr-track |

### Không thay đổi

- `skills/okr-capture/`: Không ảnh hưởng
- `skills/okr/SKILL.md` (orchestrator): Không ảnh hưởng (routing logic không đổi)
- Cấu trúc thư mục `.okr/`: Không thêm/xóa file nào

## Ràng buộc và rủi ro

| Ràng buộc | Cách xử lý |
|-----------|-----------|
| Bảng Roadmap duplicate data từ action files | SOT vẫn là action files. Bảng auto-generated, không sửa trực tiếp |
| Bảng 6 cột resources.md rộng | User đã chấp nhận. Nội dung cột giữ ngắn gọn |
| Sync phụ thuộc skill/MCP bên thứ 3 | Sync là optional. Không có integration → skip. Không block flow |
| Migration schema cũ resources.md | Auto-detect + auto-migrate. Không mất data |
| Render bảng mỗi lần update tốn logic | Đọc tất cả action files (thường <50 files), nhanh. Không bottleneck |
