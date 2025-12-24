# CLAUDE CODE CHO DEVELOPERS - MANGOADS
## Part 1: Foundation & Architecture

**Phiên bản:** 1.0
**Đối tượng:** Frontend, Backend, QA, DevOps Team
**Thời lượng:** 45-60 phút

---

## MỤC LỤC

1. [Claude Code Architecture Overview](#1-claude-code-architecture-overview)
2. [Project Memory System](#2-project-memory-system)
3. [CLI & Session Management](#3-cli--session-management)
4. [Tool System & Permissions](#4-tool-system--permissions)
5. [Checkpointing & Rollback](#5-checkpointing--rollback)

---

## 1. CLAUDE CODE ARCHITECTURE OVERVIEW

### 1.1 High-Level Architecture

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

### 1.2 Core Components

| Component | Purpose | Location |
|-----------|---------|----------|
| **Memory System** | Project context & rules | CLAUDE.md, .claude/rules/ |
| **Subagents** | Specialized AI workers | .claude/agents/ |
| **Skills** | Reusable expertise | .claude/skills/ |
| **Hooks** | Lifecycle control | .claude/settings.json |
| **Commands** | User shortcuts | .claude/commands/ |
| **MCP** | External tool integration | .mcp.json |
| **Plugins** | Shareable bundles | .claude-plugin/ |

### 1.3 Data Flow

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

## 2. PROJECT MEMORY SYSTEM

### 2.1 Memory Hierarchy

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

### 2.2 CLAUDE.md Structure cho Development

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

### 2.3 Scoped Rules với YAML Frontmatter

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

## 3. CLI & SESSION MANAGEMENT

### 3.1 CLI Commands Reference

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

### 3.2 Session States

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

### 3.3 Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Shift+Tab` | Cycle permission modes |
| `Esc Esc` | Rewind/rollback |
| `Ctrl+C` | Cancel current operation |
| `/` | Show commands |
| `@` | Reference resources |
| `#` | Add to CLAUDE.md |

### 3.4 In-Session Commands

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

## 4. TOOL SYSTEM & PERMISSIONS

### 4.1 Built-in Tools

| Tool | Purpose | Risk Level |
|------|---------|------------|
| `Read` | Read file contents | Low |
| `Write` | Create new files | Medium |
| `Edit` | Modify existing files | Medium |
| `MultiEdit` | Batch file edits | Medium |
| `Bash` | Execute shell commands | High |
| `Glob` | Find files by pattern | Low |
| `Grep` | Search file contents | Low |
| `WebFetch` | HTTP requests | Medium |
| `WebSearch` | Web search | Low |

### 4.2 Permission Modes

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

### 4.3 Tool Scoping in Subagents

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

## 5. CHECKPOINTING & ROLLBACK

### 5.1 How Checkpointing Works

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

### 5.2 Rewind Options

| Option | Code Changes | Conversation |
|--------|--------------|--------------|
| Code only | ↩️ Restored | Kept |
| Conversation only | Kept | ↩️ Restored |
| Both | ↩️ Restored | ↩️ Restored |

### 5.3 Limitations

```
⚠️ CHECKPOINT LIMITATIONS

1. Only tracks DIRECT file edits by Claude
   - Files modified via Bash are NOT tracked
   - Example: `mv`, `rm`, `cp` not checkpointed

2. Session-level only
   - Not a replacement for Git
   - Checkpoints lost when session ends

3. No external changes
   - Git pulls, IDE edits not tracked

BEST PRACTICE:
   Checkpoint = quick local undo
   Git = permanent version control
```

### 5.4 Checkpoint + Git Workflow

```bash
# Recommended workflow

1. Start task
   → Checkpoint auto-created

2. Claude makes changes
   → Checkpoints at each edit

3. If mistake:
   → /rewind to restore

4. When satisfied:
   → git add . && git commit

5. If need to restore after commit:
   → git checkout / git revert
```

---

## CHECKLIST HOÀN THÀNH PART 1

### Kiến thức
- [ ] Hiểu architecture tổng quan
- [ ] Hiểu memory hierarchy
- [ ] Biết cách dùng CLI
- [ ] Hiểu permission modes
- [ ] Hiểu checkpointing system

### Thực hành
- [ ] Tạo CLAUDE.md cho project
- [ ] Tạo ít nhất 1 scoped rule
- [ ] Thử các permission modes
- [ ] Thử rewind một lần

---

## TÓM TẮT PART 1

### Đã học:
- [x] Claude Code architecture overview
- [x] Memory system (CLAUDE.md, rules/)
- [x] CLI commands và session management
- [x] Tool system và permission modes
- [x] Checkpointing và rollback

### Part 2 sẽ học:
- [ ] Subagents - Thiết kế và implementation
- [ ] Agent Skills - Tạo reusable expertise
- [ ] Hooks - Lifecycle control
- [ ] Ví dụ code review subagent

---

**Tiếp theo:** [Part 2: Subagents & Hooks](./part-02-subagents-hooks.md)
