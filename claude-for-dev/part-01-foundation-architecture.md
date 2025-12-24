# CLAUDE CODE CHO DEVELOPERS - MANGOADS
## Phần 1: Nền Tảng & Kiến Trúc

**Phiên bản:** 1.0
**Đối tượng:** Frontend, Backend, QA, DevOps Team (bao gồm Junior & Intern)
**Thời lượng:** 45-60 phút

---

## BẢNG THUẬT NGỮ QUAN TRỌNG

> 📚 **Dành cho Junior/Intern**: Hãy đọc kỹ bảng này trước khi học các phần tiếp theo.

| Thuật ngữ | Tiếng Việt | Giải thích đơn giản |
|-----------|------------|---------------------|
| **Claude Code** | - | Công cụ lập trình AI của Anthropic, chạy trên terminal |
| **CLAUDE.md** | File bộ nhớ dự án | File markdown chứa thông tin dự án để Claude "nhớ" context |
| **Subagent** | Tác tử con | AI chuyên gia được gọi để làm task cụ thể (như code reviewer, tester) |
| **Skill** | Kỹ năng | Kiến thức chuyên môn Claude có thể tự động sử dụng khi cần |
| **Hook** | Móc sự kiện | Code tự động chạy khi có sự kiện xảy ra (như trước/sau khi sửa file) |
| **Command** | Lệnh tắt | Shortcut như `/review`, `/test` để trigger workflow |
| **MCP** | Giao thức kết nối | Cách Claude Code kết nối với tools bên ngoài (browser, database...) |
| **Tool** | Công cụ | Các khả năng Claude có thể dùng (đọc file, chạy lệnh, tìm kiếm...) |
| **Session** | Phiên làm việc | Một cuộc hội thoại với Claude Code từ lúc bắt đầu đến kết thúc |
| **Checkpoint** | Điểm lưu | Bản snapshot để khôi phục nếu có lỗi xảy ra |
| **Permission** | Quyền hạn | Mức độ Claude được phép làm gì (chỉ đọc, sửa file, chạy lệnh...) |
| **Context** | Ngữ cảnh | Thông tin xung quanh mà Claude cần biết để hiểu task |

### Giải thích chi tiết các thuật ngữ quan trọng

#### 🤖 Subagent (Tác tử con) là gì?

**Subagent** giống như việc bạn có nhiều đồng nghiệp chuyên gia, mỗi người giỏi một việc:
- `code-reviewer`: Chuyên gia review code, tìm bugs
- `test-generator`: Chuyên gia viết unit tests
- `api-builder`: Chuyên gia xây dựng API

Khi bạn yêu cầu một task phức tạp, Claude sẽ gọi đúng "chuyên gia" để xử lý.

```
Bạn: "Review code và viết tests"
     ↓
Claude gọi 2 subagents:
├── code-reviewer → Phân tích code, tìm vấn đề
└── test-generator → Viết unit tests
```

#### 🎯 Skill (Kỹ năng) là gì?

**Skill** là kiến thức chuyên môn được "cài sẵn" cho Claude. Khác với Subagent:
- **Subagent**: AI riêng biệt, chạy độc lập
- **Skill**: Kiến thức Claude dùng trong conversation hiện tại

Ví dụ: Skill "security-patterns" giúp Claude tự động nhận ra lỗi bảo mật khi review code.

#### 🔗 Hook (Móc sự kiện) là gì?

**Hook** là code tự động chạy khi có sự kiện. Giống như event listeners trong JavaScript:

```
┌─────────────────────────────────────────────────┐
│  Các loại Hook                                  │
├─────────────────────────────────────────────────┤
│  PreToolUse    → Chạy TRƯỚC khi Claude làm gì  │
│  PostToolUse   → Chạy SAU khi Claude làm xong  │
│  Stop          → Chạy khi Claude dừng lại      │
│  SessionStart  → Chạy khi bắt đầu session mới  │
└─────────────────────────────────────────────────┘
```

Ví dụ thực tế:
- Hook chặn không cho commit nếu chưa chạy tests
- Hook tự động chạy linter sau mỗi lần sửa file

#### 🔌 MCP (Model Context Protocol) là gì?

**MCP** là cách Claude Code kết nối với các công cụ bên ngoài:

```
Claude Code ←→ MCP ←→ Công cụ bên ngoài
                     ├── Browser (Playwright)
                     ├── Database (PostgreSQL)
                     ├── GitHub
                     └── Bất kỳ service nào
```

Không có MCP, Claude chỉ có thể đọc/ghi file và chạy terminal commands.
Có MCP, Claude có thể điều khiển browser, query database, tạo GitHub PRs...

---

## MỤC LỤC

1. [Kiến trúc tổng quan Claude Code](#1-kiến-trúc-tổng-quan-claude-code)
2. [Hệ thống bộ nhớ dự án](#2-hệ-thống-bộ-nhớ-dự-án)
3. [CLI và quản lý phiên làm việc](#3-cli-và-quản-lý-phiên-làm-việc)
4. [Hệ thống công cụ và quyền hạn](#4-hệ-thống-công-cụ-và-quyền-hạn)
5. [Điểm lưu và khôi phục](#5-điểm-lưu-và-khôi-phục)

---

## 1. KIẾN TRÚC TỔNG QUAN CLAUDE CODE

### 1.1 Kiến trúc cấp cao

> 💡 **Giải thích cho Junior**: Hãy hình dung Claude Code như một "trợ lý lập trình" có nhiều tầng, từ giao diện người dùng đến các công cụ thực thi.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLAUDE CODE ARCHITECTURE                           │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         USER INTERFACE                                   ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                      ││
│  │  │   CLI       │  │  Keyboard   │  │  Slash      │                      ││
│  │  │   Input     │  │  Shortcuts  │  │  Commands   │                      ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘                      ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         CORE ENGINE                                      ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    ││
│  │  │   Memory    │  │   Session   │  │   Tool      │  │   Hook      │    ││
│  │  │   System    │  │   Manager   │  │   Executor  │  │   System    │    ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         EXECUTION LAYER                                  ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    ││
│  │  │   Read      │  │   Write     │  │   Bash      │  │   MCP       │    ││
│  │  │   Tool      │  │   Tool      │  │   Tool      │  │   Client    │    ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         EXTERNAL SYSTEMS                                 ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    ││
│  │  │ File System │  │   Git       │  │  Playwright │  │  Other MCP  │    ││
│  │  │             │  │             │  │  MCP Server │  │  Servers    │    ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Các thành phần chính

| Thành phần | Chức năng | Vị trí |
|-----------|---------|----------|
| **Hệ thống bộ nhớ** | Lưu context và quy tắc dự án | CLAUDE.md, .claude/rules/ |
| **Tác tử con (Subagents)** | AI chuyên gia cho từng task | .claude/agents/ |
| **Kỹ năng (Skills)** | Kiến thức tái sử dụng | .claude/skills/ |
| **Móc sự kiện (Hooks)** | Kiểm soát vòng đời | .claude/settings.json |
| **Lệnh tắt (Commands)** | Shortcut cho người dùng | .claude/commands/ |
| **MCP** | Tích hợp công cụ bên ngoài | .mcp.json |
| **Plugins** | Gói chia sẻ được | .claude-plugin/ |

### 1.3 Luồng dữ liệu

> 💡 **Giải thích cho Junior**: Đây là cách Claude xử lý yêu cầu của bạn từ đầu đến cuối.

```
User Input
    │
    ▼
┌───────────────────┐
│ Memory Loading    │ ← CLAUDE.md, rules/
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Hook: UserPrompt  │ ← Pre-process input
│ Submit            │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Claude Processing │ ← Main AI reasoning
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Tool Selection    │ ← Choose appropriate tool
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Hook: PreToolUse  │ ← Validate/block/modify
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Tool Execution    │ ← Read/Write/Bash/MCP
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Hook: PostToolUse │ ← Log/notify
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Hook: Stop        │ ← Quality gates
└────────┬──────────┘
         │
         ▼
    Response
```

---

## 2. HỆ THỐNG BỘ NHỚ DỰ ÁN

### 2.1 Thứ tự ưu tiên bộ nhớ

> 💡 **Giải thích cho Junior**: Claude đọc nhiều file cấu hình theo thứ tự ưu tiên. File ở dưới sẽ "ghi đè" file ở trên nếu có conflict.

```
┌─────────────────────────────────────────────────────────────┐
│                    MEMORY HIERARCHY                          │
│                                                              │
│  Priority: LOW ──────────────────────────────────► HIGH     │
│                                                              │
│  ~/.claude/CLAUDE.md     (Global - all projects)            │
│       │                                                      │
│       ▼                                                      │
│  /project/CLAUDE.md      (Project root)                     │
│       │                                                      │
│       ▼                                                      │
│  /project/.claude/CLAUDE.md  (Alternative location)         │
│       │                                                      │
│       ▼                                                      │
│  /project/.claude/rules/*.md (Scoped rules)                 │
│       │                                                      │
│       ▼                                                      │
│  /project/CLAUDE.local.md (Personal, highest priority)      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Cấu trúc CLAUDE.md cho Development

> 💡 **Giải thích cho Junior**: CLAUDE.md giống như file README nhưng dành riêng cho Claude. Mọi thông tin bạn muốn Claude "nhớ" về dự án đều đặt ở đây.

```markdown
# CLAUDE.md - [Project Name]

## Project Overview
Brief description of what this project does.

## Tech Stack
- Runtime: Node.js 20
- Framework: Next.js 14 (App Router)
- Language: TypeScript (strict mode)
- Styling: Tailwind CSS + shadcn/ui
- Database: PostgreSQL + Prisma
- Testing: Vitest + Playwright

## Directory Structure
```
src/
├── app/                 # Next.js App Router
│   ├── (auth)/         # Auth routes group
│   ├── (dashboard)/    # Dashboard routes
│   └── api/            # API routes
├── components/
│   ├── ui/             # shadcn/ui components
│   └── features/       # Feature components
├── lib/
│   ├── db/             # Database utilities
│   └── utils/          # Helper functions
├── hooks/              # Custom React hooks
└── types/              # TypeScript types
```

## Development Commands
```bash
pnpm dev          # Start dev server (port 3000)
pnpm build        # Production build
pnpm test         # Run unit tests
pnpm test:e2e     # Run Playwright tests
pnpm lint         # ESLint check
pnpm typecheck    # TypeScript check
pnpm db:migrate   # Run migrations
pnpm db:seed      # Seed database
```

## Code Conventions

### Naming
- Components: PascalCase (UserCard.tsx)
- Files: kebab-case (user-card.tsx)
- Functions: camelCase (getUserById)
- Constants: SCREAMING_SNAKE_CASE (MAX_RETRIES)
- Types/Interfaces: PascalCase with prefix (IUser, TResponse)

### Component Pattern
```tsx
// Named exports only
export function ComponentName({ prop1, prop2 }: ComponentProps) {
  // Hooks at top
  // Handlers next
  // Return JSX
}
```

### API Pattern
```typescript
// Always return consistent format
return NextResponse.json({
  success: true,
  data: result,
  error: null
})
```

## Git Workflow
- Branch from: develop
- Branch naming: feature/[ticket]-description
- Commit style: Conventional Commits
- PR requires: 1 approval + CI pass

## Environment Variables
See .env.example for required variables.
NEVER commit .env files.

## Known Issues / Gotchas
- [List any quirks developers should know]

## Team
- Tech Lead: @name
- Backend: @name
- Frontend: @name
```

### 2.3 Quy tắc có phạm vi (Scoped Rules) với YAML Frontmatter

> 💡 **Giải thích cho Junior**: Bạn có thể tạo quy tắc chỉ áp dụng cho một số file nhất định. Ví dụ: quy tắc cho React components chỉ áp dụng cho file `.tsx`.

**File: `.claude/rules/react-components.md`**

```markdown
---
paths: ["src/components/**/*.tsx", "src/app/**/*.tsx"]
---

# React Component Rules

## Required Patterns

### Props Interface
Always define props interface above component:
```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  onClick?: () => void
}
```

### Forward Ref for Interactive Components
```tsx
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', ...props }, ref) => {
    return <button ref={ref} {...props} />
  }
)
Button.displayName = 'Button'
```

### Error Boundaries
Wrap feature components with error boundaries.

## Prohibited Patterns
- ❌ Default exports (use named exports)
- ❌ Inline styles (use Tailwind)
- ❌ Direct DOM manipulation
- ❌ Class components (use functional)
```

**File: `.claude/rules/api-routes.md`**

```markdown
---
paths: ["src/app/api/**/*.ts"]
---

# API Route Rules

## Required Patterns

### Input Validation
Always validate input with Zod:
```typescript
import { z } from 'zod'

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100)
})

export async function POST(request: Request) {
  const body = await request.json()
  const result = CreateUserSchema.safeParse(body)

  if (!result.success) {
    return NextResponse.json({
      success: false,
      error: result.error.flatten()
    }, { status: 400 })
  }
}
```

### Error Handling
```typescript
try {
  // operation
} catch (error) {
  console.error('[API] Error:', error)
  return NextResponse.json({
    success: false,
    error: 'Internal server error'
  }, { status: 500 })
}
```

### Response Format
```typescript
// Success
{ success: true, data: T }

// Error
{ success: false, error: string | object }
```
```

---

## 3. CLI VÀ QUẢN LÝ PHIÊN LÀM VIỆC

### 3.1 Tham khảo các lệnh CLI

> 💡 **Giải thích cho Junior**: CLI (Command Line Interface) là giao diện dòng lệnh. Bạn gõ lệnh vào terminal để điều khiển Claude Code.

```bash
# Start interactive session
claude

# Non-interactive (print mode)
claude -p "Generate a React component for user profile"

# Continue last session
claude -c
claude --continue "add tests for the component"

# Resume specific session
claude -r <session_id>
claude --resume <session_id> "fix the failing test"

# With permission mode
claude --mode acceptEdits
claude --mode plan

# Output format
claude -p --output-format json "describe this file"
```

### 3.2 Các trạng thái phiên làm việc (Session States)

> 💡 **Giải thích cho Junior**: Session là một cuộc "trò chuyện" với Claude. Bạn có thể tạm dừng và tiếp tục session sau.

```
┌─────────────────────────────────────────────────────────────┐
│                    SESSION LIFECYCLE                         │
│                                                              │
│  ┌───────────┐      ┌───────────┐      ┌───────────┐       │
│  │  Created  │ ───► │  Active   │ ───► │  Paused   │       │
│  └───────────┘      └─────┬─────┘      └─────┬─────┘       │
│                           │                   │              │
│                           ▼                   │              │
│                     ┌───────────┐             │              │
│                     │ Checkpoints│◄───────────┘              │
│                     └───────────┘                            │
│                           │                                  │
│                           ▼                                  │
│                     ┌───────────┐                            │
│                     │  Resumed  │                            │
│                     └───────────┘                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 Phím tắt (Keyboard Shortcuts)

| Phím tắt | Chức năng |
|----------|--------|
| `Shift+Tab` | Chuyển đổi chế độ quyền hạn |
| `Esc Esc` | Quay lại (rewind/rollback) |
| `Ctrl+C` | Hủy thao tác hiện tại |
| `/` | Hiện danh sách lệnh |
| `@` | Tham chiếu tài nguyên |
| `#` | Thêm vào CLAUDE.md |

### 3.4 Các lệnh trong phiên làm việc

```bash
/init           # Generate CLAUDE.md
/clear          # Clear conversation
/compact        # Reduce context size
/rewind         # Open rewind menu
/resume         # Switch sessions
/agents         # Manage subagents
/mcp            # Manage MCP servers
/hooks          # Review hooks
/help           # Show help
```

---

## 4. HỆ THỐNG CÔNG CỤ VÀ QUYỀN HẠN

### 4.1 Các công cụ tích hợp sẵn

> 💡 **Giải thích cho Junior**: Claude Code có sẵn các công cụ (tools) để làm việc. Mỗi tool có mức độ rủi ro khác nhau.

| Công cụ | Chức năng | Mức rủi ro |
|------|---------|------------|
| `Read` | Đọc nội dung file | Thấp |
| `Write` | Tạo file mới | Trung bình |
| `Edit` | Sửa file có sẵn | Trung bình |
| `MultiEdit` | Sửa nhiều file cùng lúc | Trung bình |
| `Bash` | Chạy lệnh shell | Cao |
| `Glob` | Tìm file theo pattern | Thấp |
| `Grep` | Tìm nội dung trong file | Thấp |
| `WebFetch` | Gọi HTTP requests | Trung bình |
| `WebSearch` | Tìm kiếm web | Thấp |

### 4.2 Các chế độ quyền hạn (Permission Modes)

> 💡 **Giải thích cho Junior**: Permission mode quyết định Claude được phép làm gì mà không cần hỏi bạn. Chọn mode phù hợp với độ tin cậy bạn muốn cho Claude.

```
┌─────────────────────────────────────────────────────────────┐
│                   PERMISSION MODES                           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ default                                              │    │
│  │ - Asks permission for every action                  │    │
│  │ - Safest mode                                        │    │
│  │ - Best for: learning, sensitive projects            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ acceptEdits                                          │    │
│  │ - Auto-accepts file read/write/edit                 │    │
│  │ - Still asks for bash, dangerous ops                │    │
│  │ - Best for: active development                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ plan                                                 │    │
│  │ - Only plans, no execution                          │    │
│  │ - Read-only operations allowed                      │    │
│  │ - Best for: architecture review, planning           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ bypassPermissions ⚠️                                 │    │
│  │ - Skips ALL permission prompts                      │    │
│  │ - Requires: --dangerously-skip-permissions flag     │    │
│  │ - Best for: CI/CD, sandboxed environments ONLY      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Giới hạn công cụ cho Subagent (Tool Scoping)

> 💡 **Giải thích cho Junior**: Khi tạo subagent, bạn nên giới hạn tools mà nó có thể dùng. Đây là nguyên tắc "quyền tối thiểu" (least privilege) - chỉ cấp đủ quyền cần thiết.

```yaml
# Minimal access (read-only)
tools: Read, Glob, Grep

# Development access
tools: Read, Write, Edit, Glob, Grep

# Full access
tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch

# With MCP tools
tools: Read, Write, Edit, Glob, Grep, mcp__playwright, mcp__github
```

---

## 5. ĐIỂM LƯU VÀ KHÔI PHỤC (CHECKPOINTING & ROLLBACK)

### 5.1 Checkpointing hoạt động như thế nào?

> 💡 **Giải thích cho Junior**: Checkpoint giống như "save game" trong game. Nếu có lỗi, bạn có thể quay lại điểm save trước đó.

```
┌─────────────────────────────────────────────────────────────┐
│                    CHECKPOINT SYSTEM                         │
│                                                              │
│  Initial State                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Checkpoint 0 (auto-saved)                            │    │
│  │ - All files in current state                        │    │
│  │ - Conversation empty                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  Claude edits: src/components/Button.tsx                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Checkpoint 1 (auto-saved)                            │    │
│  │ - Button.tsx state saved                            │    │
│  │ - Conversation turn 1 saved                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  Claude edits: src/lib/utils.ts                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Checkpoint 2 (auto-saved)                            │    │
│  │ - utils.ts state saved                              │    │
│  │ - Conversation turn 2 saved                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  User: /rewind                                              │
│  Options:                                                    │
│  - Restore code only                                        │
│  - Restore conversation only                                │
│  - Restore both                                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Các tùy chọn khôi phục (Rewind Options)

| Tùy chọn | Thay đổi Code | Hội thoại |
|----------|---------------|-----------|
| Chỉ code | ↩️ Khôi phục | Giữ nguyên |
| Chỉ hội thoại | Giữ nguyên | ↩️ Khôi phục |
| Cả hai | ↩️ Khôi phục | ↩️ Khôi phục |

### 5.3 Giới hạn của Checkpoint

```
⚠️ GIỚI HẠN CỦA CHECKPOINT

1. Chỉ theo dõi file Claude TRỰC TIẾP sửa
   - File sửa qua Bash KHÔNG được lưu
   - Ví dụ: `mv`, `rm`, `cp` không được checkpoint

2. Chỉ trong phiên làm việc
   - Không thay thế được Git
   - Checkpoint mất khi kết thúc session

3. Không theo dõi thay đổi bên ngoài
   - Git pull, sửa bằng IDE không được lưu

THỰC HÀNH TỐT:
   Checkpoint = hoàn tác nhanh tạm thời
   Git = quản lý phiên bản vĩnh viễn
```

### 5.4 Quy trình kết hợp Checkpoint + Git

```bash
# Quy trình khuyến nghị

1. Bắt đầu task
   → Checkpoint tự động được tạo

2. Claude thực hiện thay đổi
   → Checkpoint tại mỗi lần sửa

3. Nếu có lỗi:
   → /rewind để khôi phục

4. Khi hài lòng:
   → git add . && git commit

5. Nếu cần khôi phục sau khi commit:
   → git checkout / git revert
```

---

## CHECKLIST HOÀN THÀNH PHẦN 1

### Kiến thức
- [ ] Hiểu kiến trúc tổng quan Claude Code
- [ ] Hiểu thứ tự ưu tiên bộ nhớ (memory hierarchy)
- [ ] Biết cách dùng CLI và các lệnh cơ bản
- [ ] Hiểu các chế độ quyền hạn (permission modes)
- [ ] Hiểu hệ thống checkpoint

### Thực hành
- [ ] Tạo file CLAUDE.md cho project của bạn
- [ ] Tạo ít nhất 1 scoped rule trong `.claude/rules/`
- [ ] Thử chuyển đổi giữa các permission modes
- [ ] Thử dùng `/rewind` một lần để khôi phục

---

## TÓM TẮT PHẦN 1

### Đã học:
- [x] Kiến trúc tổng quan Claude Code
- [x] Hệ thống bộ nhớ (CLAUDE.md, rules/)
- [x] Các lệnh CLI và quản lý phiên làm việc
- [x] Hệ thống công cụ và các chế độ quyền hạn
- [x] Điểm lưu và khôi phục (Checkpointing & Rollback)

### Phần 2 sẽ học:
- [ ] Tác tử con (Subagents) - Thiết kế và triển khai
- [ ] Kỹ năng (Skills) - Tạo kiến thức tái sử dụng
- [ ] Móc sự kiện (Hooks) - Kiểm soát vòng đời
- [ ] Ví dụ thực tế: code-reviewer subagent

---

**Tiếp theo:** [Phần 2: Tác tử con & Móc sự kiện](./part-02-subagents-hooks.md)
