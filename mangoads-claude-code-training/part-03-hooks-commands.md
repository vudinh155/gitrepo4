# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 3: Hooks & Slash Commands

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1, 2

---

## MỤC LỤC PART 3

1. [Hooks - Guardrails & Policy Enforcement](#1-hooks---guardrails--policy-enforcement)
2. [Lifecycle Hook Events](#2-lifecycle-hook-events)
3. [Hook Configuration](#3-hook-configuration)
4. [Slash Commands](#4-slash-commands)
5. [MCP Slash Commands](#5-mcp-slash-commands)
6. [Best Practices & Anti-patterns](#6-best-practices--anti-patterns)

---

## 1. HOOKS - GUARDRAILS & POLICY ENFORCEMENT

### 1.1 Định nghĩa

**Hooks** là các điểm can thiệp (intervention points) trong lifecycle của Claude Code, cho phép bạn:
- Kiểm soát actions trước khi thực hiện
- Validate outputs sau khi hoàn thành
- Enforce policies tự động
- Trigger workflows bổ sung

> **Analogy MangoAds:** Hooks giống như security checkpoints tại sân bay - trước khi hành khách (action) được phép đi tiếp, phải qua checkpoint (hook) để kiểm tra.

### 1.2 Tại sao cần Hooks?

```
Không có Hooks:
┌─────────────────────────────────────────────────┐
│  Claude Code                                     │
│                                                  │
│  User: "Delete all files in /production"        │
│  Claude: "OK, executing rm -rf /production"     │
│          → THỰC HIỆN NGAY                       │
│                                                  │
└─────────────────────────────────────────────────┘

Có Hooks:
┌─────────────────────────────────────────────────┐
│  Claude Code                                     │
│                                                  │
│  User: "Delete all files in /production"        │
│  Claude: "OK, executing rm -rf /production"     │
│                    │                             │
│                    ▼                             │
│          ┌─────────────────┐                    │
│          │  PreToolUse     │                    │
│          │  Hook checks:   │                    │
│          │  - Is Bash?     │                    │
│          │  - Has rm -rf?  │                    │
│          │  - Target prod? │                    │
│          │  → BLOCKED!     │                    │
│          └─────────────────┘                    │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 1.3 Hooks như Guardrails

**Use cases quan trọng:**

| Hook Type | Guardrail Example |
|-----------|-------------------|
| PreToolUse | Block dangerous bash commands |
| PreToolUse | Prevent writes to protected files |
| Stop | Run linter before finishing |
| Stop | Require tests pass |
| SubagentStop | Validate subagent output quality |
| SessionStart | Setup environment variables |

### 1.4 File Configuration Location

```
📁 Project-level (ưu tiên)
└── .claude/settings.json

📁 User-level (global)
└── ~/.claude/settings.json
```

**Cấu trúc settings.json:**

```json
{
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...],
    "SubagentStop": [...],
    "SessionStart": [...],
    "UserPromptSubmit": [...]
  }
}
```

---

## 2. LIFECYCLE HOOK EVENTS

### 2.1 Tổng quan các Events

```
Session Lifecycle:

SessionStart ─────────────────────────────────────────────────────┐
     │                                                             │
     ▼                                                             │
UserPromptSubmit ◄────────────────────────────────────────────┐   │
     │                                                         │   │
     ▼                                                         │   │
┌─────────────────────────────────────────────────────────────┐│   │
│                     CLAUDE PROCESSING                        ││   │
│                                                              ││   │
│   PreToolUse ──► Tool Execution ──► PostToolUse             ││   │
│        │              │                   │                  ││   │
│        │              │                   ▼                  ││   │
│        │              │         PostToolUseFailure (on fail) ││   │
│        │              │                                      ││   │
│        ▼              │                                      ││   │
│   PermissionRequest   │                                      ││   │
│   (if needed)         │                                      ││   │
│                       │                                      ││   │
└───────────────────────┼──────────────────────────────────────┘│   │
                        │                                       │   │
                        ▼                                       │   │
                 SubagentStop (if subagent)                     │   │
                        │                                       │   │
                        ▼                                       │   │
                     Stop ──────────────────────────────────────┘   │
                        │                                           │
                        ▼                                           │
                   PreCompact (if context full)                     │
                        │                                           │
                        ▼                                           │
                   SessionEnd ──────────────────────────────────────┘
```

### 2.2 Chi tiết từng Event

#### 2.2.1 PreToolUse (Quan trọng nhất)

**Định nghĩa:** Trigger SAU KHI Claude chọn tool + parameters, TRƯỚC KHI thực thi.

**Timing:**
```
Claude decides: "I'll use Bash with command 'rm -rf /tmp'"
         │
         ▼
   PreToolUse hook fires ◄─── CAN BLOCK/MODIFY HERE
         │
         ▼
   Tool actually executes (if not blocked)
```

**Use cases MangoAds:**
- Block dangerous bash commands
- Prevent modification of protected files
- Require review for certain operations
- Modify parameters before execution (v2.0.10+)

**Ví dụ cấu hình:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

#### 2.2.2 PostToolUse

**Định nghĩa:** Trigger SAU KHI tool thực thi thành công.

**Use cases:**
- Log tool usage
- Notify on certain operations
- Trigger follow-up actions

#### 2.2.3 PostToolUseFailure

**Định nghĩa:** Trigger SAU KHI tool thực thi thất bại.

**Use cases:**
- Error logging
- Alert on failures
- Recovery actions

#### 2.2.4 Stop (Quan trọng)

**Định nghĩa:** Trigger khi Claude hoàn thành turn (end-of-turn).

**Timing:**
```
Claude: "Done! I've completed the task."
         │
         ▼
   Stop hook fires ◄─── RUN QUALITY GATES HERE
         │
         ▼
   Turn actually ends (if not blocked)
```

**Use cases MangoAds:**
- Run linter check
- Run tests
- Validate output format
- Auto-commit changes

**Ví dụ cấu hình:**

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint"
          },
          {
            "type": "command",
            "command": "pnpm test"
          }
        ]
      }
    ]
  }
}
```

#### 2.2.5 SubagentStop

**Định nghĩa:** Trigger khi subagent (tạo qua Task tool) hoàn thành.

**Use cases:**
- Validate subagent output
- Quality check trước khi merge vào main conversation
- Log subagent performance

**Ví dụ với prompt-based hook:**

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Has the subagent completed the assigned task correctly? Check for completeness and accuracy."
          }
        ]
      }
    ]
  }
}
```

#### 2.2.6 SessionStart

**Định nghĩa:** Trigger khi session bắt đầu.

**Use cases MangoAds:**
- Setup environment variables
- Install dependencies
- Verify project state
- Load cached context

**Ví dụ cấu hình:**

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/setup-env.sh"
          }
        ]
      }
    ]
  }
}
```

#### 2.2.7 UserPromptSubmit

**Định nghĩa:** Trigger khi user submit prompt, TRƯỚC KHI Claude xử lý.

**Use cases:**
- Pre-process user input
- Add context automatically
- Filter/validate requests

#### 2.2.8 PermissionRequest

**Định nghĩa:** Trigger khi Claude cần permission từ user.

**Use cases:**
- Auto-approve certain operations
- Log permission requests
- Custom permission UI

#### 2.2.9 PreCompact

**Định nghĩa:** Trigger trước khi context compaction (khi context gần full).

**Use cases:**
- Save important context
- Log session state
- Clean up before compaction

---

## 3. HOOK CONFIGURATION

### 3.1 Matcher Syntax

Matcher xác định hook áp dụng cho tool nào:

```json
{
  "matcher": "Write"           // Chỉ Write tool
}

{
  "matcher": "Edit|MultiEdit|Write"  // OR logic: Edit HOẶC MultiEdit HOẶC Write
}

{
  "matcher": "*"               // Tất cả tools
}

{
  "matcher": ""                // Lifecycle hooks (Stop, SessionStart) - bỏ trống
}
```

**Lưu ý quan trọng:**
- **Case-sensitive:** "Write" ≠ "write"
- **Matcher CHỈ áp dụng cho:** PreToolUse, PostToolUse, PostToolUseFailure, PermissionRequest
- **Lifecycle hooks** (Stop, SessionStart) **IGNORE** matcher

### 3.2 type: command vs type: prompt

#### 3.2.1 type: "command"

Chạy bash command và dựa vào exit code:

```json
{
  "type": "command",
  "command": "pnpm lint",
  "timeout": 30000
}
```

**Exit code behavior:**
- `0` = Success → Hook passes
- Non-zero = Failure → Hook blocks/fails

**Properties:**
- `command` (required): Shell command to execute
- `timeout` (optional): Timeout in milliseconds (default: 60000)

#### 3.2.2 type: "prompt"

Gửi prompt đến LLM để evaluate:

```json
{
  "type": "prompt",
  "prompt": "Check if the code changes follow MangoAds coding standards. Return PASS if compliant, FAIL otherwise."
}
```

**Khi nào dùng prompt:**
- Cần intelligent evaluation
- Decision không thể express bằng bash script
- Stop/SubagentStop quality gates

### 3.3 Environment Variables

Các biến môi trường available trong hook commands:

| Variable | Description | Available In |
|----------|-------------|--------------|
| `$CLAUDE_PROJECT_DIR` | Absolute path to project root | All hooks |
| `$CLAUDE_ENV_FILE` | File path for persisting env vars | All hooks |
| `$CLAUDE_PLUGIN_ROOT` | Path to plugin directory | Plugin hooks only |

**Ví dụ sử dụng:**

```bash
#!/bin/bash
# scripts/validate-bash.sh

# Access project directory
cd "$CLAUDE_PROJECT_DIR"

# Persist env var for future bash commands
echo "export MY_VAR=value" >> "$CLAUDE_ENV_FILE"

# Check for dangerous commands
if [[ "$1" =~ "rm -rf" ]]; then
    echo "BLOCKED: Dangerous rm -rf command"
    exit 1
fi

exit 0
```

### 3.4 Control Flow

#### 3.4.1 continue: false

Dừng processing sau khi hooks chạy:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "validate-command.sh"
          }
        ],
        "continue": false
      }
    ]
  }
}
```

**Behavior:**
- Cho **PreToolUse:** `continue: false` → Stop current tool attempt
- Cho **Stop/SubagentStop:** `continue: false` → Override block decision

#### 3.4.2 Multiple hooks

Hooks trong cùng array chạy tuần tự:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {"type": "command", "command": "pnpm lint"},      // Chạy trước
          {"type": "command", "command": "pnpm test"}       // Chạy sau
        ]
      }
    ]
  }
}
```

### 3.5 Hook Review Process (Bảo mật)

**Quan trọng:** Sửa trực tiếp `settings.json` KHÔNG có hiệu lực ngay trong session hiện tại.

**Workflow an toàn:**
1. Sửa `.claude/settings.json`
2. Chạy `/hooks` để review changes
3. Approve hooks mới
4. Hooks được activate

**Lý do:** Ngăn malicious code inject hooks trong session đang chạy.

---

## 4. SLASH COMMANDS

### 4.1 Định nghĩa

**Slash Commands** là shortcuts user gõ để trigger specific workflows. Đây là **user-invoked** (khác với Skills là model-invoked).

```
User: /weekly-report q1-2025
         │
         ▼
Claude loads: .claude/commands/weekly-report.md
         │
         ▼
Executes command with arguments
```

### 4.2 File Location

```
📁 Project commands
└── .claude/commands/
    ├── weekly-report.md
    ├── create-landing-page.md
    └── run-tests.md

📁 User commands (global)
└── ~/.claude/commands/
    ├── format-code.md
    └── daily-standup.md
```

### 4.3 Command File Structure

```markdown
---
description: Generates weekly marketing report from campaign data
allowed-tools: Read, Glob, Grep, WebFetch
argument-hint: "campaign-name"
model: sonnet
---

# Weekly Report Generator

Generate a comprehensive weekly report for the specified campaign.

## Instructions
1. Read campaign data from reports/ directory
2. Calculate key metrics
3. Compare with previous week
4. Generate formatted report

## Arguments
- $ARGUMENTS[0]: Campaign name

## Output Format
...
```

### 4.4 YAML Frontmatter Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Hiển thị trong `/help` |
| `allowed-tools` | string | Comma-separated list of tools |
| `argument-hint` | string | Gợi ý cho user về arguments |
| `model` | string | Model để dùng (sonnet, opus, haiku) |

### 4.5 Arguments Handling

**User input:**
```
/deploy staging feature-xyz
```

**Trong command markdown:**
```markdown
## Arguments
- Environment: $ARGUMENTS[0] = "staging"
- Feature: $ARGUMENTS[1] = "feature-xyz"

Deploy to $ARGUMENTS[0] environment with feature $ARGUMENTS[1].
```

### 4.6 Ví dụ Commands MangoAds

#### 4.6.1 UTM Generator Command

**File: `.claude/commands/utm.md`**

```markdown
---
description: Generate UTM tracking links for MangoAds campaigns
allowed-tools: Read
argument-hint: "campaign-name source medium [content] [term]"
model: haiku
---

# UTM Link Generator

Generate UTM tracking links following MangoAds conventions.

## Arguments
- campaign: $ARGUMENTS[0] (required)
- source: $ARGUMENTS[1] (required)
- medium: $ARGUMENTS[2] (required)
- content: $ARGUMENTS[3] (optional)
- term: $ARGUMENTS[4] (optional)

## Base URL
Ask user for base URL if not provided in prompt.

## Output
Generate full UTM link:
```
{base_url}?utm_source={source}&utm_medium={medium}&utm_campaign={campaign}[&utm_content={content}][&utm_term={term}]
```

## Validation
- All values lowercase
- No spaces (use hyphens)
- No Vietnamese diacritics
- Campaign format: {year}-{quarter}-{name}

## Example
Input: /utm q1-product-launch google cpc header-cta
Output: https://example.com?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-product-launch&utm_content=header-cta
```

**Usage:**
```
/utm q1-launch facebook paid_social
```

#### 4.6.2 Daily Standup Command

**File: `.claude/commands/standup.md`**

```markdown
---
description: Generate daily standup summary from git commits and issues
allowed-tools: Read, Glob, Grep, Bash
argument-hint: "[days-back]"
model: sonnet
---

# Daily Standup Generator

Generate standup summary based on recent activity.

## Arguments
- days: $ARGUMENTS[0] (default: 1)

## Data Sources
1. Git commits (last N days)
2. Modified files
3. TODO comments

## Output Format

```markdown
# Standup - [Date]

## Yesterday
- [Completed tasks from git commits]

## Today
- [Planned tasks based on TODOs and WIP]

## Blockers
- [Any identified blockers]
```

## Commands to Run
```bash
# Get recent commits
git log --oneline --since="$ARGUMENTS[0] days ago"

# Get modified files
git diff --name-only HEAD~5

# Find TODOs
grep -r "TODO" --include="*.ts" --include="*.tsx" src/
```
```

**Usage:**
```
/standup 2
```

### 4.7 Skill vs Slash Command - Khi nào dùng gì?

| Criteria | Slash Command | Skill |
|----------|---------------|-------|
| Invocation | User gõ `/command` | Claude tự chọn |
| Predictability | User biết chính xác khi nào chạy | Model decides |
| Arguments | Truyền trực tiếp | Context-based |
| Use case | Frequent workflows | Expertise areas |
| Discovery | `/help` shows list | Model-internal |

**Quy tắc MangoAds:**
- **Dùng Command khi:** User thường xuyên cần chạy cùng một task
- **Dùng Skill khi:** Expertise cần áp dụng tự động trong nhiều context

---

## 5. MCP SLASH COMMANDS

### 5.1 MCP Prompts → Slash Commands

Khi MCP server expose **prompts**, chúng tự động trở thành slash commands:

```
MCP Server: playwright
Prompt: create-test

→ Slash command: /mcp__playwright__create-test
```

### 5.2 Format

```
/mcp__<server-name>__<prompt-name> [arguments]
```

**Ví dụ:**
```
/mcp__playwright__create-test login-flow
/mcp__github__list-prs open
/mcp__slack__send-message #dev-team "Deployment complete"
```

### 5.3 Discovery

Xem tất cả MCP commands available:

```bash
claude
> /
# Shows all commands including MCP commands
```

Hoặc:
```bash
> /help
# Lists all commands with descriptions
```

### 5.4 Ví dụ MCP Commands MangoAds

**Giả sử đã cấu hình Playwright MCP:**

```
/mcp__playwright__screenshot https://mangoads.com/landing-page

/mcp__playwright__test-form https://mangoads.com/contact

/mcp__playwright__check-responsive https://mangoads.com --viewport=mobile
```

---

## 6. BEST PRACTICES & ANTI-PATTERNS

### 6.1 Hooks Best Practices

**✅ DO:**

```markdown
1. Specific matchers
   "matcher": "Bash"  # Chỉ check Bash, không check tất cả

2. Fast hooks
   # Hooks nên chạy nhanh (< 5s)
   # Tránh network calls trong PreToolUse

3. Meaningful messages
   echo "BLOCKED: Cannot delete production files"
   # User hiểu tại sao bị block

4. Idempotent hooks
   # Hook có thể chạy nhiều lần không gây side effects

5. Test hooks trước khi deploy
   # Verify hooks work correctly
```

**❌ DON'T (Anti-patterns):**

```markdown
1. Block quá nhiều
   # Hook block mọi thứ → Claude không làm được gì

2. Slow hooks
   # Hook mất 30s → UX tệ

3. Silent failures
   exit 1  # Không có message → User confused

4. Catch-all matchers unnecessarily
   "matcher": "*"  # Không cần thiết cho hầu hết cases
```

### 6.2 Slash Commands Best Practices

**✅ DO:**

```markdown
1. Clear description
   description: Generates weekly marketing report with metrics comparison

2. Helpful argument-hint
   argument-hint: "campaign-name [start-date] [end-date]"

3. Document in README
   # List all available commands với examples

4. Consistent naming
   /report-weekly, /report-monthly (không phải /weekly-report, /monthly_report)
```

**❌ DON'T (Anti-patterns):**

```markdown
1. Vague description
   description: Does stuff

2. No argument hints
   # User không biết truyền gì

3. Too many commands
   # 50 commands → Khó remember

4. Overlapping commands
   /report và /generate-report làm cùng việc
```

### 6.3 Security Checklist

```markdown
## Hooks Security Checklist

### PreToolUse
- [ ] Block rm -rf on important directories
- [ ] Block writes to .env files
- [ ] Block pushes to main/master without PR
- [ ] Log all Bash commands

### Stop
- [ ] Run security lint (eslint-plugin-security)
- [ ] Check for hardcoded secrets
- [ ] Validate no TODO security comments

### SessionStart
- [ ] Verify not in production environment
- [ ] Check required tools installed
- [ ] Validate .env exists (without exposing values)

### General
- [ ] All hook scripts in version control
- [ ] Hooks reviewed in PR
- [ ] Test hooks in dev before prod
- [ ] Document hook purposes
```

### 6.4 Ví dụ Full Hook Configuration MangoAds

**File: `.claude/settings.json`**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/hooks/validate-bash.sh",
            "timeout": 5000
          }
        ]
      },
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/hooks/check-protected-files.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint --quiet"
          },
          {
            "type": "command",
            "command": "pnpm typecheck"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review the subagent's output. Has it completed the task correctly and completely? Check for errors, incomplete work, or missing requirements."
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/hooks/setup-session.sh"
          }
        ]
      }
    ]
  }
}
```

**Script: `scripts/hooks/validate-bash.sh`**

```bash
#!/bin/bash
set -e

# Get the command from stdin or argument
COMMAND="$1"

# Dangerous patterns
DANGEROUS_PATTERNS=(
    "rm -rf /"
    "rm -rf /*"
    "rm -rf ~"
    "> /dev/sda"
    "mkfs"
    "dd if="
    ":(){:|:&};:"
)

# Check for dangerous patterns
for pattern in "${DANGEROUS_PATTERNS[@]}"; do
    if [[ "$COMMAND" == *"$pattern"* ]]; then
        echo "BLOCKED: Dangerous command detected: $pattern"
        exit 1
    fi
done

# Block production database access
if [[ "$COMMAND" == *"production"* ]] && [[ "$COMMAND" == *"database"* ]]; then
    echo "BLOCKED: Production database access not allowed from Claude Code"
    exit 1
fi

# Block credential files
if [[ "$COMMAND" == *".env"* ]] || [[ "$COMMAND" == *"credentials"* ]]; then
    echo "WARNING: Credential file access - proceeding with caution"
fi

exit 0
```

**Script: `scripts/hooks/check-protected-files.sh`**

```bash
#!/bin/bash
set -e

# Protected files that should not be modified
PROTECTED_FILES=(
    ".env"
    ".env.production"
    "package-lock.json"
    "pnpm-lock.yaml"
    ".claude/settings.json"
)

# Get the file path from argument
FILE_PATH="$1"

for protected in "${PROTECTED_FILES[@]}"; do
    if [[ "$FILE_PATH" == *"$protected"* ]]; then
        echo "BLOCKED: Cannot modify protected file: $protected"
        exit 1
    fi
done

exit 0
```

---

## TÓM TẮT PART 3

### Đã cover trong Part 3:
- [x] Hooks - Định nghĩa và tại sao cần
- [x] Lifecycle Hook Events (PreToolUse, Stop, SubagentStop, SessionStart, etc.)
- [x] Hook Configuration (matcher, type command/prompt, environment variables)
- [x] Control flow và Hook Review Process
- [x] Slash Commands - Structure và YAML fields
- [x] MCP Slash Commands format
- [x] Best practices và Security checklist
- [x] Ví dụ full configuration MangoAds

### Part 4 sẽ cover:
- [ ] Plugins - Bundle và distribution
- [ ] MCP (Model Context Protocol) - Chi tiết
- [ ] MCP Server vs Client
- [ ] Playwright MCP chi tiết
- [ ] Browser DevTools MCP
- [ ] Security model của MCP

---

**Tiếp theo:** [Part 4: Plugins & MCP](./part-04-plugins-mcp.md)
