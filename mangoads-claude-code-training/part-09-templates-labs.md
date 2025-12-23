# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 9: Templates & Hands-on Labs

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-8

---

## MỤC LỤC PART 9

1. [Templates Chuẩn MangoAds](#1-templates-chuẩn-mangoads)
2. [Hands-on Lab 1: Weekly Report Subagent](#2-hands-on-lab-1-weekly-report-subagent)
3. [Hands-on Lab 2: UTM Generator Skill](#3-hands-on-lab-2-utm-generator-skill)
4. [Hands-on Lab 3: MCP Playwright Web Test](#4-hands-on-lab-3-mcp-playwright-web-test)

---

## 1. TEMPLATES CHUẨN MANGOADS

### 1.1 Template: Subagent Specification

```markdown
# SUBAGENT TEMPLATE

---
name: [kebab-case-name]
description: [Clear description - Claude dựa vào đây để quyết định invoke]
model: [sonnet|opus|haiku|inherit]
tools: [Comma-separated: Read, Write, Edit, Bash, Glob, Grep, WebFetch, WebSearch, mcp__*]
permissionMode: [default|acceptEdits|plan|bypassPermissions]
skills: [Optional: skill-1, skill-2]
---

# [Subagent Name] - MangoAds

## Role
[Mô tả role của subagent này]

## Capabilities
1. [Capability 1]
2. [Capability 2]
3. [Capability 3]

## Tasks
### Task 1: [Task Name]
- Step 1
- Step 2
- Step 3

### Task 2: [Task Name]
- Step 1
- Step 2

## Rules
- [Rule 1]
- [Rule 2]
- [Rule 3]

## Constraints
- [Constraint 1]
- [Constraint 2]

## Output Format
```
[Define expected output format]
```

## Examples

### Example 1: [Scenario]
Input: [Sample input]
Expected Output: [Sample output]

### Example 2: [Scenario]
Input: [Sample input]
Expected Output: [Sample output]
```

---

### 1.2 Template: Skill (SKILL.md)

```markdown
# SKILL TEMPLATE

---
name: [skill-name]
description: [Clear description - Claude dựa vào đây để tự động sử dụng skill]
---

# [Skill Name]

## Purpose
[Why this skill exists and when it's useful]

## When to Use
- [Scenario 1]
- [Scenario 2]
- [Scenario 3]

## Instructions

### Step 1: [Step Name]
[Detailed instructions]

### Step 2: [Step Name]
[Detailed instructions]

### Step 3: [Step Name]
[Detailed instructions]

## Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| [param1] | string | Yes | [Description] |
| [param2] | number | No | [Description] |

## Validation Rules
- [Rule 1]
- [Rule 2]

## Examples

### Example 1: Basic Usage
**Input:**
[Input data]

**Process:**
[Step-by-step process]

**Output:**
[Expected output]

### Example 2: Advanced Usage
**Input:**
[Input data]

**Output:**
[Expected output]

## Constraints
- [Constraint 1]
- [Constraint 2]

## Anti-patterns
❌ [What NOT to do 1]
❌ [What NOT to do 2]

## Related Skills
- [Related skill 1]
- [Related skill 2]
```

---

### 1.3 Template: Hook Configuration

```json
{
  "_comment": "HOOK TEMPLATE - .claude/settings.json",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "[Tool1|Tool2]",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/scripts/[script-name].sh",
            "timeout": 5000
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "[ToolName]",
        "hooks": [
          {
            "type": "command",
            "command": "[command]"
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
            "command": "[lint/test command]"
          },
          {
            "type": "prompt",
            "prompt": "[Quality check prompt]"
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
            "prompt": "[Validation prompt for subagent output]"
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
            "command": "$CLAUDE_PROJECT_DIR/scripts/setup.sh"
          }
        ]
      }
    ]
  }
}
```

**Hook Script Template:**

```bash
#!/bin/bash
# HOOK SCRIPT TEMPLATE
# scripts/hooks/[hook-name].sh

set -e

# Environment variables available:
# $CLAUDE_PROJECT_DIR - Project root
# $CLAUDE_ENV_FILE - Env file for persistence

# Get input from argument or stdin
INPUT="${1:-}"

# Validation logic
validate() {
    # Add validation logic here
    echo "Validating: $INPUT"

    # Return 0 for pass, non-zero for fail
    return 0
}

# Main execution
main() {
    echo "Running hook: [hook-name]"

    if validate; then
        echo "✅ Validation passed"
        exit 0
    else
        echo "❌ Validation failed: [reason]"
        exit 1
    fi
}

main "$@"
```

---

### 1.4 Template: Slash Command

```markdown
# SLASH COMMAND TEMPLATE

---
description: [Brief description shown in /help]
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep]
argument-hint: "[arg1] [arg2] [--option]"
model: [sonnet|opus|haiku]
---

# [Command Name]

## Purpose
[What this command does]

## Arguments
- `$ARGUMENTS[0]`: [First argument description]
- `$ARGUMENTS[1]`: [Second argument description]
- `--option`: [Optional flag description]

## Process
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Output Format
```
[Expected output format]
```

## Examples

### Example 1
```
/[command] arg1 arg2
```
Output: [Expected output]

### Example 2
```
/[command] arg1 --option
```
Output: [Expected output]

## Error Handling
- If [condition]: [action]
- If [condition]: [action]
```

---

### 1.5 Template: Plugin Checklist

```markdown
# PLUGIN CHECKLIST

## Plugin Metadata
- [ ] .claude-plugin/plugin.json exists
- [ ] name: unique, kebab-case
- [ ] version: semantic versioning (x.y.z)
- [ ] description: clear, concise
- [ ] author: team/individual name

## Structure
- [ ] Commands in /commands/
- [ ] Agents in /agents/
- [ ] Skills in /skills/
- [ ] Hooks in /hooks/hooks.json
- [ ] MCP config in /.mcp.json (if needed)
- [ ] README.md with documentation

## Quality
- [ ] All components tested individually
- [ ] Integration tests pass
- [ ] No hardcoded project-specific values
- [ ] Documentation complete

## Security
- [ ] No credentials in code
- [ ] MCP permissions reviewed
- [ ] Hook scripts audited
- [ ] Tool scoping appropriate

## Distribution
- [ ] Version tagged
- [ ] Changelog updated
- [ ] Team notified
- [ ] Installation instructions clear
```

---

### 1.6 Template: MCP Usage & Security Checklist

```markdown
# MCP USAGE & SECURITY CHECKLIST

## Before Adding MCP Server

### Research
- [ ] Reviewed official documentation
- [ ] Checked maintainer reputation
- [ ] Verified no known vulnerabilities
- [ ] Understood data access scope

### Approval
- [ ] Team lead approved
- [ ] Security team notified (if sensitive)
- [ ] Use case documented

## Configuration

### .mcp.json Setup
- [ ] Server command correct
- [ ] Arguments validated
- [ ] Environment variables use ${VAR} syntax
- [ ] No hardcoded credentials

### Permissions
- [ ] Minimum necessary tools exposed
- [ ] Subagent tool scoping reviewed
- [ ] Permission mode appropriate

## Runtime

### Monitoring
- [ ] Usage logging enabled
- [ ] Alerts for unusual activity
- [ ] Regular access review

### Restrictions
- [ ] No production access without approval
- [ ] Test data only in development
- [ ] Human review for sensitive operations

## Incident Response
- [ ] Rollback procedure documented
- [ ] Contact list for issues
- [ ] Incident report template ready

---

## MCP Server Approval Matrix

| Server | Risk Level | Approval Required | Notes |
|--------|------------|-------------------|-------|
| Playwright | Medium | Team Lead | Browser automation |
| DevTools | Medium | Team Lead | Page inspection |
| GitHub | Medium | Team Lead + Security | Code access |
| Slack | Low | Team Lead | Notifications |
| Database | High | Security Team | Data access |
| Filesystem | High | Security Team | File system |
```

---

## 2. HANDS-ON LAB 1: WEEKLY REPORT SUBAGENT

### 2.1 Lab Overview

**Objective:** Tạo subagent sinh weekly marketing report tự động.

**Duration:** 45-60 phút

**Prerequisites:**
- Đã đọc Part 1-6
- Có project với CLAUDE.md

**Deliverables:**
- Subagent definition file
- Metric calculator skill
- Slash command trigger
- Test với sample data

### 2.2 Step-by-Step Instructions

#### Step 1: Tạo Project Structure (5 min)

```bash
# Tạo thư mục
mkdir -p .claude/agents
mkdir -p .claude/skills/metric-calculator
mkdir -p .claude/commands
mkdir -p data/reports
```

#### Step 2: Tạo Sample Data (5 min)

**File: `data/reports/q1-campaign-week1.json`**

```json
{
  "campaign": "Q1 Brand Awareness",
  "period": {
    "start": "2025-01-01",
    "end": "2025-01-07"
  },
  "metrics": {
    "impressions": 125000,
    "clicks": 2875,
    "conversions": 145,
    "spend": 5000,
    "revenue": 21000
  },
  "previous_week": {
    "impressions": 108695,
    "clicks": 2174,
    "conversions": 120,
    "spend": 4500,
    "revenue": 17500
  },
  "channels": {
    "google": { "spend": 2500, "conversions": 80 },
    "facebook": { "spend": 1500, "conversions": 45 },
    "linkedin": { "spend": 1000, "conversions": 20 }
  }
}
```

#### Step 3: Tạo Metric Calculator Skill (10 min)

**File: `.claude/skills/metric-calculator/SKILL.md`**

```markdown
---
name: metric-calculator
description: Calculates marketing metrics like CTR, CPA, ROAS from campaign data
---

# Metric Calculator

## Purpose
Calculate standard marketing metrics accurately and consistently.

## Supported Metrics

### Click Metrics
- **CTR** = (Clicks / Impressions) × 100
- **CPC** = Spend / Clicks

### Conversion Metrics
- **CVR** = (Conversions / Clicks) × 100
- **CPA** = Spend / Conversions

### Revenue Metrics
- **ROAS** = Revenue / Spend
- **ROI** = ((Revenue - Spend) / Spend) × 100

## Instructions

### Step 1: Identify metric needed
Check what metric user/agent needs.

### Step 2: Apply formula
Use formulas above. Handle division by zero.

### Step 3: Format output
- Percentages: 2 decimal places + %
- Currency: $ + 2 decimal places
- Ratios: 2 decimal places + x

### Step 4: Compare with benchmarks
| Metric | Poor | Average | Good |
|--------|------|---------|------|
| CTR | <1% | 1-2% | >2% |
| CVR | <2% | 2-5% | >5% |
| ROAS | <3x | 3-5x | >5x |

## Example

**Input:**
- Impressions: 125,000
- Clicks: 2,875
- Conversions: 145
- Spend: $5,000
- Revenue: $21,000

**Output:**
- CTR: 2.30% (Good)
- CVR: 5.04% (Good)
- CPA: $34.48
- ROAS: 4.20x (Good)
```

#### Step 4: Tạo Weekly Report Subagent (15 min)

**File: `.claude/agents/weekly-report-generator.md`**

```markdown
---
name: weekly-report-generator
description: Generates weekly marketing performance reports with metrics analysis and recommendations
model: sonnet
tools: Read, Glob, Grep
permissionMode: default
skills: metric-calculator
---

# Weekly Report Generator - MangoAds

## Role
You are a senior marketing analyst generating weekly performance reports.

## Tasks

### 1. Data Collection
- Read campaign data from data/reports/
- Parse JSON structure
- Validate required fields

### 2. Metric Calculation
Use metric-calculator skill to compute:
- CTR, CVR, CPA, ROAS
- Week-over-week changes

### 3. Analysis
- Identify top performers
- Flag underperformers
- Spot trends

### 4. Recommendations
- Actionable optimization suggestions
- Budget reallocation ideas
- A/B test proposals

## Output Format

```markdown
# Weekly Report: [Campaign Name]
**Period:** [Start] - [End]
**Generated:** [Today]

## Executive Summary
- [Key insight 1]
- [Key insight 2]
- [Key insight 3]

## Performance Overview

### This Week vs Last Week
| Metric | This Week | Last Week | Change |
|--------|-----------|-----------|--------|
| Impressions | X | Y | +Z% |
| Clicks | X | Y | +Z% |
| CTR | X% | Y% | +Z% |
| Conversions | X | Y | +Z% |
| CVR | X% | Y% | +Z% |
| Spend | $X | $Y | +Z% |
| Revenue | $X | $Y | +Z% |
| ROAS | Xx | Yx | +Z% |
| CPA | $X | $Y | +Z% |

### Performance Assessment
🟢 Good: [metrics performing well]
🟡 Average: [metrics at benchmark]
🔴 Needs attention: [metrics below benchmark]

## Channel Breakdown
| Channel | Spend | Conv | CPA | % of Total |
|---------|-------|------|-----|------------|
| ... | ... | ... | ... | ... |

## Top Performers
1. [Best performing element]
2. [Second best]

## Areas for Improvement
1. [Issue + suggested action]
2. [Issue + suggested action]

## Recommendations
1. **[Category]:** [Specific recommendation]
2. **[Category]:** [Specific recommendation]

---
*Report generated by MangoAds Weekly Report System*
```

## Constraints
- Always verify data before reporting
- Never fabricate numbers
- Flag missing data clearly
- Use consistent formatting
```

#### Step 5: Tạo Slash Command (10 min)

**File: `.claude/commands/weekly-report.md`**

```markdown
---
description: Generate weekly marketing report from campaign data
allowed-tools: Read, Glob, Grep
argument-hint: "campaign-name [week-number]"
model: sonnet
---

# Weekly Report Command

Generate weekly performance report for specified campaign.

## Arguments
- `$ARGUMENTS[0]`: Campaign name (e.g., "q1-campaign")
- `$ARGUMENTS[1]`: Week number (optional, defaults to latest)

## Process
1. Find data file in data/reports/
2. Invoke weekly-report-generator subagent
3. Generate formatted report
4. Display to user

## Data File Format
Expected: data/reports/[campaign-name]-week[N].json

## Usage
```
/weekly-report q1-campaign
/weekly-report q1-campaign 1
```
```

#### Step 6: Test (10 min)

```bash
# Start Claude Code
claude

# Test command
> /weekly-report q1-campaign

# Verify output has:
# - Executive summary
# - Metrics table
# - Week-over-week comparison
# - Recommendations
```

### 2.3 Lab Checklist

```markdown
## Lab 1 Completion Checklist

### Files Created
- [ ] .claude/skills/metric-calculator/SKILL.md
- [ ] .claude/agents/weekly-report-generator.md
- [ ] .claude/commands/weekly-report.md
- [ ] data/reports/q1-campaign-week1.json

### Functionality
- [ ] /weekly-report command works
- [ ] Metrics calculated correctly
- [ ] Week-over-week comparison shows
- [ ] Recommendations provided

### Quality
- [ ] Output format matches template
- [ ] Numbers formatted correctly
- [ ] Assessment colors (🟢🟡🔴) accurate
```

---

## 3. HANDS-ON LAB 2: UTM GENERATOR SKILL

### 3.1 Lab Overview

**Objective:** Tạo skill sinh UTM links theo chuẩn MangoAds.

**Duration:** 30-45 phút

**Prerequisites:**
- Đã đọc Part 2 (Skills)
- Hiểu UTM parameters

**Deliverables:**
- SKILL.md file
- Optional: Slash command wrapper
- Test với các scenarios

### 3.2 Step-by-Step Instructions

#### Step 1: Tạo Skill Directory (2 min)

```bash
mkdir -p .claude/skills/utm-generator
```

#### Step 2: Tạo SKILL.md (20 min)

**File: `.claude/skills/utm-generator/SKILL.md`**

```markdown
---
name: utm-generator
description: Generates UTM tracking links following MangoAds conventions for GA4 tracking
---

# UTM Link Generator - MangoAds

## Purpose
Generate properly formatted UTM tracking links that comply with MangoAds standards for Google Analytics 4.

## When to Use
- Creating campaign tracking links
- Setting up ad destination URLs
- Email marketing links
- Social media campaign URLs
- Any marketing link needing tracking

## UTM Parameters

### Required Parameters
| Parameter | Description | Format |
|-----------|-------------|--------|
| utm_source | Traffic source | lowercase |
| utm_medium | Marketing medium | lowercase |
| utm_campaign | Campaign name | year-quarter-name |

### Optional Parameters
| Parameter | Description | Format |
|-----------|-------------|--------|
| utm_content | Ad/link variant | kebab-case |
| utm_term | Paid keywords | lowercase |

## Allowed Values

### utm_source (strict)
- google
- facebook
- instagram
- linkedin
- twitter
- tiktok
- youtube
- email
- newsletter
- direct
- referral

### utm_medium (strict)
- cpc
- ppc
- social
- organic_social
- paid_social
- email
- newsletter
- display
- banner
- video
- affiliate
- referral

## Instructions

### Step 1: Validate Inputs
```
Check:
✓ Source is in allowed list
✓ Medium is in allowed list
✓ Campaign follows format
✓ No spaces or special chars
✓ All lowercase
```

### Step 2: Format Campaign Name
```
Input: "Q1 Product Launch"
Process:
1. Get current year: 2025
2. Identify quarter: Q1
3. Convert description: product-launch
Output: "2025-q1-product-launch"
```

### Step 3: Build URL
```
1. Start with base URL
2. Add ? if no existing params
3. Add & if params exist
4. Append utm_source=value
5. Append utm_medium=value
6. Append utm_campaign=value
7. Append optional params if provided
```

### Step 4: Validate Output
```
Check:
✓ URL is valid
✓ No encoding issues
✓ All params present
✓ No duplicate params
```

## Examples

### Example 1: Basic Google Ads Link
**Input:**
- URL: https://mangoads.com/services
- Source: google
- Medium: cpc
- Campaign: Q1 Brand Awareness

**Output:**
```
https://mangoads.com/services?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-brand-awareness
```

### Example 2: Facebook with Content Variant
**Input:**
- URL: https://mangoads.com/ebook
- Source: facebook
- Medium: paid_social
- Campaign: Q1 Lead Gen
- Content: header-cta

**Output:**
```
https://mangoads.com/ebook?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-lead-gen&utm_content=header-cta
```

### Example 3: Email Newsletter
**Input:**
- URL: https://mangoads.com/blog/post
- Source: newsletter
- Medium: email
- Campaign: Weekly Digest Jan W1

**Output:**
```
https://mangoads.com/blog/post?utm_source=newsletter&utm_medium=email&utm_campaign=2025-q1-weekly-digest-jan-w1
```

### Example 4: URL with Existing Params
**Input:**
- URL: https://mangoads.com/shop?product=widget
- Source: google
- Medium: cpc
- Campaign: Q1 Sales

**Output:**
```
https://mangoads.com/shop?product=widget&utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-sales
```

## Constraints
- NEVER use spaces in any parameter
- NEVER use Vietnamese diacritics
- ALWAYS use lowercase
- ALWAYS include year and quarter in campaign
- NEVER modify base URL path

## Anti-patterns

❌ **Incorrect:**
```
utm_campaign=Q1 Product Launch    # Has spaces
utm_source=Google                 # Uppercase
utm_campaign=sản-phẩm-mới        # Vietnamese
utm_medium=social-media          # Not in allowed list
```

✅ **Correct:**
```
utm_campaign=2025-q1-product-launch
utm_source=google
utm_campaign=2025-q1-san-pham-moi
utm_medium=social
```

## Error Handling
- Invalid source → Suggest closest match
- Invalid medium → Suggest closest match
- Missing required param → Request it
- Invalid URL → Report error
```

#### Step 3: Optional - Tạo Slash Command (10 min)

**File: `.claude/commands/utm.md`**

```markdown
---
description: Generate UTM tracking link
allowed-tools: Read
argument-hint: "url source medium campaign [content]"
model: haiku
---

# UTM Command

Quick UTM link generation.

## Arguments
- url: Base URL
- source: Traffic source
- medium: Marketing medium
- campaign: Campaign description
- content (optional): Ad variant

## Example
```
/utm https://mangoads.com google cpc "Q1 Launch"
```

Output:
```
https://mangoads.com?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-launch
```
```

#### Step 4: Test (10 min)

```bash
claude

# Test 1: Basic usage
> Generate UTM link for mangoads.com with google cpc for Q1 brand campaign

# Test 2: With content
> Create tracking link for facebook paid social, Q1 lead gen, header CTA variant

# Test 3: Error handling
> Generate UTM with source "Social Media" (should suggest correction)

# Test 4: Via command
> /utm https://mangoads.com/landing google cpc "Q1 Sales"
```

### 3.3 Lab Checklist

```markdown
## Lab 2 Completion Checklist

### Files Created
- [ ] .claude/skills/utm-generator/SKILL.md
- [ ] .claude/commands/utm.md (optional)

### Functionality
- [ ] Claude auto-invokes skill for UTM requests
- [ ] All required params validated
- [ ] Campaign name formatted correctly
- [ ] Optional params handled

### Validation
- [ ] Invalid source rejected
- [ ] Invalid medium rejected
- [ ] Spaces converted to hyphens
- [ ] Vietnamese handled appropriately

### Output Quality
- [ ] URL is valid
- [ ] Params correctly encoded
- [ ] Format matches examples
```

---

## 4. HANDS-ON LAB 3: MCP PLAYWRIGHT WEB TEST

### 4.1 Lab Overview

**Objective:** Setup Playwright MCP và test landing page form.

**Duration:** 60-90 phút

**Prerequisites:**
- Đã đọc Part 4 (MCP)
- Node.js installed
- Sample web page to test

**Deliverables:**
- MCP configuration
- QA Tester subagent
- Test execution
- Test report

### 4.2 Step-by-Step Instructions

#### Step 1: Setup MCP Configuration (10 min)

**File: `.mcp.json`**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-playwright"],
      "env": {
        "BROWSER": "chromium",
        "HEADLESS": "false"
      }
    }
  }
}
```

#### Step 2: Verify MCP Setup (5 min)

```bash
# Start Claude Code
claude

# Check MCP servers
> /mcp

# Should see playwright server listed
```

#### Step 3: Tạo Sample Test Page (15 min)

Nếu không có page để test, tạo simple HTML form:

**File: `test-page/index.html`**

```html
<!DOCTYPE html>
<html>
<head>
  <title>MangoAds Test Form</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 500px; margin: 50px auto; }
    .form-group { margin-bottom: 15px; }
    label { display: block; margin-bottom: 5px; }
    input, textarea { width: 100%; padding: 8px; }
    button { background: #007bff; color: white; padding: 10px 20px; border: none; cursor: pointer; }
    .error { color: red; font-size: 12px; }
    .success { background: #d4edda; padding: 15px; margin-top: 20px; }
  </style>
</head>
<body>
  <h1>Contact Form</h1>
  <form id="contactForm">
    <div class="form-group">
      <label for="name">Name *</label>
      <input type="text" id="name" required>
      <div class="error" id="nameError"></div>
    </div>
    <div class="form-group">
      <label for="email">Email *</label>
      <input type="email" id="email" required>
      <div class="error" id="emailError"></div>
    </div>
    <div class="form-group">
      <label for="phone">Phone</label>
      <input type="tel" id="phone">
    </div>
    <div class="form-group">
      <label for="message">Message *</label>
      <textarea id="message" rows="4" required></textarea>
    </div>
    <button type="submit">Submit</button>
  </form>
  <div id="successMessage" class="success" style="display:none;">
    Thank you! Your message has been sent.
  </div>

  <script>
    document.getElementById('contactForm').addEventListener('submit', function(e) {
      e.preventDefault();
      // Simple validation
      let valid = true;
      if (!document.getElementById('name').value) {
        document.getElementById('nameError').textContent = 'Name is required';
        valid = false;
      } else {
        document.getElementById('nameError').textContent = '';
      }
      if (!document.getElementById('email').value.includes('@')) {
        document.getElementById('emailError').textContent = 'Valid email required';
        valid = false;
      } else {
        document.getElementById('emailError').textContent = '';
      }
      if (valid) {
        document.getElementById('contactForm').style.display = 'none';
        document.getElementById('successMessage').style.display = 'block';
      }
    });
  </script>
</body>
</html>
```

Serve the page:
```bash
cd test-page
npx serve .
# Note the URL, e.g., http://localhost:3000
```

#### Step 4: Tạo QA Tester Subagent (15 min)

**File: `.claude/agents/qa-form-tester.md`**

```markdown
---
name: qa-form-tester
description: Tests web forms using Playwright MCP for validation, submission, and UI verification
model: sonnet
tools: Read, mcp__playwright
permissionMode: default
---

# QA Form Tester - MangoAds

## Role
Test web forms thoroughly using browser automation.

## Test Categories

### 1. Happy Path Tests
- Fill all fields correctly
- Submit form
- Verify success message

### 2. Validation Tests
- Submit empty form
- Invalid email format
- Missing required fields

### 3. UI Tests
- Fields render correctly
- Labels visible
- Error messages appear

## Test Flow

```
1. Navigate to page
2. Wait for form load
3. Screenshot initial state
4. Run test cases
5. Screenshot results
6. Generate report
```

## Playwright MCP Tools

Use these tools:
- playwright_navigate: Go to URL
- playwright_fill: Fill input fields
- playwright_click: Click buttons
- playwright_screenshot: Capture screen
- playwright_get_text: Get element text
- playwright_wait: Wait for element

## Output Format

```markdown
# Form Test Report

## Page: [URL]
## Date: [Date]

## Test Results

### Happy Path
| Test | Status | Notes |
|------|--------|-------|
| Fill name | ✅ | |
| Fill email | ✅ | |
| Fill message | ✅ | |
| Submit | ✅ | |
| Success shown | ✅ | |

### Validation Tests
| Test | Status | Notes |
|------|--------|-------|
| Empty submit | ✅ | Error shown |
| Invalid email | ✅ | Error shown |

### Screenshots
- Initial state: [attached]
- After success: [attached]

## Issues Found
1. [Issue if any]

## Overall: PASS/FAIL
```
```

#### Step 5: Run Tests (20 min)

```bash
claude

# Test 1: Navigate to page
> Use playwright to navigate to http://localhost:3000

# Test 2: Fill form
> Fill the contact form with name "Test User", email "test@mangoads.com", message "Test message"

# Test 3: Submit
> Click the submit button and verify success message appears

# Test 4: Validation test
> Navigate back to form, try submitting empty form and verify error messages

# Test 5: Full test
> Run complete form test on http://localhost:3000 using qa-form-tester
```

#### Step 6: Generate Report (10 min)

```bash
> Generate a test report for the form tests we just ran
```

### 4.3 Lab Checklist

```markdown
## Lab 3 Completion Checklist

### Setup
- [ ] .mcp.json configured
- [ ] Playwright MCP server running
- [ ] Test page accessible

### Subagent
- [ ] qa-form-tester.md created
- [ ] Tools correctly scoped
- [ ] Output format defined

### Tests Executed
- [ ] Navigate to page works
- [ ] Form fill works
- [ ] Form submit works
- [ ] Validation errors shown
- [ ] Success message shown
- [ ] Screenshots captured

### Report
- [ ] Test results documented
- [ ] Pass/fail status clear
- [ ] Issues listed if any
```

---

## TÓM TẮT PART 9

### Đã cover trong Part 9:
- [x] Templates chuẩn MangoAds:
  - Subagent specification
  - SKILL.md
  - Hook configuration
  - Slash command
  - Plugin checklist
  - MCP security checklist
- [x] Lab 1: Weekly Report Subagent (45-60 min)
- [x] Lab 2: UTM Generator Skill (30-45 min)
- [x] Lab 3: MCP Playwright Web Test (60-90 min)

### Part 10 sẽ cover:
- [ ] Governance & Safety
- [ ] Rubric đánh giá Subagent
- [ ] Cheatsheet 1 trang

---

**Tiếp theo:** [Part 10: Governance, Rubric & Cheatsheet](./part-10-governance-cheatsheet.md)
