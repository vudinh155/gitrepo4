# CLAUDE CODE CHO DEVELOPERS - MANGOADS
## Part 2: Subagents & Hooks

**Phiên bản:** 1.0
**Đối tượng:** Frontend, Backend, QA, DevOps Team
**Thời lượng:** 60-90 phút

---

## MỤC LỤC

1. [Subagents Deep Dive](#1-subagents-deep-dive)
2. [Agent Skills](#2-agent-skills)
3. [Hooks System](#3-hooks-system)
4. [Practical Examples](#4-practical-examples)

---

## 1. SUBAGENTS DEEP DIVE

### 1.1 Subagent Concept

**Subagent** = Specialized AI worker với:
- Isolated context (không ô nhiễm main conversation)
- Scoped tools (least privilege)
- Custom system prompt
- Task-specific expertise

```
┌─────────────────────────────────────────────────────────────┐
│                     MAIN CONVERSATION                        │
│                                                              │
│  User: "Review this PR và test form submission"             │
│                                                              │
│  Claude: Tôi sẽ gọi 2 subagents...                          │
│                                                              │
│  ┌───────────────────────┐  ┌───────────────────────┐       │
│  │    code-reviewer      │  │    form-tester        │       │
│  │                       │  │                       │       │
│  │ Context: PR diff only │  │ Context: form specs   │       │
│  │ Tools: Read, Grep     │  │ Tools: Playwright MCP │       │
│  │                       │  │                       │       │
│  │ Output: Review report │  │ Output: Test results  │       │
│  └───────────────────────┘  └───────────────────────┘       │
│                                                              │
│  Claude: Đây là kết quả từ 2 subagents...                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Subagent File Structure

**Location:** `.claude/agents/[name].md`

```markdown
---
name: code-reviewer
description: Reviews code changes for bugs, security issues, performance problems, and style violations
model: opus
tools: Read, Glob, Grep
permissionMode: plan
skills: security-patterns, performance-patterns
---

# Code Reviewer

You are a senior developer performing code reviews.

## Review Checklist
1. Security vulnerabilities
2. Performance issues
3. Code style violations
4. Test coverage
5. Documentation

## Severity Levels
- 🔴 Critical: Must fix before merge
- 🟡 Warning: Should fix
- 🟢 Suggestion: Nice to have

## Output Format
```markdown
# Code Review: [PR/File Name]

## Summary
[1-2 sentence overview]

## Critical Issues
[List if any]

## Warnings
[List if any]

## Suggestions
[List if any]

## Approval Status
[ ] Approved
[ ] Request Changes
```
```

### 1.3 YAML Fields Reference

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `name` | Yes | string | Unique identifier (kebab-case) |
| `description` | Yes | string | When to invoke (Claude uses this!) |
| `model` | No | enum | `haiku`, `sonnet`, `opus`, `inherit` |
| `tools` | No | string | Comma-separated list |
| `permissionMode` | No | enum | `default`, `acceptEdits`, `plan`, `bypassPermissions` |
| `skills` | No | string | Comma-separated skill names |

### 1.4 Model Selection Guide

| Model | Use Case | Cost | Speed |
|-------|----------|------|-------|
| `haiku` | Simple, fast tasks | Low | Fast |
| `sonnet` | Balanced (default) | Medium | Medium |
| `opus` | Complex reasoning | High | Slow |
| `inherit` | Use parent's model | - | - |

**MangoAds Recommendations:**
```yaml
# Quick formatting/linting
model: haiku

# Standard code review
model: sonnet

# Architecture decisions, security audit
model: opus
```

### 1.5 Tool Scoping Patterns

**Pattern 1: Read-Only Auditor**
```yaml
tools: Read, Glob, Grep
permissionMode: plan
```
- No file modifications
- No command execution
- Safe for security/compliance

**Pattern 2: Code Writer**
```yaml
tools: Read, Write, Edit, Glob, Grep
permissionMode: acceptEdits
```
- Can create/modify files
- No bash access
- Supervised by hooks

**Pattern 3: Full Developer**
```yaml
tools: Read, Write, Edit, Bash, Glob, Grep
permissionMode: default
```
- Full access
- Still asks permissions
- For trusted tasks

**Pattern 4: QA Tester with MCP**
```yaml
tools: Read, Glob, Grep, mcp__playwright
permissionMode: default
```
- Read-only code access
- Browser automation via MCP

---

## 2. AGENT SKILLS

### 2.1 Skill Concept

**Skill** = Reusable expertise that Claude can invoke automatically.

Key differences from Subagent:
- Skills share context with caller
- Skills don't have isolated tools
- Skills are more portable

### 2.2 Skill File Structure

**Location:** `.claude/skills/[skill-name]/SKILL.md`

```markdown
---
name: security-patterns
description: Identifies common security vulnerabilities and suggests fixes
---

# Security Patterns

## Purpose
Detect and fix security vulnerabilities in code.

## Vulnerabilities Covered

### 1. SQL Injection
**Pattern:**
```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE id = ${userId}`

// ✅ Safe
const query = 'SELECT * FROM users WHERE id = ?'
db.query(query, [userId])
```

### 2. XSS (Cross-Site Scripting)
**Pattern:**
```javascript
// ❌ Vulnerable
element.innerHTML = userInput

// ✅ Safe
element.textContent = userInput
// or use DOMPurify for HTML
element.innerHTML = DOMPurify.sanitize(userInput)
```

### 3. Command Injection
**Pattern:**
```javascript
// ❌ Vulnerable
exec(`ls ${userPath}`)

// ✅ Safe
execFile('ls', [userPath])
```

### 4. Path Traversal
**Pattern:**
```javascript
// ❌ Vulnerable
const file = fs.readFileSync(`./uploads/${filename}`)

// ✅ Safe
const safePath = path.join('./uploads', path.basename(filename))
const file = fs.readFileSync(safePath)
```

## Instructions
1. Scan code for vulnerable patterns
2. Identify specific vulnerability type
3. Show vulnerable code
4. Suggest fixed code
5. Explain the risk

## Severity Classification
- 🔴 Critical: Remote code execution, data breach
- 🟡 High: Auth bypass, privilege escalation
- 🟠 Medium: Information disclosure
- 🟢 Low: Best practice violation
```

### 2.3 Skill Discovery

Claude discovers skills at startup:
1. Scans `.claude/skills/` directories
2. Loads skill names + descriptions
3. Full content loaded when invoked

```
User: "Check this code for security issues"

Claude (internally):
"Security-related request..."
"I have skill: security-patterns"
"Description matches - invoking skill"
```

---

## 3. HOOKS SYSTEM

### 3.1 Hook Events

```
┌─────────────────────────────────────────────────────────────┐
│                     HOOK EVENTS                              │
│                                                              │
│  SessionStart ─────────────────────────────────────────────► │
│       │                                                      │
│       ▼                                                      │
│  UserPromptSubmit ◄────────────────────────────────────────┐ │
│       │                                                    │ │
│       ▼                                                    │ │
│  PreToolUse ──► Tool ──► PostToolUse / PostToolUseFailure │ │
│       │                                                    │ │
│       ▼                                                    │ │
│  SubagentStop (if subagent)                               │ │
│       │                                                    │ │
│       ▼                                                    │ │
│  Stop ─────────────────────────────────────────────────────┘ │
│       │                                                      │
│       ▼                                                      │
│  PreCompact (if context full)                               │
│       │                                                      │
│       ▼                                                      │
│  SessionEnd                                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Hook Configuration

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
            "command": "$CLAUDE_PROJECT_DIR/scripts/validate-bash.sh",
            "timeout": 5000
          }
        ]
      },
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/check-protected.sh"
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
          },
          {
            "type": "prompt",
            "prompt": "Verify all changes follow the project coding standards defined in CLAUDE.md"
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
            "prompt": "Validate the subagent completed its task correctly and completely."
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
            "command": "$CLAUDE_PROJECT_DIR/scripts/setup-dev.sh"
          }
        ]
      }
    ]
  }
}
```

### 3.3 Hook Types

**type: "command"**
```json
{
  "type": "command",
  "command": "pnpm lint",
  "timeout": 30000
}
```
- Executes shell command
- Exit 0 = pass, non-zero = fail
- Can block operations

**type: "prompt"**
```json
{
  "type": "prompt",
  "prompt": "Check if the changes are complete and correct."
}
```
- Sends prompt to LLM
- For intelligent evaluation
- Best for Stop/SubagentStop

### 3.4 Matcher Syntax

| Matcher | Matches |
|---------|---------|
| `"Bash"` | Only Bash tool |
| `"Edit\|MultiEdit\|Write"` | Any of these |
| `"*"` | All tools |
| `""` | Lifecycle hooks (Stop, SessionStart) |

**Note:** Case-sensitive!

### 3.5 Environment Variables

| Variable | Description |
|----------|-------------|
| `$CLAUDE_PROJECT_DIR` | Project root path |
| `$CLAUDE_ENV_FILE` | File for persisting env vars |
| `$CLAUDE_PLUGIN_ROOT` | Plugin directory (in plugins) |

### 3.6 Hook Scripts

**scripts/validate-bash.sh**
```bash
#!/bin/bash
set -e

COMMAND="$1"

# Block dangerous patterns
DANGEROUS=(
    "rm -rf /"
    "rm -rf ~"
    "> /dev/sda"
    ":(){ :|:& };"
)

for pattern in "${DANGEROUS[@]}"; do
    if [[ "$COMMAND" == *"$pattern"* ]]; then
        echo "BLOCKED: Dangerous command pattern detected"
        exit 1
    fi
done

# Block production database
if [[ "$COMMAND" == *"production"* ]] && [[ "$COMMAND" == *"database"* ]]; then
    echo "BLOCKED: Production database access not allowed"
    exit 1
fi

exit 0
```

**scripts/check-protected.sh**
```bash
#!/bin/bash
set -e

FILE_PATH="$1"

PROTECTED=(
    ".env"
    ".env.production"
    "package-lock.json"
    "pnpm-lock.yaml"
)

for protected in "${PROTECTED[@]}"; do
    if [[ "$FILE_PATH" == *"$protected"* ]]; then
        echo "BLOCKED: Cannot modify protected file: $protected"
        exit 1
    fi
done

exit 0
```

**scripts/setup-dev.sh**
```bash
#!/bin/bash

echo "Setting up development environment..."

# Check Node version
if ! node -v | grep -q "v20"; then
    echo "Warning: Expected Node.js 20.x"
fi

# Check dependencies
if [ ! -d "node_modules" ]; then
    echo "Installing dependencies..."
    pnpm install
fi

# Persist environment
echo "export PROJECT_READY=true" >> "$CLAUDE_ENV_FILE"

echo "Setup complete!"
```

---

## 4. PRACTICAL EXAMPLES

### 4.1 Code Reviewer Subagent

**File: `.claude/agents/code-reviewer.md`**

```markdown
---
name: code-reviewer
description: Performs thorough code reviews checking for bugs, security issues, performance problems, and style violations
model: opus
tools: Read, Glob, Grep
permissionMode: plan
skills: security-patterns, performance-patterns
---

# Code Reviewer - MangoAds

You are a senior developer at MangoAds performing code reviews.

## Review Process

### Step 1: Understand Context
- Read the files/changes being reviewed
- Understand the purpose of the changes

### Step 2: Check Categories
1. **Correctness**
   - Logic errors
   - Edge cases
   - Error handling

2. **Security**
   - Use security-patterns skill
   - Check for OWASP Top 10

3. **Performance**
   - N+1 queries
   - Unnecessary re-renders
   - Memory leaks

4. **Code Quality**
   - Naming conventions
   - Code duplication
   - Comments/documentation

5. **Testing**
   - Test coverage
   - Test quality

### Step 3: Categorize Findings

🔴 **Critical** - Must fix before merge
- Security vulnerabilities
- Data loss risks
- Breaking changes

🟡 **Warning** - Should fix
- Performance issues
- Code smell
- Missing tests

🟢 **Suggestion** - Nice to have
- Style improvements
- Refactoring opportunities

## Output Format

```markdown
# Code Review: [File/PR Name]

## Summary
[1-2 sentence overview]

## Findings

### 🔴 Critical Issues
1. **[Issue Title]** (line X-Y)
   - Problem: [description]
   - Impact: [why it matters]
   - Fix: [suggested fix]

### 🟡 Warnings
1. **[Issue Title]** (line X)
   - [description and fix]

### 🟢 Suggestions
1. **[Suggestion Title]**
   - [description]

## Approval
- [ ] ✅ Approved
- [ ] 🔄 Approved with suggestions
- [ ] ❌ Request changes (critical issues found)
```
```

### 4.2 Test Generator Subagent

**File: `.claude/agents/test-generator.md`**

```markdown
---
name: test-generator
description: Generates unit tests and integration tests for components and functions
model: sonnet
tools: Read, Write, Edit, Glob, Grep
permissionMode: acceptEdits
---

# Test Generator - MangoAds

You generate comprehensive tests for code.

## Test Framework
- Unit tests: Vitest
- Component tests: React Testing Library
- E2E tests: Playwright

## Test Patterns

### Unit Test Template
```typescript
import { describe, it, expect, vi } from 'vitest'
import { functionName } from './module'

describe('functionName', () => {
  it('should handle normal case', () => {
    const result = functionName(input)
    expect(result).toBe(expected)
  })

  it('should handle edge case', () => {
    // ...
  })

  it('should throw on invalid input', () => {
    expect(() => functionName(invalid)).toThrow()
  })
})
```

### Component Test Template
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { ComponentName } from './ComponentName'

describe('ComponentName', () => {
  it('should render correctly', () => {
    render(<ComponentName prop="value" />)
    expect(screen.getByText('expected text')).toBeInTheDocument()
  })

  it('should handle user interaction', async () => {
    render(<ComponentName />)
    fireEvent.click(screen.getByRole('button'))
    expect(screen.getByText('updated text')).toBeInTheDocument()
  })
})
```

## Coverage Requirements
- Functions: Test all branches
- Components: Test render, interactions, edge cases
- APIs: Test success, error, validation

## Output
- Place tests in __tests__/ or alongside source with .test.ts(x)
- Follow project conventions from CLAUDE.md
```

### 4.3 Complete Hook Setup

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
            "command": "$CLAUDE_PROJECT_DIR/scripts/hooks/check-protected.sh"
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
            "command": "pnpm lint --quiet || true"
          },
          {
            "type": "command",
            "command": "pnpm typecheck || true"
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
            "prompt": "Verify the subagent output is complete and follows MangoAds coding standards."
          }
        ]
      }
    ]
  }
}
```

---

## CHECKLIST HOÀN THÀNH PART 2

### Subagents
- [ ] Hiểu subagent architecture
- [ ] Biết YAML fields và ý nghĩa
- [ ] Tạo được code-reviewer subagent
- [ ] Biết tool scoping patterns

### Skills
- [ ] Hiểu skill vs subagent
- [ ] Tạo được skill file
- [ ] Hiểu skill discovery mechanism

### Hooks
- [ ] Hiểu hook events
- [ ] Tạo được settings.json với hooks
- [ ] Viết được hook scripts
- [ ] Hiểu matcher syntax

---

## TÓM TẮT PART 2

### Đã học:
- [x] Subagents - Context isolation, tool scoping
- [x] Agent Skills - Reusable expertise
- [x] Hooks - Lifecycle control, validation
- [x] Practical examples (code-reviewer, test-generator)

### Part 3 sẽ học:
- [ ] MCP (Model Context Protocol) deep dive
- [ ] Playwright MCP for testing
- [ ] Browser DevTools MCP for debugging
- [ ] Automated testing workflows

---

**Tiếp theo:** [Part 3: MCP & Testing Automation](./part-03-mcp-testing.md)
