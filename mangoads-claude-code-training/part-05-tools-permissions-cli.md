# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 5: Tools, Permission Modes & CLI/Sessions

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-4

---

## MỤC LỤC PART 5

1. [Tools trong Claude Code](#1-tools-trong-claude-code)
2. [Permission Modes](#2-permission-modes)
3. [Tool Scoping cho Subagents](#3-tool-scoping-cho-subagents)
4. [CLI Reference](#4-cli-reference)
5. [Sessions & Resume](#5-sessions--resume)
6. [Checkpointing & Rollback](#6-checkpointing--rollback)

---

## 1. TOOLS TRONG CLAUDE CODE

### 1.1 Tổng quan Tools

**Tools** là các capabilities mà Claude Code có thể sử dụng để thực hiện actions. Có 2 loại:

```
┌─────────────────────────────────────────────────────────────┐
│                    CLAUDE CODE TOOLS                         │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              BUILT-IN TOOLS                          │    │
│  │  Read, Write, Edit, Bash, Glob, Grep,               │    │
│  │  WebFetch, WebSearch                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              MCP TOOLS                               │    │
│  │  playwright_navigate, github_create_issue,          │    │
│  │  slack_send_message, ...                            │    │
│  │  (từ MCP servers được cấu hình)                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Built-in Tools Chi Tiết

| Tool | Chức năng | Risk Level | Ví dụ MangoAds |
|------|-----------|------------|----------------|
| **Read** | Đọc nội dung file | Low | Đọc config, source code |
| **Write** | Tạo file mới | Medium | Tạo component, config |
| **Edit** | Sửa file có sẵn | Medium | Fix bug, update code |
| **Bash** | Chạy shell commands | High | Build, test, deploy |
| **Glob** | Tìm files theo pattern | Low | Tìm *.tsx, package.json |
| **Grep** | Tìm content trong files | Low | Tìm function, import |
| **WebFetch** | HTTP requests | Medium | Call APIs, fetch data |
| **WebSearch** | Tìm kiếm web | Low | Research, documentation |

### 1.3 Tool Risk Classification

```
┌─────────────────────────────────────────────────────────────┐
│                    TOOL RISK LEVELS                          │
│                                                              │
│  HIGH RISK                                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Bash         - Can execute any command                 │ │
│  │ Write/Edit   - Can modify/create files                 │ │
│  │ MCP Tools    - External system access                  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  MEDIUM RISK                                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ WebFetch     - Network access, potential data exposure │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  LOW RISK                                                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Read         - Read-only file access                   │ │
│  │ Glob         - File listing only                       │ │
│  │ Grep         - Content search only                     │ │
│  │ WebSearch    - Public web search                       │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. PERMISSION MODES

### 2.1 Các Permission Modes

Claude Code có 4 permission modes:

```
┌─────────────────────────────────────────────────────────────┐
│                  PERMISSION MODES                            │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  DEFAULT                                                 ││
│  │  - Hỏi permission trước mọi action                      ││
│  │  - An toàn nhất                                         ││
│  │  - Recommended cho: Learning, sensitive work            ││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  ACCEPTEDITS                                             ││
│  │  - Auto-accept file edits                               ││
│  │  - Vẫn hỏi cho dangerous actions (rm, deploy)           ││
│  │  - Recommended cho: Focused development                 ││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  PLAN                                                    ││
│  │  - Chỉ analyze và plan                                  ││
│  │  - KHÔNG execute bất kỳ action nào                      ││
│  │  - Recommended cho: Review strategy                     ││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  BYPASSPERMISSIONS ⚠️                                    ││
│  │  - Skip ALL permission prompts                          ││
│  │  - DANGEROUS - chỉ dùng trong sandbox                   ││
│  │  - Requires --dangerously-skip-permissions flag         ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 So sánh chi tiết

| Action | default | acceptEdits | plan | bypassPermissions |
|--------|---------|-------------|------|-------------------|
| Read file | ✅ Auto | ✅ Auto | ✅ Auto | ✅ Auto |
| Write file | ❓ Ask | ✅ Auto | ❌ Block | ✅ Auto |
| Edit file | ❓ Ask | ✅ Auto | ❌ Block | ✅ Auto |
| Run bash | ❓ Ask | ❓ Ask | ❌ Block | ✅ Auto |
| Delete file | ❓ Ask | ❓ Ask | ❌ Block | ✅ Auto |
| Deploy | ❓ Ask | ❓ Ask | ❌ Block | ✅ Auto |
| MCP tool | ❓ Ask | ❓ Ask | ❌ Block | ✅ Auto |

### 2.3 Chuyển đổi Permission Mode

**Trong session (Interactive):**
```
Shift + Tab → Cycle: default → acceptEdits → plan → default
```

**Qua CLI:**
```bash
# Start với acceptEdits mode
claude --mode acceptEdits

# Start với plan mode
claude --mode plan
```

**Trong settings:**
```json
{
  "defaultMode": "acceptEdits"
}
```

### 2.4 Trade-off: Tốc độ vs An toàn

```
AN TOÀN ◄───────────────────────────────────────────► TỐC ĐỘ

  default    acceptEdits    plan    bypassPermissions
     │            │           │              │
     ▼            ▼           ▼              ▼
  Hỏi mọi     Hỏi risky   Chỉ plan     Không hỏi
  action      actions       không        gì hết
                            execute

RECOMMENDED FOR:
- default: Learning, sensitive projects, production
- acceptEdits: Daily development, trusted codebase
- plan: Strategy review, architecture planning
- bypassPermissions: CI/CD, sandboxed environments ONLY
```

### 2.5 Gợi ý cho MangoAds

| Scenario | Mode | Lý do |
|----------|------|-------|
| Onboarding nhân viên mới | default | Học cách Claude hoạt động |
| Development hàng ngày | acceptEdits | Balance tốc độ và an toàn |
| Review architecture | plan | Chỉ xem plan, không execute |
| Automated testing (CI) | bypassPermissions | Cần tự động hoàn toàn |
| Production hotfix | default | An toàn tối đa |

---

## 3. TOOL SCOPING CHO SUBAGENTS

### 3.1 Tại sao cần Tool Scoping?

**Principle of Least Privilege:** Chỉ cấp tools cần thiết cho task.

```
❌ KHÔNG TỐT:
Subagent "code-reviewer" có quyền Write, Edit, Bash
→ Reviewer không cần modify code!

✅ TỐT:
Subagent "code-reviewer" chỉ có Read, Glob, Grep
→ Chỉ có thể đọc và tìm kiếm
```

### 3.2 Cách configure Tool Scoping

**Trong subagent definition:**

```yaml
---
name: security-auditor
description: Audits code for security vulnerabilities
tools: Read, Glob, Grep
---
```

**Nếu không specify `tools`:** Subagent inherit TẤT CẢ tools từ main agent (bao gồm MCP).

### 3.3 Tool Scoping Patterns

#### Pattern 1: Read-only Auditor

```yaml
---
name: security-auditor
tools: Read, Glob, Grep
permissionMode: plan
---

# Chỉ đọc code, không thể modify
```

#### Pattern 2: Content Writer

```yaml
---
name: content-writer
tools: Read, Write, Edit, Glob, Grep
permissionMode: acceptEdits
---

# Có thể tạo/sửa files, nhưng không có Bash
```

#### Pattern 3: Full Developer

```yaml
---
name: feature-developer
tools: Read, Write, Edit, Bash, Glob, Grep
permissionMode: default
---

# Full access nhưng vẫn hỏi permissions
```

#### Pattern 4: QA Tester với MCP

```yaml
---
name: qa-tester
tools: Read, Glob, Grep, mcp__playwright
permissionMode: default
---

# Read-only code access + Playwright cho testing
```

### 3.4 MCP Tools trong Subagents

**Include specific MCP tools:**

```yaml
tools: Read, Glob, Grep, mcp__playwright, mcp__github
```

**Exclude MCP tools (chỉ built-in):**

```yaml
tools: Read, Write, Edit, Bash, Glob, Grep
# Không list MCP tools → không access
```

### 3.5 Checklist Tool Scoping MangoAds

```markdown
## Tool Scoping Checklist

### Security Auditor
- [x] Read - đọc code
- [x] Glob - tìm files
- [x] Grep - tìm patterns
- [ ] Write - KHÔNG CẦN
- [ ] Edit - KHÔNG CẦN
- [ ] Bash - KHÔNG CẦN
- [ ] MCP - KHÔNG CẦN

### Content Writer
- [x] Read - đọc context
- [x] Write - tạo content files
- [x] Edit - sửa content
- [x] Glob - tìm templates
- [x] Grep - tìm existing content
- [ ] Bash - KHÔNG CẦN
- [ ] MCP - KHÔNG CẦN

### QA Tester
- [x] Read - đọc test specs
- [x] Glob - tìm test files
- [x] Grep - tìm test patterns
- [ ] Write - KHÔNG modify tests
- [ ] Edit - KHÔNG modify tests
- [ ] Bash - KHÔNG chạy commands
- [x] mcp__playwright - web testing

### Full Stack Developer
- [x] All built-in tools
- [x] mcp__github - version control
- [ ] mcp__playwright - optional (cho debug)
```

---

## 4. CLI REFERENCE

### 4.1 Basic Commands

**Khởi động Claude Code:**

```bash
# Interactive mode (REPL)
claude

# Non-interactive (print mode)
claude -p "your prompt"

# Continue last session
claude -c
claude --continue "additional prompt"

# Resume specific session
claude -r session_id
claude --resume session_id "prompt"
```

### 4.2 CLI Options

| Flag | Short | Description |
|------|-------|-------------|
| `--print` | `-p` | Non-interactive mode |
| `--continue` | `-c` | Continue last session |
| `--resume` | `-r` | Resume specific session |
| `--mode` | | Set permission mode |
| `--output-format` | | json, text, stream |
| `--no-interactive` | | Disable interactive features |
| `--dangerously-skip-permissions` | | Enable bypassPermissions |

### 4.3 Interactive Mode

```
┌─────────────────────────────────────────────────────────────┐
│                  INTERACTIVE MODE (REPL)                     │
│                                                              │
│  Features:                                                   │
│  - Real-time streaming output                               │
│  - Permission prompts với user input                        │
│  - Keyboard shortcuts                                       │
│  - Session persistence                                      │
│                                                              │
│  Keyboard Shortcuts:                                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Shift+Tab    │ Cycle permission modes                  │ │
│  │ Esc+Esc      │ Rewind/rollback                        │ │
│  │ Ctrl+C       │ Cancel current operation                │ │
│  │ /            │ Show available commands                 │ │
│  │ @            │ Reference resources                     │ │
│  │ #            │ Add to CLAUDE.md                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.4 Non-Interactive Mode (Print)

**Khi nào dùng:**
- CI/CD pipelines
- Automated scripts
- Pre-commit hooks
- Batch processing

**Ví dụ:**

```bash
# Simple prompt
claude -p "Explain this function" < src/utils/helper.ts

# With JSON output
claude -p --output-format json "Generate test cases"

# In CI/CD
claude -p "Run lint and fix issues" --dangerously-skip-permissions
```

### 4.5 Output Formats

```bash
# Text (default)
claude -p "hello"
# Output: Hello! How can I help you?

# JSON
claude -p --output-format json "hello"
# Output: {"session_id": "abc123", "response": "Hello! How can I help you?"}

# Stream
claude -p --output-format stream "hello"
# Output: [streams token by token]
```

---

## 5. SESSIONS & RESUME

### 5.1 Session Concept

```
┌─────────────────────────────────────────────────────────────┐
│                    SESSION LIFECYCLE                         │
│                                                              │
│  Session Start                                               │
│       │                                                      │
│       ▼                                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Session ID: abc123                                   │    │
│  │ Branch: feature/landing-page                        │    │
│  │ Created: 2025-12-23 10:00                           │    │
│  │                                                      │    │
│  │ Turn 1: User → "Create landing page component"      │    │
│  │ Turn 2: Claude → [Creates component]                │    │
│  │ Turn 3: User → "Add form validation"                │    │
│  │ Turn 4: Claude → [Adds validation]                  │    │
│  │ ...                                                 │    │
│  │                                                      │    │
│  │ Checkpoints saved automatically                     │    │
│  └─────────────────────────────────────────────────────┘    │
│       │                                                      │
│       ▼                                                      │
│  Session End / Pause                                        │
│       │                                                      │
│       ▼                                                      │
│  Resume Later                                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Resume Options

**Option 1: Continue most recent session**

```bash
claude -c
# hoặc
claude --continue "thêm tests cho component vừa tạo"
```

**Option 2: Resume specific session by ID**

```bash
claude -r abc123
claude --resume abc123 "continue from where we left"
```

**Option 3: Interactive resume selector**

```bash
claude --resume
# Shows list of sessions with search/filter
```

**Option 4: In-session resume**

```
/resume
# Switch conversations without exiting
```

### 5.3 Session Persistence

**Session data được lưu:**
- Conversation history
- File checkpoints
- Context state
- Branch information

**Tự động tạo summaries** trong background để hiển thị trong resume list.

### 5.4 Best Practices cho Sessions

**✅ DO:**

```markdown
1. Descriptive first prompt
   "Implement user authentication for MangoAds dashboard"
   → Session summary rõ ràng

2. Resume đúng session
   claude --resume abc123
   → Không tạo duplicate sessions

3. Clean up finished sessions
   /sessions → Delete completed sessions
```

**❌ DON'T:**

```markdown
1. Repeated --continue trong scripts
   # May create new sessions unintentionally
   → Use fixed session_id + --resume

2. Abandon sessions without cleanup
   → Clutter session list, waste storage

3. Resume wrong session
   → Confused context, wrong changes
```

---

## 6. CHECKPOINTING & ROLLBACK

### 6.1 Checkpointing là gì?

**Checkpointing** (v2.0.0+) là tính năng tự động lưu trạng thái code trước mỗi thay đổi, cho phép rollback nhanh chóng.

```
┌─────────────────────────────────────────────────────────────┐
│                  CHECKPOINTING WORKFLOW                      │
│                                                              │
│  Initial State                                               │
│  [Checkpoint 0] ←── Saved automatically                     │
│       │                                                      │
│       ▼                                                      │
│  Claude edits file A                                        │
│  [Checkpoint 1] ←── Saved automatically                     │
│       │                                                      │
│       ▼                                                      │
│  Claude edits file B                                        │
│  [Checkpoint 2] ←── Saved automatically                     │
│       │                                                      │
│       ▼                                                      │
│  Oops! File B edit is wrong                                 │
│       │                                                      │
│       ▼                                                      │
│  /rewind → Restore to Checkpoint 1                          │
│  File B restored, File A changes kept                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Cách sử dụng Rewind

**Qua command:**
```
/rewind
# Opens interactive rewind menu
```

**Qua keyboard:**
```
Esc + Esc (press Escape twice quickly)
```

### 6.3 Restore Options

Khi rewind, có 3 options:

| Option | Code Changes | Conversation |
|--------|--------------|--------------|
| **Conversation only** | KEEP | REWIND |
| **Code only** | REWIND | KEEP |
| **Both** | REWIND | REWIND |

**Ví dụ sử dụng:**

```markdown
Scenario: Claude sửa code đúng nhưng conversation bị dài/confused

→ Chọn "Conversation only"
→ Code changes vẫn giữ
→ Conversation restart từ checkpoint

---

Scenario: Claude sửa code sai, nhưng conversation context quan trọng

→ Chọn "Code only"
→ Code revert về trước
→ Conversation vẫn giữ để tiếp tục explain

---

Scenario: Cả code và conversation đều cần reset

→ Chọn "Both"
→ Full restore to checkpoint
```

### 6.4 Limitations

```markdown
## Checkpoint Limitations

1. ❌ KHÔNG track files modified by Bash commands
   # Chỉ track direct file edits trong Claude Code
   # rm, mv, cp qua Bash không được checkpoint

2. ❌ KHÔNG capture external changes
   # Git pull, IDE edits không được track

3. ❌ Session-level only
   # Không phải permanent history như Git

4. ✅ Combines well with Git
   # Checkpoint = local undo
   # Git = permanent history
```

### 6.5 Checkpoint + Git Workflow

**Recommended workflow MangoAds:**

```
┌─────────────────────────────────────────────────────────────┐
│            CHECKPOINT + GIT WORKFLOW                         │
│                                                              │
│  1. Start task                                               │
│     └── Checkpoint auto-created                             │
│                                                              │
│  2. Claude makes changes                                    │
│     └── Checkpoints at each edit                            │
│                                                              │
│  3. If mistake:                                              │
│     └── /rewind → Quick local undo                          │
│                                                              │
│  4. When satisfied:                                          │
│     └── git add . && git commit                             │
│     └── Permanent save in Git                               │
│                                                              │
│  5. If need to restore after commit:                        │
│     └── git checkout / git revert                           │
│     └── Git handles it                                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.6 Ví dụ thực tế MangoAds

**Scenario: Refactoring landing page component**

```
1. User: "Refactor LandingPage component để dùng shadcn/ui"

2. Claude edits LandingPage.tsx
   [Checkpoint 1 saved]

3. Claude edits Header.tsx
   [Checkpoint 2 saved]

4. Claude edits Footer.tsx
   [Checkpoint 3 saved]

5. User: "Hmm, Footer changes break layout"

6. User: /rewind
   - Select: Code only
   - Restore to: Checkpoint 2

7. Footer.tsx restored, LandingPage.tsx và Header.tsx vẫn có changes

8. User: "Thử approach khác cho Footer"

9. Claude makes different changes to Footer.tsx
   [Checkpoint 4 saved]

10. User: "Looks good! Let's commit"

11. git add . && git commit -m "refactor: landing page with shadcn/ui"
```

---

## TÓM TẮT PART 5

### Đã cover trong Part 5:
- [x] Built-in Tools và risk classification
- [x] Permission Modes (default, acceptEdits, plan, bypassPermissions)
- [x] Trade-off tốc độ vs an toàn
- [x] Tool Scoping cho Subagents
- [x] CLI Reference (interactive vs non-interactive)
- [x] Sessions và Resume options
- [x] Checkpointing và Rollback
- [x] Checkpoint + Git workflow

### Part 6 sẽ cover:
- [ ] Decision Tree chọn Claude Code component
- [ ] Workflow chuẩn MangoAds
- [ ] Brief → Phân tích → Thiết kế → Test → Rollout

---

**Tiếp theo:** [Part 6: Decision Tree & Workflow MangoAds](./part-06-decision-tree-workflow.md)
