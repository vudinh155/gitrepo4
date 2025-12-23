# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 7: Use Cases Thực Tế MangoAds

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-6

---

## MỤC LỤC PART 7

1. [Digital Marketing Use Cases](#1-digital-marketing-use-cases)
2. [Web Development Use Cases](#2-web-development-use-cases)
3. [QA/Testing Use Cases với MCP](#3-qatesting-use-cases-với-mcp)
4. [Cross-functional Use Cases](#4-cross-functional-use-cases)

---

## TỔNG QUAN 12 USE CASES

| # | Use Case | Track | MCP | Components |
|---|----------|-------|-----|------------|
| 1 | Weekly Report Generator | Marketing | ❌ | Subagent, Skill, Command |
| 2 | UTM Campaign Builder | Marketing | ❌ | Skill, Command |
| 3 | Content Brief Generator | Marketing | ❌ | Subagent, Rules |
| 4 | SEO Audit Assistant | Marketing | ✅ DevTools | Subagent, MCP |
| 5 | Landing Page Form Test | QA | ✅ Playwright | Subagent, MCP |
| 6 | Visual Regression Test | QA | ✅ Playwright | Subagent, MCP, Hook |
| 7 | Browser Smoke Test | QA | ✅ Playwright | Subagent, MCP, Hook |
| 8 | Code Review Automation | Dev | ❌ | Subagent, Hook |
| 9 | Component Generator | Dev | ❌ | Skill, Command, Rules |
| 10 | API Documentation | Dev | ❌ | Subagent, Skill |
| 11 | Performance Audit | Cross | ✅ DevTools | Subagent, MCP |
| 12 | Deploy Pipeline | Cross | ✅ Playwright | Hook, MCP, Command |

---

## 1. DIGITAL MARKETING USE CASES

### Use Case 1: Weekly Report Generator

**Mô tả:** Tự động sinh weekly marketing performance report từ campaign data.

**Problem Statement:**
- Manual report mất 3-4 giờ mỗi tuần
- Format không consistent
- Dễ miss metrics quan trọng

**Solution Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│            WEEKLY REPORT GENERATOR                           │
│                                                              │
│  Input: Campaign name + date range                          │
│                                                              │
│  ┌─────────────────┐                                        │
│  │ /weekly-report  │ User trigger                           │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  CLAUDE.md      │ Project context                        │
│  │  (metrics def)  │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ report-generator│ Subagent                               │
│  │    subagent     │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │metric-calculator│ Skill                                  │
│  │     skill       │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  Output: Formatted markdown report                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components Required:**

```
.claude/
├── CLAUDE.md                     # Metric definitions, KPIs
├── skills/
│   └── metric-calculator/
│       └── SKILL.md
├── agents/
│   └── report-generator.md
└── commands/
    └── weekly-report.md
```

**Usage:**
```
/weekly-report q1-brand-awareness 2025-01-01 2025-01-07
```

**Output Example:**
```markdown
# Weekly Report: Q1 Brand Awareness Campaign
**Period:** Jan 1-7, 2025

## Executive Summary
- Impressions tăng 15% so với tuần trước
- CTR đạt 2.3%, cao hơn benchmark (2.0%)
- ROAS đạt 4.2x, vượt target (4.0x)

## Performance Metrics
| Metric | This Week | Last Week | Change |
|--------|-----------|-----------|--------|
| Impressions | 125,000 | 108,695 | +15% |
| Clicks | 2,875 | 2,174 | +32% |
| CTR | 2.3% | 2.0% | +15% |
...
```

---

### Use Case 2: UTM Campaign Builder

**Mô tả:** Generate UTM tracking links theo chuẩn MangoAds.

**Problem Statement:**
- UTM không consistent giữa các campaigns
- Typos trong parameter names
- Tracking bị miss trong GA4

**Solution Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│              UTM CAMPAIGN BUILDER                            │
│                                                              │
│  ┌─────────────────┐                                        │
│  │   /utm command  │ User input                             │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ utm-generator   │ Skill (model-invoked)                  │
│  │     skill       │                                        │
│  │                 │ - Validate inputs                      │
│  │                 │ - Apply conventions                    │
│  │                 │ - Generate link                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  Output: Validated UTM link                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Skill Implementation:**

```markdown
---
name: utm-generator
description: Generates UTM tracking links following MangoAds conventions
---

# UTM Generator

## Validation Rules
- All parameters lowercase
- No spaces (use hyphens)
- Campaign format: {year}-{quarter}-{name}
- Only allowed sources/mediums

## Allowed Values
### utm_source
google, facebook, instagram, linkedin, twitter, email, direct

### utm_medium
cpc, ppc, social, email, display, affiliate
```

**Usage Examples:**
```
/utm q1-product-launch google cpc
→ https://mangoads.com?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-product-launch

/utm q1-launch facebook social header-cta
→ https://mangoads.com?utm_source=facebook&utm_medium=social&utm_campaign=2025-q1-launch&utm_content=header-cta
```

---

### Use Case 3: Content Brief Generator

**Mô tả:** Generate content briefs từ campaign objectives.

**Problem Statement:**
- Content briefs thiếu structure
- Missing SEO requirements
- Inconsistent brand voice guidelines

**Solution:**

```
.claude/
├── rules/
│   ├── brand-voice.md          # Brand guidelines
│   └── seo-requirements.md     # SEO checklist
├── agents/
│   └── content-strategist.md   # Content brief generator
└── commands/
    └── content-brief.md
```

**Subagent Definition:**

```yaml
---
name: content-strategist
description: Generates comprehensive content briefs including SEO requirements and brand voice guidelines
model: opus
tools: Read, Glob, Grep, WebSearch
---

# Content Strategist

You are a senior content strategist at MangoAds.

## Tasks
1. Analyze topic/keyword
2. Research competitors
3. Define content structure
4. Include SEO requirements
5. Apply brand voice guidelines

## Output Format
- Target keyword
- Search intent
- Content outline
- Word count recommendation
- CTA suggestions
- SEO checklist
```

---

### Use Case 4: SEO Audit Assistant (MCP)

**Mô tả:** Audit SEO technical issues sử dụng Browser DevTools MCP.

**MCP Required:** ✅ Chrome DevTools MCP

**Problem Statement:**
- Manual SEO audit mất nhiều thời gian
- Miss technical issues
- Không có systematic approach

**Solution Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│               SEO AUDIT ASSISTANT                            │
│                                                              │
│  ┌─────────────────┐                                        │
│  │   seo-auditor   │ Subagent                               │
│  │                 │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  DevTools MCP   │                                        │
│  │                 │ - Check meta tags                      │
│  │                 │ - Analyze load time                    │
│  │                 │ - Check console errors                 │
│  │                 │ - Analyze network requests             │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  Output: SEO audit report                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Subagent với MCP tools:**

```yaml
---
name: seo-auditor
description: Performs technical SEO audits using browser DevTools
model: sonnet
tools: Read, Glob, Grep, mcp__devtools
---

# SEO Auditor

## Audit Checklist
1. Meta tags (title, description, robots)
2. Heading structure (H1, H2, H3)
3. Image alt texts
4. Page load time
5. Console errors
6. Mobile responsiveness
7. Core Web Vitals
```

---

## 2. WEB DEVELOPMENT USE CASES

### Use Case 5: Landing Page Form Test (MCP)

**Mô tả:** Automated form testing trên landing pages với Playwright MCP.

**MCP Required:** ✅ Playwright MCP

**Problem Statement:**
- Forms không được test kỹ trước launch
- Edge cases bị miss
- Validation errors không được catch

**Solution Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│           LANDING PAGE FORM TEST                             │
│                                                              │
│  ┌─────────────────┐                                        │
│  │   qa-tester     │ Subagent                               │
│  │   subagent      │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              PLAYWRIGHT MCP                          │    │
│  │                                                      │    │
│  │  1. playwright_navigate → Go to landing page        │    │
│  │  2. playwright_fill → Fill form fields              │    │
│  │  3. playwright_click → Submit form                  │    │
│  │  4. playwright_wait → Wait for response             │    │
│  │  5. playwright_screenshot → Capture result          │    │
│  │  6. playwright_get_text → Verify success message    │    │
│  │                                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│           │                                                  │
│           ▼                                                  │
│  Output: Test report with screenshots                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Test Cases:**

```markdown
## Form Test Cases

### Happy Path
1. Fill all required fields with valid data
2. Submit form
3. Verify success message
4. Verify data received by backend

### Validation Tests
1. Submit empty form → Verify error messages
2. Invalid email → Verify email error
3. Phone number format → Verify phone error
4. Required field empty → Verify required error

### Edge Cases
1. Special characters in name
2. Very long input values
3. Script injection attempt
4. SQL injection attempt
```

**Subagent Definition:**

```yaml
---
name: form-tester
description: Tests landing page forms using Playwright browser automation
model: sonnet
tools: Read, mcp__playwright
permissionMode: default
---

# Form Tester

You test web forms thoroughly.

## Test Flow
1. Navigate to page
2. Identify form fields
3. Run happy path test
4. Run validation tests
5. Run edge case tests
6. Generate report with screenshots

## Output
- Test case name
- Status (Pass/Fail)
- Screenshot
- Error details if failed
```

**Usage:**
```
User: "Test contact form trên https://staging.mangoads.com/contact"

Claude:
1. Navigate to URL
2. Identify form: name, email, phone, message fields
3. Test 1: Valid submission → Pass
4. Test 2: Empty email → Fail (no error shown)
5. ...
6. Report: 8/10 tests pass, 2 issues found
```

---

### Use Case 6: Visual Regression Test (MCP)

**Mô tả:** So sánh screenshots để detect unintended visual changes.

**MCP Required:** ✅ Playwright MCP

**Problem Statement:**
- CSS changes break other parts of site
- Visual bugs slip through code review
- No systematic visual verification

**Solution Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│            VISUAL REGRESSION TEST                            │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                 WORKFLOW                              │   │
│  │                                                       │   │
│  │  1. Load baseline images (previous approved state)   │   │
│  │                                                       │   │
│  │  2. Capture current screenshots via Playwright       │   │
│  │     - Desktop viewport                               │   │
│  │     - Tablet viewport                                │   │
│  │     - Mobile viewport                                │   │
│  │                                                       │   │
│  │  3. Compare images pixel-by-pixel                    │   │
│  │                                                       │   │
│  │  4. Generate diff report                             │   │
│  │     - Highlight changed areas                        │   │
│  │     - Calculate diff percentage                      │   │
│  │                                                       │   │
│  │  5. Alert if diff > threshold                        │   │
│  │                                                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Hook Integration:**

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Before finishing, run visual regression test on the modified pages. Compare with baseline and report any significant differences (>5% pixel change)."
          }
        ]
      }
    ]
  }
}
```

**Viewports to Test:**

| Device | Viewport | Priority |
|--------|----------|----------|
| Desktop | 1920x1080 | High |
| Laptop | 1366x768 | High |
| Tablet | 768x1024 | Medium |
| Mobile | 375x667 | High |

---

### Use Case 7: Browser Smoke Test (MCP)

**Mô tả:** Pre-deploy smoke test tự động với Playwright.

**MCP Required:** ✅ Playwright MCP

**Problem Statement:**
- Deploy breaks critical functionality
- No quick verification before release
- Manual smoke test inconsistent

**Smoke Test Checklist:**

```markdown
## Smoke Test Checklist

### Critical Paths
1. [ ] Homepage loads < 3s
2. [ ] Navigation works
3. [ ] Contact form accessible
4. [ ] Footer links work
5. [ ] No JavaScript errors

### Responsive
1. [ ] Desktop layout OK
2. [ ] Mobile layout OK
3. [ ] Touch targets accessible

### Tracking
1. [ ] GA4 loads
2. [ ] Facebook Pixel fires
3. [ ] GTM container loads
```

**Hook for Pre-deploy:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/check-deploy-command.sh"
          }
        ]
      }
    ]
  }
}
```

**check-deploy-command.sh:**
```bash
#!/bin/bash
if [[ "$1" == *"deploy"* ]] || [[ "$1" == *"release"* ]]; then
    echo "Deploy command detected. Running smoke test first..."
    # Trigger smoke test via Claude
    exit 0  # Allow after smoke test
fi
exit 0
```

---

### Use Case 8: Code Review Automation

**Mô tả:** Automated code review trước merge.

**Problem Statement:**
- Inconsistent review quality
- Security issues missed
- Style violations slip through

**Solution:**

```yaml
---
name: code-reviewer
description: Reviews code changes for bugs, security issues, performance problems, and style violations
model: opus
tools: Read, Glob, Grep
permissionMode: plan
---

# Code Reviewer

## Review Checklist
1. Security vulnerabilities
2. Performance issues
3. Code style violations
4. Test coverage
5. Documentation

## Severity Levels
- Critical: Security vulnerabilities, data loss risk
- High: Bugs, performance degradation
- Medium: Style violations, minor issues
- Low: Suggestions, improvements
```

---

### Use Case 9: Component Generator

**Mô tả:** Generate React components theo chuẩn MangoAds.

**Problem Statement:**
- Inconsistent component structure
- Missing TypeScript types
- Boilerplate repetitive

**Solution:**

```markdown
# .claude/commands/component.md

---
description: Generate React component with TypeScript, tests, and stories
allowed-tools: Read, Write, Edit, Glob
argument-hint: "ComponentName [--with-story] [--with-test]"
---

# Component Generator

Generate a new React component following MangoAds standards.

## Arguments
- ComponentName: PascalCase component name
- --with-story: Include Storybook story
- --with-test: Include test file

## Files Generated
- src/components/{name}/{name}.tsx
- src/components/{name}/{name}.types.ts
- src/components/{name}/{name}.test.tsx (optional)
- src/components/{name}/{name}.stories.tsx (optional)
- src/components/{name}/index.ts

## Template
Follow MangoAds component conventions:
- Functional component with forwardRef
- TypeScript strict mode
- Named exports
- Props interface in separate file
```

---

### Use Case 10: API Documentation

**Mô tả:** Generate API documentation từ code.

**Problem Statement:**
- Docs outdated
- Manual documentation tedious
- Missing edge cases

**Subagent:**

```yaml
---
name: api-documenter
description: Generates API documentation from code comments and types
model: sonnet
tools: Read, Glob, Grep, Write
---

# API Documenter

## Tasks
1. Scan API routes
2. Extract TypeScript types
3. Parse JSDoc comments
4. Generate OpenAPI spec
5. Create markdown docs

## Output Format
- OpenAPI 3.0 specification
- Markdown documentation
- Request/response examples
```

---

## 3. QA/TESTING USE CASES VỚI MCP

### Use Case 11: Performance Audit (MCP)

**Mô tả:** Comprehensive performance audit với DevTools MCP.

**MCP Required:** ✅ Chrome DevTools MCP

**Audit Areas:**

```markdown
## Performance Audit

### Core Web Vitals
- LCP (Largest Contentful Paint) < 2.5s
- FID (First Input Delay) < 100ms
- CLS (Cumulative Layout Shift) < 0.1

### Loading Performance
- Time to First Byte (TTFB)
- First Contentful Paint (FCP)
- Speed Index
- Time to Interactive (TTI)

### Resource Analysis
- JavaScript bundle size
- CSS bundle size
- Image optimization
- Third-party scripts impact

### Network Analysis
- Number of requests
- Total transfer size
- Caching effectiveness
```

**Subagent Definition:**

```yaml
---
name: performance-auditor
description: Performs comprehensive performance audits using Chrome DevTools
model: sonnet
tools: Read, mcp__devtools
---

# Performance Auditor

## Workflow
1. Connect to page via DevTools
2. Start performance recording
3. Load page
4. Collect metrics
5. Analyze bottlenecks
6. Generate recommendations

## Output
- Core Web Vitals scores
- Bottleneck analysis
- Prioritized recommendations
- Comparison with benchmarks
```

---

## 4. CROSS-FUNCTIONAL USE CASES

### Use Case 12: Deploy Pipeline (MCP)

**Mô tả:** Automated deploy với pre-deploy checks.

**MCP Required:** ✅ Playwright MCP (for smoke test)

**Pipeline:**

```
┌─────────────────────────────────────────────────────────────┐
│                  DEPLOY PIPELINE                             │
│                                                              │
│  ┌─────────────────┐                                        │
│  │  /deploy cmd    │ User trigger                           │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Pre-deploy     │ Hook                                   │
│  │  checks         │                                        │
│  │  - Lint         │                                        │
│  │  - Tests        │                                        │
│  │  - Build        │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Deploy to      │ Bash                                   │
│  │  staging        │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Smoke test     │ Playwright MCP                         │
│  │  staging        │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│       Pass? ──NO──► Rollback + Alert                        │
│           │                                                  │
│          YES                                                 │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Deploy to      │ Bash                                   │
│  │  production     │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Smoke test     │ Playwright MCP                         │
│  │  production     │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  Output: Deploy report                                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Command Definition:**

```markdown
# .claude/commands/deploy.md

---
description: Deploy to staging/production with automated checks and smoke tests
allowed-tools: Read, Bash, mcp__playwright
argument-hint: "environment [staging|production]"
---

# Deploy Command

## Pre-deploy Checklist
- [ ] All tests pass
- [ ] Lint clean
- [ ] Build successful
- [ ] Git status clean

## Workflow
1. Run pre-deploy checks
2. Deploy to target environment
3. Run smoke tests
4. Report status

## Rollback
If smoke test fails:
1. Revert deployment
2. Alert team
3. Generate incident report
```

---

## TÓM TẮT USE CASES

### Use Cases theo Component

| Component | Use Cases |
|-----------|-----------|
| Subagent | 1, 3, 4, 5, 6, 7, 8, 10, 11 |
| Skill | 1, 2, 9, 10 |
| Command | 1, 2, 3, 9, 12 |
| Hook | 6, 7, 8, 12 |
| MCP | 4, 5, 6, 7, 11, 12 |
| Rules | 3, 8, 9 |

### Use Cases theo Track

| Track | Use Cases |
|-------|-----------|
| Marketing | 1, 2, 3, 4 |
| Development | 5, 8, 9, 10 |
| QA | 5, 6, 7, 11 |
| Cross-functional | 11, 12 |

### Use Cases với MCP

| MCP Server | Use Cases |
|------------|-----------|
| Playwright | 5, 6, 7, 12 |
| DevTools | 4, 11 |

---

## TÓM TẮT PART 7

### Đã cover trong Part 7:
- [x] 12 Use Cases thực tế MangoAds
- [x] 4 Digital Marketing use cases
- [x] 4 Web Development use cases
- [x] 4 QA/Testing use cases với MCP
- [x] Playwright MCP: Form test, Visual regression, Smoke test, Deploy
- [x] DevTools MCP: SEO audit, Performance audit
- [x] Cross-functional: Deploy pipeline

### Part 8 sẽ cover:
- [ ] Ví dụ kỹ thuật chi tiết
- [ ] Next.js + Tailwind + shadcn/ui
- [ ] React JSON Schema Form (Campaign Brief)
- [ ] Subagent + MCP test page sinh ra

---

**Tiếp theo:** [Part 8: Ví Dụ Kỹ Thuật Chi Tiết](./part-08-technical-examples.md)
