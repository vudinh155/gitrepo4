# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 6: Decision Tree & Workflow Chuẩn MangoAds

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-5

---

## MỤC LỤC PART 6

1. [Decision Tree - Chọn Claude Code Component](#1-decision-tree---chọn-claude-code-component)
2. [Workflow Chuẩn MangoAds](#2-workflow-chuẩn-mangoads)
3. [Quy trình Brief → Deploy](#3-quy-trình-brief--deploy)
4. [Ví dụ Áp Dụng theo Track](#4-ví-dụ-áp-dụng-theo-track)

---

## 1. DECISION TREE - CHỌN CLAUDE CODE COMPONENT

### 1.1 Master Decision Tree

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CLAUDE CODE COMPONENT DECISION TREE                       │
│                                                                              │
│  START: Bạn cần làm gì?                                                     │
│         │                                                                    │
│         ▼                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q1: Cần thiết lập CONTEXT CHO PROJECT?                                │   │
│  │     (Coding standards, workflow, team rules)                          │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► MEMORY (CLAUDE.md / .claude/rules/)                    │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q2: Cần EXPERTISE có thể tái sử dụng?                                 │   │
│  │     (UTM generation, metric calculation, formatting)                  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► SKILL (.claude/skills/)                                │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q3: Cần AI CHUYÊN BIỆT với context riêng?                             │   │
│  │     (Code reviewer, content writer, QA tester)                        │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► SUBAGENT (.claude/agents/)                             │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q4: Cần KIỂM SOÁT LIFECYCLE?                                          │   │
│  │     (Block actions, validate outputs, run tests)                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► HOOK (.claude/settings.json)                           │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q5: Cần USER TRIGGER shortcut?                                        │   │
│  │     (Weekly report, deploy, generate docs)                            │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► SLASH COMMAND (.claude/commands/)                      │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q6: Cần KẾT NỐI EXTERNAL TOOLS?                                       │   │
│  │     (Browser, GitHub, Slack, databases)                               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► MCP (.mcp.json)                                        │
│         │                                                                    │
│         └─── NO                                                              │
│              │                                                               │
│              ▼                                                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ Q7: Cần ĐÓNG GÓI ĐỂ SHARE?                                            │   │
│  │     (Team toolkit, versioning, distribution)                          │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                    │
│         ├─── YES ──► PLUGIN (.claude-plugin/)                               │
│         │                                                                    │
│         └─── NO ──► Use built-in Claude Code features                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Quick Reference Table

| Nhu cầu | Component | File Location | Kích hoạt |
|---------|-----------|---------------|-----------|
| Project context | Memory | CLAUDE.md, .claude/rules/ | Auto-load |
| Reusable expertise | Skill | .claude/skills/ | Model-invoked |
| Specialized AI | Subagent | .claude/agents/ | Model-invoked |
| Lifecycle control | Hook | .claude/settings.json | Auto-trigger |
| User shortcut | Command | .claude/commands/ | User /command |
| External tools | MCP | .mcp.json | Tool call |
| Share/distribute | Plugin | .claude-plugin/ | Install |

### 1.3 Decision Examples

**Example 1: "Muốn Claude luôn follow coding standards"**
```
Q1: Cần context cho project? → YES
Answer: MEMORY (CLAUDE.md với coding standards)
```

**Example 2: "Muốn auto-generate UTM links"**
```
Q1: Project context? → NO
Q2: Reusable expertise? → YES
Answer: SKILL (utm-generator)
```

**Example 3: "Cần AI review code trước khi merge"**
```
Q1-Q2: NO
Q3: AI chuyên biệt? → YES
Answer: SUBAGENT (code-reviewer)
```

**Example 4: "Block deploy khi tests fail"**
```
Q1-Q3: NO
Q4: Lifecycle control? → YES
Answer: HOOK (PreToolUse for deploy commands)
```

**Example 5: "Shortcut để tạo weekly report"**
```
Q1-Q4: NO
Q5: User trigger shortcut? → YES
Answer: SLASH COMMAND (/weekly-report)
```

**Example 6: "Test landing page tự động"**
```
Q1-Q5: NO
Q6: External tools? → YES
Answer: MCP (Playwright MCP)
```

**Example 7: "Share marketing toolkit với team"**
```
Q1-Q6: Có thể là bất kỳ
Q7: Đóng gói để share? → YES
Answer: PLUGIN (mangoads-marketing-toolkit)
```

### 1.4 Combination Patterns

Thường cần kết hợp nhiều components:

```
┌─────────────────────────────────────────────────────────────┐
│            COMMON COMBINATION PATTERNS                       │
│                                                              │
│  Pattern 1: Complete Development Setup                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ CLAUDE.md (project context)                          │    │
│  │     +                                                │    │
│  │ Subagent (code-reviewer)                            │    │
│  │     +                                                │    │
│  │ Hook (lint/test on stop)                            │    │
│  │     +                                                │    │
│  │ MCP (GitHub integration)                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Pattern 2: Marketing Automation                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ CLAUDE.md (brand guidelines)                         │    │
│  │     +                                                │    │
│  │ Skill (utm-generator, metric-calculator)            │    │
│  │     +                                                │    │
│  │ Command (/weekly-report, /campaign-brief)           │    │
│  │     +                                                │    │
│  │ Subagent (content-writer)                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Pattern 3: QA Automation                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Rules (testing standards)                            │    │
│  │     +                                                │    │
│  │ Subagent (qa-tester)                                │    │
│  │     +                                                │    │
│  │ MCP (Playwright + DevTools)                         │    │
│  │     +                                                │    │
│  │ Hook (pre-deploy smoke test)                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. WORKFLOW CHUẨN MANGOADS

### 2.1 Tổng quan Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MANGOADS CLAUDE CODE WORKFLOW                             │
│                                                                              │
│  ┌─────┐   ┌─────────┐   ┌─────────┐   ┌────────┐   ┌────────┐   ┌───────┐ │
│  │BRIEF│──►│PHÂN TÍCH│──►│CHỌN COMP│──►│THIẾT KẾ│──►│  TEST  │──►│ROLLOUT│ │
│  └─────┘   └─────────┘   └─────────┘   └────────┘   └────────┘   └───────┘ │
│                                             │                         │      │
│                                             ▼                         ▼      │
│                                        ┌────────┐               ┌────────┐  │
│                                        │ITERATE │               │ AUDIT  │  │
│                                        └────────┘               └────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Phase 1: Brief

**Input:** Yêu cầu từ stakeholder

**Activities:**
- Thu thập requirements
- Clarify scope
- Identify constraints
- Document acceptance criteria

**Output:** Brief document

**Template Brief:**

```markdown
# Brief: [Tên Task]

## Stakeholder
- Người yêu cầu:
- Team:
- Priority: High/Medium/Low

## Requirements
1. [Requirement 1]
2. [Requirement 2]
3. ...

## Constraints
- Timeline:
- Resources:
- Technical limitations:

## Acceptance Criteria
- [ ] Criteria 1
- [ ] Criteria 2

## Out of Scope
- [Những gì KHÔNG làm]
```

### 2.3 Phase 2: Phân tích

**Input:** Brief document

**Activities:**
- Analyze current state
- Identify gaps
- Map to Claude Code capabilities
- Estimate complexity

**Output:** Analysis document

**Checklist Phân tích:**

```markdown
## Analysis Checklist

### Current State
- [ ] Review existing codebase/content
- [ ] Identify relevant files/resources
- [ ] Understand dependencies

### Gap Analysis
- [ ] What's missing?
- [ ] What needs to change?
- [ ] What can be reused?

### Complexity Assessment
- [ ] Simple (1-2 hours)
- [ ] Medium (1 day)
- [ ] Complex (multiple days)

### Risk Assessment
- [ ] Production impact?
- [ ] Data sensitivity?
- [ ] Rollback possible?
```

### 2.4 Phase 3: Chọn Components

**Input:** Analysis document

**Activities:**
- Apply Decision Tree
- Select appropriate components
- Plan combinations
- Document rationale

**Output:** Component selection document

**Template:**

```markdown
## Component Selection: [Task Name]

### Selected Components

| Component | Type | Rationale |
|-----------|------|-----------|
| CLAUDE.md | Memory | Need project context |
| code-reviewer | Subagent | Need specialized review |
| pre-deploy-check | Hook | Need quality gate |

### Why NOT other components
- Plugin: Not needed - single project scope
- MCP: No external tool requirement

### Dependencies
- Subagent depends on CLAUDE.md for context
- Hook depends on Subagent output
```

### 2.5 Phase 4: Thiết kế

**Input:** Component selection

**Activities:**
- Design component structure
- Write specifications
- Create test cases
- Review with team

**Output:** Design documents + specs

**Checklist Thiết kế:**

```markdown
## Design Checklist

### For Memory/Rules
- [ ] CLAUDE.md structure defined
- [ ] Rules scoped correctly
- [ ] No sensitive info included

### For Subagent
- [ ] Clear description
- [ ] Appropriate tools selected
- [ ] System prompt written
- [ ] Output format defined

### For Skill
- [ ] SKILL.md complete
- [ ] Examples provided
- [ ] Constraints documented

### For Hook
- [ ] Correct event selected
- [ ] Matcher appropriate
- [ ] Script tested standalone

### For Command
- [ ] YAML frontmatter complete
- [ ] Arguments documented
- [ ] Output format clear

### For MCP
- [ ] Server selected
- [ ] Config correct
- [ ] Security reviewed
```

### 2.6 Phase 5: Test

**Input:** Design documents

**Activities:**
- Implement components
- Unit test each component
- Integration test combinations
- User acceptance test

**Output:** Tested components

**Test Matrix:**

```markdown
## Test Matrix

### Unit Tests
| Component | Test Case | Status | Notes |
|-----------|-----------|--------|-------|
| utm-skill | Basic UTM generation | ✅ Pass | |
| utm-skill | Edge case: special chars | ✅ Pass | |
| code-reviewer | Simple file review | ✅ Pass | |
| code-reviewer | Multi-file review | ⚠️ Partial | Need more context |

### Integration Tests
| Flow | Status | Notes |
|------|--------|-------|
| Skill → Subagent | ✅ Pass | |
| Hook → Command | ✅ Pass | |
| MCP → Subagent | ⚠️ Partial | Timeout issues |

### UAT
| Scenario | Tester | Status |
|----------|--------|--------|
| Weekly report generation | @marketing | ✅ Approved |
| Code review workflow | @dev-lead | ✅ Approved |
```

### 2.7 Phase 6: Rollout

**Input:** Tested components

**Activities:**
- Deploy to staging
- Team training
- Gradual rollout
- Monitor usage

**Output:** Deployed solution

**Rollout Checklist:**

```markdown
## Rollout Checklist

### Pre-rollout
- [ ] All tests pass
- [ ] Documentation complete
- [ ] Team trained
- [ ] Rollback plan ready

### Deployment
- [ ] Deploy to .claude/ directories
- [ ] Update .mcp.json if needed
- [ ] Configure hooks
- [ ] Verify installation

### Post-rollout
- [ ] Announce to team
- [ ] Monitor for issues
- [ ] Collect feedback
- [ ] Iterate if needed
```

### 2.8 Phase 7: Audit

**Input:** Deployed solution

**Activities:**
- Regular review of usage
- Performance analysis
- Security audit
- Improvement identification

**Output:** Audit report + improvements

**Audit Schedule:**

```markdown
## Audit Schedule

| Frequency | Focus | Owner |
|-----------|-------|-------|
| Weekly | Usage metrics | Team lead |
| Monthly | Performance review | Tech lead |
| Quarterly | Security audit | Security team |
| Yearly | Full review | Management |
```

---

## 3. QUY TRÌNH BRIEF → DEPLOY

### 3.1 Detailed Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DETAILED WORKFLOW FLOW                                │
│                                                                              │
│  DAY 1: BRIEF + ANALYSIS                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Morning:                                                             │    │
│  │ - Receive requirements from stakeholder                             │    │
│  │ - Create brief document                                              │    │
│  │ - Clarify questions                                                  │    │
│  │                                                                      │    │
│  │ Afternoon:                                                           │    │
│  │ - Analyze current state                                              │    │
│  │ - Identify gaps                                                      │    │
│  │ - Apply Decision Tree                                               │    │
│  │ - Document component selection                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 2: DESIGN                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Morning:                                                             │    │
│  │ - Design component structure                                         │    │
│  │ - Write YAML specs                                                   │    │
│  │ - Draft system prompts                                               │    │
│  │                                                                      │    │
│  │ Afternoon:                                                           │    │
│  │ - Review with team                                                   │    │
│  │ - Iterate based on feedback                                         │    │
│  │ - Finalize designs                                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 3: IMPLEMENT + TEST                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Morning:                                                             │    │
│  │ - Implement components                                               │    │
│  │ - Write test cases                                                   │    │
│  │                                                                      │    │
│  │ Afternoon:                                                           │    │
│  │ - Run unit tests                                                     │    │
│  │ - Run integration tests                                              │    │
│  │ - Fix issues                                                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 4: UAT + ROLLOUT                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Morning:                                                             │    │
│  │ - User acceptance testing                                            │    │
│  │ - Collect feedback                                                   │    │
│  │                                                                      │    │
│  │ Afternoon:                                                           │    │
│  │ - Deploy to staging                                                  │    │
│  │ - Team training                                                      │    │
│  │ - Deploy to production                                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ONGOING: MONITOR + AUDIT                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ - Monitor usage                                                      │    │
│  │ - Collect metrics                                                    │    │
│  │ - Regular audits                                                     │    │
│  │ - Continuous improvement                                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Checklist Master

```markdown
# MANGOADS CLAUDE CODE IMPLEMENTATION CHECKLIST

## Phase 1: Brief
- [ ] Requirements documented
- [ ] Scope defined
- [ ] Constraints identified
- [ ] Acceptance criteria listed

## Phase 2: Analysis
- [ ] Current state reviewed
- [ ] Gaps identified
- [ ] Complexity assessed
- [ ] Risks documented

## Phase 3: Component Selection
- [ ] Decision Tree applied
- [ ] Components selected
- [ ] Rationale documented
- [ ] Dependencies mapped

## Phase 4: Design
- [ ] Structures defined
- [ ] Specs written
- [ ] System prompts drafted
- [ ] Team review completed

## Phase 5: Implementation
- [ ] Components created
- [ ] Files in correct locations
- [ ] Configuration complete

## Phase 6: Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] UAT approved
- [ ] Edge cases covered

## Phase 7: Rollout
- [ ] Staging deployed
- [ ] Team trained
- [ ] Production deployed
- [ ] Documentation updated

## Phase 8: Audit
- [ ] Monitoring setup
- [ ] Metrics collected
- [ ] Regular reviews scheduled
```

---

## 4. VÍ DỤ ÁP DỤNG THEO TRACK

### 4.1 Track Marketing: Campaign Report Automation

**Brief:**
> Cần tự động hóa weekly report cho marketing campaigns. Report phải có metrics, comparison với tuần trước, và recommendations.

**Phân tích:**
- Current: Manual report creation in Google Sheets
- Gap: Time-consuming, inconsistent format
- Claude Code fit: High - structured task, repeatable

**Chọn Components:**

| Component | Rationale |
|-----------|-----------|
| CLAUDE.md | Project context, metric definitions |
| metric-calculator Skill | Reusable calculation logic |
| weekly-report-generator Subagent | Specialized report generation |
| /weekly-report Command | User trigger shortcut |

**Thiết kế:**

```
.claude/
├── CLAUDE.md                    # Campaign context
├── skills/
│   └── metric-calculator/
│       └── SKILL.md
├── agents/
│   └── weekly-report-generator.md
└── commands/
    └── weekly-report.md
```

**Workflow sử dụng:**
```
User: /weekly-report q1-campaign

Claude:
1. Load CLAUDE.md context
2. Invoke weekly-report-generator subagent
3. Subagent uses metric-calculator skill
4. Generate formatted report
5. Return to user
```

### 4.2 Track Development: Code Review Automation

**Brief:**
> Cần automated code review trước khi merge PR. Review phải check security, performance, và coding standards.

**Phân tích:**
- Current: Manual review, sometimes missed
- Gap: Inconsistent, time-consuming
- Claude Code fit: High - pattern matching, rule-based

**Chọn Components:**

| Component | Rationale |
|-----------|-----------|
| .claude/rules/coding-standards.md | Rules for review |
| security-auditor Subagent | Specialized security review |
| code-reviewer Subagent | General code review |
| PreToolUse Hook | Block merge without review |
| MCP GitHub | Integrate with PRs |

**Thiết kế:**

```
.claude/
├── rules/
│   ├── coding-standards.md
│   └── security-rules.md
├── agents/
│   ├── security-auditor.md
│   └── code-reviewer.md
├── settings.json              # Hooks
└── .mcp.json                  # GitHub MCP
```

**Workflow:**
```
Developer: "Review PR #123"

Claude:
1. MCP GitHub: Fetch PR diff
2. Apply coding-standards rules
3. Invoke security-auditor subagent
4. Invoke code-reviewer subagent
5. Aggregate results
6. Report findings
7. Hook: Block merge if critical issues
```

### 4.3 Track QA: Landing Page Testing

**Brief:**
> Cần automated testing cho landing pages trước deploy. Test form submission, responsive, và performance.

**Phân tích:**
- Current: Manual testing, often incomplete
- Gap: Time-consuming, inconsistent coverage
- Claude Code fit: High - Playwright MCP ideal

**Chọn Components:**

| Component | Rationale |
|-----------|-----------|
| .claude/rules/testing-standards.md | Test criteria |
| qa-tester Subagent | Specialized testing |
| Playwright MCP | Browser automation |
| DevTools MCP | Performance analysis |
| Stop Hook | Run tests before finish |

**Thiết kế:**

```
.claude/
├── rules/
│   └── testing-standards.md
├── agents/
│   └── qa-tester.md
├── settings.json              # Hooks
└── .mcp.json                  # Playwright + DevTools
```

**Workflow:**
```
Developer: "Test landing page https://staging.mangoads.com/q1-campaign"

Claude:
1. Invoke qa-tester subagent
2. MCP Playwright: Navigate to page
3. Test form submission
4. Test responsive layouts
5. MCP DevTools: Check performance
6. Generate test report
7. Hook: Flag issues if critical
```

---

## TÓM TẮT PART 6

### Đã cover trong Part 6:
- [x] Master Decision Tree cho Claude Code components
- [x] Quick reference table
- [x] Decision examples
- [x] Combination patterns
- [x] MangoAds Workflow chi tiết 7 phases
- [x] Brief → Deploy quy trình
- [x] Master checklist
- [x] Ví dụ theo track (Marketing, Development, QA)

### Part 7 sẽ cover:
- [ ] 10+ Use Cases thực tế MangoAds
- [ ] Use cases với MCP (Playwright, DevTools)
- [ ] Form regression testing
- [ ] Visual regression testing

---

**Tiếp theo:** [Part 7: Use Cases Thực Tế MangoAds](./part-07-use-cases.md)
