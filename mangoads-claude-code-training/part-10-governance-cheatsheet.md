# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 10: Governance, Safety, Rubric & Cheatsheet

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-9

---

## MỤC LỤC PART 10

1. [Governance & Safety](#1-governance--safety)
2. [Rubric Đánh Giá Subagent](#2-rubric-đánh-giá-subagent)
3. [Cheatsheet 1 Trang](#3-cheatsheet-1-trang)
4. [Tổng Kết Chương Trình Training](#4-tổng-kết-chương-trình-training)

---

## 1. GOVERNANCE & SAFETY

### 1.1 MCP Sandbox & Isolation

```
┌─────────────────────────────────────────────────────────────┐
│                   MCP SECURITY LAYERS                        │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 1: Process Isolation                             │ │
│  │ - Each MCP server runs in separate process             │ │
│  │ - Claude Code ≠ MCP Server process                     │ │
│  │ - Communication via JSON-RPC                           │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 2: Network Controls                              │ │
│  │ - MCP servers có network access riêng                  │ │
│  │ - Có thể restrict network per server                   │ │
│  │ - Firewall rules áp dụng                               │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 3: Permission Model                              │ │
│  │ - User phải approve MCP tool usage                     │ │
│  │ - Subagent tool scoping                                │ │
│  │ - Permission modes                                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 4: Credential Separation                         │ │
│  │ - Credentials KHÔNG trong Claude Code sandbox          │ │
│  │ - MCP server manages own credentials                   │ │
│  │ - Environment variables at startup                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 No Auto-run Browser Policy

**Quy định MangoAds:**

```markdown
## BROWSER MCP POLICY

### KHÔNG được phép:
❌ Auto-run browser tests on production
❌ Execute browser actions without user awareness
❌ Store credentials in MCP config
❌ Run MCP browsers in headless mode for sensitive operations

### BẮT BUỘC:
✅ User phải approve browser launch
✅ Visible browser window for testing
✅ Human review trước khi test production
✅ Logging all browser actions

### Permission Matrix
| Environment | Auto-run | Approval Needed |
|-------------|----------|-----------------|
| localhost | ✅ Allowed | Team Lead |
| staging | ⚠️ With caution | Team Lead |
| production | ❌ Never auto | Director + Security |
```

### 1.3 Logging & Audit

**Log Requirements:**

```markdown
## LOGGING REQUIREMENTS

### What to Log
1. **Session logs**
   - Start/end time
   - User identity
   - Actions performed

2. **Tool usage**
   - Tool name
   - Parameters (sanitized)
   - Result status
   - Duration

3. **MCP operations**
   - Server invoked
   - Tools used
   - External calls made

4. **File changes**
   - Files created/modified/deleted
   - Before/after (for edits)
   - Git commits

### Log Storage
- Location: [defined by org policy]
- Retention: Minimum 90 days
- Access: Security team + authorized personnel

### Log Format
```json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "session_id": "abc123",
  "user": "developer@mangoads.com",
  "action": "mcp_tool_call",
  "details": {
    "server": "playwright",
    "tool": "navigate",
    "params": { "url": "https://staging.mangoads.com" },
    "status": "success",
    "duration_ms": 1500
  }
}
```
```

### 1.4 Review Output Policy

```markdown
## OUTPUT REVIEW POLICY

### Review Required For:
1. **Code changes to production**
   - All edits reviewed before merge
   - Subagent output verified

2. **Data operations**
   - Database modifications
   - File system changes outside project
   - API calls to external services

3. **MCP operations**
   - Browser actions on non-localhost
   - GitHub operations (PR, issues)
   - Slack messages to public channels

### Review Process
```
Claude Output → Human Review → Approve/Reject → Execute/Revise
```

### Reviewer Responsibilities
- [ ] Verify output matches request
- [ ] Check for security issues
- [ ] Validate data accuracy
- [ ] Confirm no unintended changes
```

### 1.5 Security Checklist Master

```markdown
## MANGOADS CLAUDE CODE SECURITY CHECKLIST

### Setup Phase
- [ ] CLAUDE.md không chứa credentials
- [ ] .mcp.json reviewed và approved
- [ ] Hooks audited cho security
- [ ] Permission modes appropriate

### Development Phase
- [ ] Sensitive files protected via hooks
- [ ] Production access blocked
- [ ] Logging enabled
- [ ] Regular session reviews

### Deployment Phase
- [ ] All outputs reviewed
- [ ] No auto-deploy without approval
- [ ] Rollback plan ready
- [ ] Incident response documented

### Ongoing
- [ ] Weekly security audit
- [ ] Monthly access review
- [ ] Quarterly policy update
- [ ] Annual training refresh
```

### 1.6 Incident Response

```markdown
## INCIDENT RESPONSE PLAN

### Severity Levels

| Level | Description | Response Time |
|-------|-------------|---------------|
| P1 | Data breach, production down | Immediate |
| P2 | Security vulnerability | < 4 hours |
| P3 | Policy violation | < 24 hours |
| P4 | Minor issue | < 1 week |

### Response Steps

#### P1/P2 Incidents
1. **Contain** (0-15 min)
   - Stop Claude Code session
   - Revoke MCP access
   - Block affected systems

2. **Assess** (15-60 min)
   - Review logs
   - Identify scope
   - Document impact

3. **Remediate** (1-4 hours)
   - Apply fixes
   - Verify resolution
   - Monitor

4. **Report** (24 hours)
   - Incident report
   - Root cause analysis
   - Prevention measures

### Contact List
- Security Team: security@mangoads.com
- Tech Lead: techlead@mangoads.com
- Management: management@mangoads.com
```

---

## 2. RUBRIC ĐÁNH GIÁ SUBAGENT

### 2.1 Scoring Criteria

```
┌─────────────────────────────────────────────────────────────┐
│              SUBAGENT EVALUATION RUBRIC                      │
│                                                              │
│  Total: 100 points                                          │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 1. DEFINITION (25 points)                              │ │
│  │    - Name clarity (5)                                  │ │
│  │    - Description quality (10)                          │ │
│  │    - YAML correctness (5)                              │ │
│  │    - Model selection (5)                               │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 2. TOOL SCOPING (20 points)                            │ │
│  │    - Least privilege (10)                              │ │
│  │    - Appropriate tools (5)                             │ │
│  │    - MCP tools if needed (5)                           │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 3. SYSTEM PROMPT (25 points)                           │ │
│  │    - Role clarity (5)                                  │ │
│  │    - Task instructions (10)                            │ │
│  │    - Output format (5)                                 │ │
│  │    - Constraints (5)                                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 4. FUNCTIONALITY (20 points)                           │ │
│  │    - Task completion (10)                              │ │
│  │    - Output quality (5)                                │ │
│  │    - Error handling (5)                                │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 5. INTEGRATION (10 points)                             │ │
│  │    - Works with other components (5)                   │ │
│  │    - Documentation (5)                                 │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Detailed Scoring Guide

#### Category 1: Definition (25 points)

| Criterion | Points | Excellent (100%) | Good (75%) | Fair (50%) | Poor (25%) |
|-----------|--------|------------------|------------|------------|------------|
| **Name clarity** | 5 | Descriptive, kebab-case, unique | Mostly clear | Vague | Confusing |
| **Description** | 10 | Clear use case, actionable | Good but incomplete | Generic | Missing/wrong |
| **YAML** | 5 | All fields correct | Minor issues | Multiple issues | Invalid |
| **Model** | 5 | Optimal for task | Acceptable | Suboptimal | Wrong choice |

#### Category 2: Tool Scoping (20 points)

| Criterion | Points | Excellent | Good | Fair | Poor |
|-----------|--------|-----------|------|------|------|
| **Least privilege** | 10 | Only needed tools | Slightly over | Many extra | No restriction |
| **Appropriate** | 5 | Perfect match | Mostly correct | Some mismatch | Wrong tools |
| **MCP tools** | 5 | Correct MCP usage | Minor issues | Unnecessary MCP | Missing needed MCP |

#### Category 3: System Prompt (25 points)

| Criterion | Points | Excellent | Good | Fair | Poor |
|-----------|--------|-----------|------|------|------|
| **Role clarity** | 5 | Clear persona | Mostly clear | Vague | Missing |
| **Instructions** | 10 | Step-by-step, complete | Good coverage | Incomplete | Unclear |
| **Output format** | 5 | Well-defined | Defined | Vague | Missing |
| **Constraints** | 5 | Clear boundaries | Some constraints | Few | None |

#### Category 4: Functionality (20 points)

| Criterion | Points | Excellent | Good | Fair | Poor |
|-----------|--------|-----------|------|------|------|
| **Task completion** | 10 | Always completes | Usually | Sometimes | Rarely |
| **Output quality** | 5 | High quality | Good | Acceptable | Poor |
| **Error handling** | 5 | Graceful | Adequate | Minimal | None |

#### Category 5: Integration (10 points)

| Criterion | Points | Excellent | Good | Fair | Poor |
|-----------|--------|-----------|------|------|------|
| **Integration** | 5 | Seamless | Minor issues | Some friction | Doesn't work |
| **Documentation** | 5 | Complete | Good | Minimal | None |

### 2.3 Grade Scale

| Score | Grade | Status |
|-------|-------|--------|
| 90-100 | A | Production Ready |
| 80-89 | B | Approved with minor fixes |
| 70-79 | C | Needs improvement |
| 60-69 | D | Major revision required |
| <60 | F | Rejected - redo |

### 2.4 Sample Evaluation

```markdown
## Subagent Evaluation: weekly-report-generator

### Scores

| Category | Max | Score | Notes |
|----------|-----|-------|-------|
| Definition | 25 | 23 | Excellent description |
| Tool Scoping | 20 | 18 | Could be more restrictive |
| System Prompt | 25 | 22 | Good but output format could be clearer |
| Functionality | 20 | 18 | Works well, minor edge cases |
| Integration | 10 | 9 | Good docs |
| **TOTAL** | **100** | **90** | |

### Grade: A - Production Ready

### Feedback
- Strengths: Clear description, good tool selection
- Improvements: Consider restricting tools further, enhance error messages

### Approved by: [Reviewer Name]
### Date: 2025-01-15
```

---

## 3. CHEATSHEET 1 TRANG

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    CLAUDE CODE CHEATSHEET - MANGOADS                         ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  COMPONENTS                                                                  ║
║  ┌──────────────┬────────────────────┬────────────────────────────────────┐ ║
║  │ Component    │ Location           │ Purpose                            │ ║
║  ├──────────────┼────────────────────┼────────────────────────────────────┤ ║
║  │ Memory       │ CLAUDE.md          │ Project context (auto-load)        │ ║
║  │ Rules        │ .claude/rules/     │ Scoped instructions                │ ║
║  │ Subagent     │ .claude/agents/    │ Specialized AI (model-invoked)     │ ║
║  │ Skill        │ .claude/skills/    │ Reusable expertise (model-invoked) │ ║
║  │ Hook         │ .claude/settings.json │ Lifecycle control              │ ║
║  │ Command      │ .claude/commands/  │ User shortcuts (/command)          │ ║
║  │ Plugin       │ .claude-plugin/    │ Shareable bundle                   │ ║
║  │ MCP          │ .mcp.json          │ External tool integration          │ ║
║  └──────────────┴────────────────────┴────────────────────────────────────┘ ║
║                                                                              ║
║  DECISION QUICK GUIDE                                                        ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ Need...                          │ Use...                              │ ║
║  ├──────────────────────────────────┼─────────────────────────────────────┤ ║
║  │ Project context                  │ CLAUDE.md                           │ ║
║  │ Scoped rules by file type        │ .claude/rules/ with paths:          │ ║
║  │ Reusable expertise               │ Skill                               │ ║
║  │ Isolated AI for specific task    │ Subagent                            │ ║
║  │ Block/validate actions           │ Hook (PreToolUse)                   │ ║
║  │ Quality gate before finish       │ Hook (Stop)                         │ ║
║  │ User shortcut                    │ Slash Command                       │ ║
║  │ Browser automation               │ MCP (Playwright)                    │ ║
║  │ Debug/performance                │ MCP (DevTools)                      │ ║
║  │ Share with team                  │ Plugin                              │ ║
║  └──────────────────────────────────┴─────────────────────────────────────┘ ║
║                                                                              ║
║  PERMISSION MODES                                                            ║
║  ┌────────────────┬───────────────────────────────────────────────────────┐ ║
║  │ default        │ Ask permission for everything (safest)               │ ║
║  │ acceptEdits    │ Auto-accept edits, ask for dangerous                 │ ║
║  │ plan           │ Analyze only, no execution                           │ ║
║  │ bypassPermissions │ Skip all prompts (DANGEROUS)                      │ ║
║  └────────────────┴───────────────────────────────────────────────────────┘ ║
║  Toggle: Shift+Tab                                                          ║
║                                                                              ║
║  HOOK EVENTS                                                                 ║
║  ┌────────────────┬───────────────────────────────────────────────────────┐ ║
║  │ PreToolUse     │ Before tool executes (can block)                     │ ║
║  │ PostToolUse    │ After tool succeeds                                  │ ║
║  │ Stop           │ End of turn (quality gates)                          │ ║
║  │ SubagentStop   │ When subagent finishes                               │ ║
║  │ SessionStart   │ When session begins (setup)                          │ ║
║  └────────────────┴───────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  CLI QUICK REFERENCE                                                         ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ claude                    │ Start interactive REPL                    │ ║
║  │ claude -p "prompt"        │ Non-interactive (print mode)              │ ║
║  │ claude -c                 │ Continue last session                     │ ║
║  │ claude -r <id>            │ Resume specific session                   │ ║
║  │ /init                     │ Generate CLAUDE.md                        │ ║
║  │ /rewind (or Esc+Esc)      │ Rollback changes                          │ ║
║  │ /agents                   │ Manage subagents                          │ ║
║  │ /mcp                      │ Manage MCP servers                        │ ║
║  │ /hooks                    │ Review hooks                              │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  MCP TOOLS (PLAYWRIGHT)                                                      ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ playwright_navigate <url>     │ Go to URL                             │ ║
║  │ playwright_click <selector>   │ Click element                         │ ║
║  │ playwright_fill <sel> <val>   │ Fill input                            │ ║
║  │ playwright_screenshot         │ Capture screen                        │ ║
║  │ playwright_get_text <sel>     │ Get element text                      │ ║
║  │ playwright_wait <sel>         │ Wait for element                      │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  SUBAGENT YAML FIELDS                                                        ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ name: kebab-case-name                                                 │ ║
║  │ description: When Claude should invoke (IMPORTANT!)                   │ ║
║  │ model: sonnet|opus|haiku|inherit                                      │ ║
║  │ tools: Read, Write, Edit, Bash, Glob, Grep, mcp__*                    │ ║
║  │ permissionMode: default|acceptEdits|plan|bypassPermissions            │ ║
║  │ skills: skill1, skill2                                                │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  MANGOADS CONVENTIONS                                                        ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ UTM Campaign: {year}-{quarter}-{description}                          │ ║
║  │ Example: 2025-q1-product-launch                                       │ ║
║  │ Sources: google, facebook, instagram, linkedin, email                 │ ║
║  │ Mediums: cpc, social, email, display, affiliate                       │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  SECURITY REMINDERS                                                          ║
║  • Never put credentials in CLAUDE.md or .mcp.json                          ║
║  • Always review MCP output before production                               ║
║  • Use hooks for sensitive operation guardrails                             ║
║  • Log all Claude Code sessions                                             ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 4. TỔNG KẾT CHƯƠNG TRÌNH TRAINING

### 4.1 Requirement Checklist Hoàn Thành

```markdown
## FINAL REQUIREMENT CHECKLIST

### Nội dung bắt buộc
- [x] Bám sát tài liệu Claude Code
- [x] Không suy đoán tính năng chưa xác nhận
- [x] Cover đầy đủ Memory / Skill / Subagent / Hook / Command / Plugin / MCP
- [x] Có Decision Tree
- [x] ≥ 10 use cases (có MCP) - Đã có 12 use cases
- [x] Có ví dụ shadcn + RJSF
- [x] Có ví dụ Playwright / Browser MCP
- [x] Có template đầy đủ
- [x] Có hands-on labs (3 bài)
- [x] Có governance & security
- [x] Chia thành nhiều part, mỗi part < 2000 dòng

### Deliverables
- [x] Overview + phân track Marketing vs Dev
- [x] Decision Tree
- [x] Workflow chuẩn MangoAds (7 phases)
- [x] 12 Use Cases MangoAds (với MCP)
- [x] Ví dụ kỹ thuật (Next.js + Tailwind + shadcn/ui + RJSF)
- [x] Templates (6 templates)
- [x] Hands-on Labs (3 labs)
- [x] Governance & Safety
- [x] Rubric đánh giá Subagent
- [x] Cheatsheet 1 trang
```

### 4.2 Training Parts Summary

| Part | Nội dung | Trang |
|------|----------|-------|
| 1 | Overview, Memory, Rules, CLAUDE.md | part-01-overview-memory-rules.md |
| 2 | Subagents, Agent Skills | part-02-subagents-skills.md |
| 3 | Hooks, Slash Commands | part-03-hooks-commands.md |
| 4 | Plugins, MCP chi tiết | part-04-plugins-mcp.md |
| 5 | Tools, Permissions, CLI, Sessions | part-05-tools-permissions-cli.md |
| 6 | Decision Tree, Workflow MangoAds | part-06-decision-tree-workflow.md |
| 7 | 12 Use Cases thực tế | part-07-use-cases.md |
| 8 | Ví dụ kỹ thuật (Next.js, RJSF) | part-08-technical-examples.md |
| 9 | Templates, Hands-on Labs | part-09-templates-labs.md |
| 10 | Governance, Rubric, Cheatsheet | part-10-governance-cheatsheet.md |

### 4.3 Learning Path Recommendations

```
┌─────────────────────────────────────────────────────────────┐
│                    LEARNING PATHS                            │
│                                                              │
│  TRACK: MARKETING (2-3 days)                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Day 1: Part 1-2 (Foundation)                         │    │
│  │ Day 2: Part 6-7 (Workflow + Use Cases 1-4)          │    │
│  │ Day 3: Part 9 Lab 1-2 (Weekly Report, UTM)          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TRACK: DEVELOPMENT (4-5 days)                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Day 1: Part 1-2 (Foundation)                         │    │
│  │ Day 2: Part 3-4 (Hooks, MCP)                        │    │
│  │ Day 3: Part 5-6 (Tools, Workflow)                   │    │
│  │ Day 4: Part 7-8 (Use Cases, Technical Examples)     │    │
│  │ Day 5: Part 9 Lab 3 + Part 10 (MCP Test, Governance)│    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TRACK: FULL (1 week)                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ All 10 parts + All 3 labs                           │    │
│  │ Recommended for: Tech Leads, Senior Developers      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.4 Next Steps After Training

```markdown
## POST-TRAINING ACTIONS

### Immediate (Week 1)
- [ ] Setup CLAUDE.md cho project hiện tại
- [ ] Complete at least 1 hands-on lab
- [ ] Create first subagent/skill

### Short-term (Month 1)
- [ ] Implement 2-3 use cases from Part 7
- [ ] Setup MCP for team project
- [ ] Document learnings

### Long-term (Quarter 1)
- [ ] Create team plugin
- [ ] Share best practices
- [ ] Mentor others
```

---

## KẾT THÚC CHƯƠNG TRÌNH TRAINING

Chương trình training Claude Code của MangoAds đã hoàn thành với:

- **10 Parts** covering all Claude Code components
- **12 Use Cases** với MCP integration
- **6 Templates** chuẩn MangoAds
- **3 Hands-on Labs**
- **1 Cheatsheet** tổng hợp

**Feedback & Cải tiến:**
- Email: training@mangoads.com
- Slack: #claude-code-training

**Chúc các bạn thành công với Claude Code!**

---

*Tài liệu này được soạn bởi MangoAds Training Team*
*Cập nhật: Tháng 12/2025*
*Version: 1.0*
