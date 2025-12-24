# CLAUDE CODE CHO DEVELOPERS - MANGOADS
## Phần 2: Tác tử con (Subagents) & Móc sự kiện (Hooks)

**Phiên bản:** 1.0
**Đối tượng:** Frontend, Backend, QA, DevOps Team (bao gồm Junior & Intern)
**Thời lượng:** 60-90 phút

---

## NHẮC LẠI THUẬT NGỮ

> 📚 **Dành cho Junior/Intern**: Nếu chưa đọc Phần 1, hãy xem lại bảng thuật ngữ trước.

| Thuật ngữ | Tiếng Việt | Tóm tắt |
|-----------|------------|---------|
| **Subagent** | Tác tử con | AI chuyên gia được gọi để làm task cụ thể |
| **Skill** | Kỹ năng | Kiến thức Claude tự động dùng trong conversation |
| **Hook** | Móc sự kiện | Code tự động chạy khi có sự kiện |
| **YAML Frontmatter** | Phần header YAML | Cấu hình ở đầu file markdown, nằm giữa `---` |
| **Matcher** | Bộ lọc | Pattern để xác định hook áp dụng cho tool nào |

---

## MỤC LỤC

1. [Tìm hiểu sâu về Tác tử con (Subagents)](#1-tìm-hiểu-sâu-về-tác-tử-con)
2. [Kỹ năng cho Tác tử (Agent Skills)](#2-kỹ-năng-cho-tác-tử)
3. [Hệ thống Móc sự kiện (Hooks)](#3-hệ-thống-móc-sự-kiện)
4. [Ví dụ thực hành](#4-ví-dụ-thực-hành)

---

## 1. TÌM HIỂU SÂU VỀ TÁC TỬ CON (SUBAGENTS)

### 1.1 Khái niệm Subagent

> 💡 **Giải thích cho Junior**: Subagent giống như việc bạn có nhiều đồng nghiệp chuyên gia. Thay vì một người làm mọi thứ, bạn giao việc cho người giỏi nhất về lĩnh vực đó.

**Subagent** = AI chuyên gia với các đặc điểm:
- **Ngữ cảnh riêng biệt** (không ảnh hưởng đến cuộc hội thoại chính)
- **Công cụ giới hạn** (chỉ có quyền cần thiết)
- **Prompt hệ thống tùy chỉnh** (hướng dẫn riêng)
- **Chuyên môn cụ thể** (giỏi một việc)

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

### 1.2 Cấu trúc file Subagent

> 💡 **Giải thích cho Junior**: Mỗi subagent được định nghĩa trong một file markdown riêng. File này chứa cấu hình YAML và hướng dẫn chi tiết.

**Vị trí:** `.claude/agents/[tên].md`

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

### 1.3 Tham khảo các trường YAML

> 💡 **Giải thích cho Junior**: Đây là các trường cấu hình trong phần YAML frontmatter. Claude đọc `description` để biết khi nào nên gọi subagent này.

| Trường | Bắt buộc | Kiểu | Mô tả |
|--------|----------|------|-------|
| `name` | Có | string | Tên định danh (dùng kebab-case như `code-reviewer`) |
| `description` | Có | string | Mô tả khi nào dùng (Claude dựa vào đây!) |
| `model` | Không | enum | `haiku`, `sonnet`, `opus`, `inherit` |
| `tools` | Không | string | Danh sách công cụ, phân cách bằng dấu phẩy |
| `permissionMode` | Không | enum | `default`, `acceptEdits`, `plan`, `bypassPermissions` |
| `skills` | Không | string | Danh sách kỹ năng, phân cách bằng dấu phẩy |

### 1.4 Hướng dẫn chọn Model

> 💡 **Giải thích cho Junior**: Model là "bộ não" của AI. Model mạnh hơn thì thông minh hơn nhưng chậm và đắt hơn.

| Model | Trường hợp sử dụng | Chi phí | Tốc độ |
|-------|-------------------|---------|--------|
| `haiku` | Task đơn giản, cần nhanh | Thấp | Nhanh |
| `sonnet` | Cân bằng (mặc định) | Trung bình | Trung bình |
| `opus` | Suy luận phức tạp | Cao | Chậm |
| `inherit` | Kế thừa từ parent | - | - |

**Khuyến nghị của MangoAds:**
```yaml
# Quick formatting/linting
model: haiku

# Standard code review
model: sonnet

# Architecture decisions, security audit
model: opus
```

### 1.5 Các mẫu giới hạn công cụ (Tool Scoping Patterns)

> 💡 **Giải thích cho Junior**: Nguyên tắc "quyền tối thiểu" - chỉ cấp đủ quyền cần thiết. Subagent chỉ review code thì không cần quyền sửa file.

**Mẫu 1: Kiểm tra viên chỉ đọc (Read-Only Auditor)**
```yaml
tools: Read, Glob, Grep
permissionMode: plan
```
- Không sửa file
- Không chạy lệnh
- An toàn cho security/compliance

**Mẫu 2: Người viết code (Code Writer)**
```yaml
tools: Read, Write, Edit, Glob, Grep
permissionMode: acceptEdits
```
- Có thể tạo/sửa file
- Không có quyền bash
- Được giám sát bởi hooks

**Mẫu 3: Developer đầy đủ quyền (Full Developer)**
```yaml
tools: Read, Write, Edit, Bash, Glob, Grep
permissionMode: default
```
- Quyền đầy đủ
- Vẫn hỏi permission
- Cho các task đáng tin cậy

**Mẫu 4: QA Tester với MCP**
```yaml
tools: Read, Glob, Grep, mcp__playwright
permissionMode: default
```
- Chỉ đọc code
- Tự động hóa browser qua MCP

---

## 2. KỸ NĂNG CHO TÁC TỬ (AGENT SKILLS)

### 2.1 Khái niệm Skill

> 💡 **Giải thích cho Junior**: Skill giống như "kiến thức chuyên môn" được cài sẵn. Khi Claude cần kiến thức đó, nó tự động dùng skill phù hợp.

**Skill** = Kiến thức tái sử dụng mà Claude tự động gọi khi cần.

**Khác biệt chính với Subagent:**

| Đặc điểm | Subagent | Skill |
|----------|----------|-------|
| Ngữ cảnh | Riêng biệt, cô lập | Chia sẻ với caller |
| Công cụ | Có tools riêng | Không có tools riêng |
| Tính di động | Cố định trong dự án | Dễ tái sử dụng |

Nói đơn giản:
- **Subagent**: AI riêng biệt, chạy độc lập
- **Skill**: Kiến thức Claude dùng trong cuộc hội thoại hiện tại

### 2.2 Cấu trúc file Skill

> 💡 **Giải thích cho Junior**: Skill được định nghĩa trong file markdown. File này chứa kiến thức chi tiết mà Claude sẽ dùng.

**Vị trí:** `.claude/skills/[tên-skill]/SKILL.md`

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

### 2.3 Cách Claude phát hiện Skill

> 💡 **Giải thích cho Junior**: Claude tự động tìm và dùng skill phù hợp dựa trên yêu cầu của bạn.

Claude phát hiện skills khi khởi động:
1. Quét các thư mục `.claude/skills/`
2. Đọc tên + mô tả skill
3. Nội dung đầy đủ được load khi cần dùng

```
Người dùng: "Kiểm tra code này có lỗi bảo mật không"

Claude (xử lý nội bộ):
"Đây là yêu cầu về bảo mật..."
"Tôi có skill: security-patterns"
"Mô tả khớp - gọi skill này"
```

---

## 3. HỆ THỐNG MÓC SỰ KIỆN (HOOKS)

### 3.1 Các sự kiện Hook

> 💡 **Giải thích cho Junior**: Hook giống như "bẫy" được đặt sẵn. Khi có sự kiện xảy ra, hook sẽ tự động chạy. Giống như event listeners trong JavaScript.

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

### 3.2 Cấu hình Hook

> 💡 **Giải thích cho Junior**: Hooks được cấu hình trong file settings.json. Mỗi hook có `matcher` để xác định áp dụng cho tool nào.

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

### 3.3 Các loại Hook

> 💡 **Giải thích cho Junior**: Có 2 loại hook chính - chạy lệnh shell hoặc gửi prompt cho AI đánh giá.

**Loại: "command" (chạy lệnh shell)**
```json
{
  "type": "command",
  "command": "pnpm lint",
  "timeout": 30000
}
```
- Chạy lệnh shell
- Exit 0 = thành công, khác 0 = thất bại
- Có thể chặn thao tác

**Loại: "prompt" (gửi prompt cho AI)**
```json
{
  "type": "prompt",
  "prompt": "Check if the changes are complete and correct."
}
```
- Gửi prompt cho LLM đánh giá
- Cho đánh giá thông minh
- Tốt nhất cho Stop/SubagentStop

### 3.4 Cú pháp Matcher (Bộ lọc)

> 💡 **Giải thích cho Junior**: Matcher xác định hook sẽ chạy cho tool nào. Dùng `|` để chọn nhiều tools.

| Matcher | Khớp với |
|---------|---------|
| `"Bash"` | Chỉ Bash tool |
| `"Edit\|MultiEdit\|Write"` | Bất kỳ tool nào trong danh sách |
| `"*"` | Tất cả tools |
| `""` | Các hook vòng đời (Stop, SessionStart) |

**Lưu ý:** Phân biệt chữ hoa/thường (case-sensitive)!

### 3.5 Biến môi trường (Environment Variables)

> 💡 **Giải thích cho Junior**: Các biến này có sẵn trong hook scripts để bạn sử dụng.

| Biến | Mô tả |
|----------|-------------|
| `$CLAUDE_PROJECT_DIR` | Đường dẫn thư mục gốc dự án |
| `$CLAUDE_ENV_FILE` | File để lưu biến môi trường |
| `$CLAUDE_PLUGIN_ROOT` | Thư mục plugin (trong plugins) |

### 3.6 Ví dụ Hook Scripts

> 💡 **Giải thích cho Junior**: Đây là các script shell được hook gọi. Bạn có thể viết logic kiểm tra tùy ý.

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

## 4. VÍ DỤ THỰC HÀNH

### 4.1 Subagent Review Code

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

### 4.2 Subagent Sinh Test

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

### 4.3 Cấu hình Hook hoàn chỉnh

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

## CHECKLIST HOÀN THÀNH PHẦN 2

### Tác tử con (Subagents)
- [ ] Hiểu kiến trúc subagent và cách hoạt động
- [ ] Biết các trường YAML và ý nghĩa từng trường
- [ ] Tạo được code-reviewer subagent
- [ ] Biết các mẫu giới hạn công cụ (tool scoping patterns)

### Kỹ năng (Skills)
- [ ] Phân biệt được skill và subagent
- [ ] Tạo được file skill
- [ ] Hiểu cơ chế Claude phát hiện skill

### Móc sự kiện (Hooks)
- [ ] Hiểu các loại hook events
- [ ] Tạo được settings.json với cấu hình hooks
- [ ] Viết được hook scripts
- [ ] Hiểu cú pháp matcher

---

## TÓM TẮT PHẦN 2

### Đã học:
- [x] Tác tử con (Subagents) - Ngữ cảnh riêng biệt, giới hạn công cụ
- [x] Kỹ năng (Skills) - Kiến thức tái sử dụng
- [x] Móc sự kiện (Hooks) - Kiểm soát vòng đời, validation
- [x] Ví dụ thực hành (code-reviewer, test-generator)

### Phần 3 sẽ học:
- [ ] MCP (Giao thức kết nối) - Tìm hiểu sâu
- [ ] Playwright MCP cho testing
- [ ] DevTools MCP cho debugging
- [ ] Quy trình testing tự động

---

**Tiếp theo:** [Phần 3: MCP & Testing tự động](./part-03-mcp-testing.md)
