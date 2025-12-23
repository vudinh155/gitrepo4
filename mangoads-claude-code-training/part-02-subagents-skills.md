# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 2: Subagents & Agent Skills

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1

---

## MỤC LỤC PART 2

1. [Subagents - AI Personality Chuyên Biệt](#1-subagents---ai-personality-chuyên-biệt)
2. [Agent Skills - Khả Năng Tái Sử Dụng](#2-agent-skills---khả-năng-tái-sử-dụng)
3. [So Sánh Subagent vs Skill](#3-so-sánh-subagent-vs-skill)
4. [Ví Dụ Thực Tế MangoAds](#4-ví-dụ-thực-tế-mangoads)
5. [Best Practices & Anti-patterns](#5-best-practices--anti-patterns)

---

## 1. SUBAGENTS - AI PERSONALITY CHUYÊN BIỆT

### 1.1 Định nghĩa

**Subagent** là một AI assistant chuyên biệt được thiết kế để xử lý một loại task cụ thể, với:
- Context riêng biệt (không làm ô nhiễm conversation chính)
- Cấu hình tools riêng (principle of least privilege)
- Personality/expertise riêng (prompt system riêng)

> **Analogy MangoAds:** Nếu Claude Code là một agency, thì Subagent là các chuyên gia trong agency đó - Content Specialist, SEO Expert, QA Engineer - mỗi người có chuyên môn riêng.

### 1.2 Kiến trúc Subagent

```
┌─────────────────────────────────────────────────────────────┐
│                     MAIN CLAUDE SESSION                      │
│                                                              │
│  User: "Viết landing page và test nó"                       │
│                                                              │
│  ┌─────────────────────┐    ┌─────────────────────┐        │
│  │   SUBAGENT:         │    │   SUBAGENT:         │        │
│  │   content-writer    │    │   qa-tester         │        │
│  │   ┌─────────────┐   │    │   ┌─────────────┐   │        │
│  │   │ Context     │   │    │   │ Context     │   │        │
│  │   │ riêng       │   │    │   │ riêng       │   │        │
│  │   └─────────────┘   │    │   └─────────────┘   │        │
│  │   Tools: Write,Edit │    │   Tools: Read,      │        │
│  │                     │    │   Playwright MCP    │        │
│  └─────────────────────┘    └─────────────────────┘        │
│                                                              │
│  ← Results returned to main conversation                    │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 File Location & Structure

**Vị trí file:**
```
📁 Project-level (ưu tiên)
└── .claude/agents/
    ├── content-writer.md
    ├── code-reviewer.md
    └── qa-tester.md

📁 User-level (global)
└── ~/.claude/agents/
    ├── general-assistant.md
    └── research-agent.md
```

**Cấu trúc file subagent:**

```markdown
---
name: subagent-name
description: Mô tả ngắn gọn khi nào Claude nên dùng subagent này
model: sonnet
tools: Read, Write, Edit, Glob, Grep
permissionMode: default
skills: skill-name-1, skill-name-2
---

# System Prompt

Bạn là [role description].

## Nhiệm vụ
- Task 1
- Task 2

## Quy tắc
- Rule 1
- Rule 2

## Output format
Describe expected output format...
```

### 1.4 YAML Fields Chi Tiết

#### 1.4.1 name (bắt buộc)

```yaml
name: code-reviewer
```

- Identifier duy nhất cho subagent
- Dùng kebab-case
- Không chứa space hoặc ký tự đặc biệt

#### 1.4.2 description (bắt buộc)

```yaml
description: Reviews code for bugs, security issues, and style violations
```

- Claude dựa vào description để quyết định có invoke subagent không
- Viết rõ ràng, cụ thể use case
- Đây là phần QUAN TRỌNG NHẤT - description tốt = subagent được gọi đúng lúc

**Best practices cho description:**
```yaml
# ✅ TỐT - Cụ thể, actionable
description: Analyzes React components for performance issues, unnecessary re-renders, and suggests optimizations

# ❌ XẤU - Quá chung chung
description: Helps with React
```

#### 1.4.3 model (optional)

```yaml
model: sonnet    # hoặc: opus, haiku, inherit
```

| Model | Khi nào dùng | Cost |
|-------|--------------|------|
| `haiku` | Tasks đơn giản, nhanh | Thấp |
| `sonnet` | Balance giữa quality và speed | Trung bình |
| `opus` | Tasks phức tạp, cần reasoning sâu | Cao |
| `inherit` | Dùng model của main session | Tùy |

**Gợi ý cho MangoAds:**
- Content generation: `sonnet`
- Code review: `sonnet` hoặc `opus`
- Simple formatting: `haiku`
- Complex analysis: `opus`

#### 1.4.4 tools (optional)

```yaml
tools: Read, Write, Edit, Glob, Grep, WebFetch
```

**Built-in tools available:**

| Tool | Chức năng | Risk level |
|------|-----------|------------|
| `Read` | Đọc file | Low |
| `Write` | Tạo file mới | Medium |
| `Edit` | Sửa file | Medium |
| `Bash` | Chạy commands | High |
| `Glob` | Tìm file theo pattern | Low |
| `Grep` | Tìm content trong files | Low |
| `WebFetch` | HTTP requests | Medium |
| `WebSearch` | Tìm kiếm web | Low |

**Nếu không specify tools:** Subagent inherit TẤT CẢ tools từ main session (bao gồm MCP tools)

**Best practice - Principle of Least Privilege:**
```yaml
# Read-only subagent (an toàn)
tools: Read, Glob, Grep

# Content writer (cần write)
tools: Read, Write, Edit, Glob, Grep

# Full access (chỉ khi thực sự cần)
tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch
```

#### 1.4.5 permissionMode (optional)

```yaml
permissionMode: default    # hoặc: acceptEdits, plan, bypassPermissions
```

| Mode | Behavior | Khi nào dùng |
|------|----------|--------------|
| `default` | Hỏi permission trước mỗi action | Learning, sensitive tasks |
| `acceptEdits` | Auto-accept file edits | Trusted development |
| `plan` | Chỉ plan, không execute | Review strategy |
| `bypassPermissions` | Skip all prompts | ⚠️ Dangerous, chỉ dùng trong sandbox |

#### 1.4.6 skills (optional)

```yaml
skills: utm-generator, content-formatter
```

- List các skills mà subagent có thể sử dụng
- Skills phải tồn tại trong `.claude/skills/` hoặc `~/.claude/skills/`

### 1.5 /agents Command

Quản lý subagents qua interactive menu:

```bash
claude
> /agents
```

**Chức năng:**
- List tất cả subagents available
- Xem/edit tool configuration
- Enable/disable tools cho từng subagent
- Xem description và settings

### 1.6 Context Isolation

**Tại sao cần context isolation?**

```
Main conversation:
"Tôi đang build landing page cho campaign Q1..."
+ 1000 lines of context about the project

Khi invoke code-reviewer subagent:
→ KHÔNG gửi toàn bộ 1000 lines
→ CHỈ gửi: task description + relevant code
→ Subagent trả về: review results
→ Results merge vào main conversation
```

**Lợi ích:**
- Không làm ô nhiễm main context
- Subagent focus vào task cụ thể
- Tiết kiệm tokens
- Parallel execution possible

### 1.7 Khi Nào Dùng / Không Dùng Subagent

**✅ NÊN dùng Subagent khi:**

| Scenario | Ví dụ MangoAds |
|----------|----------------|
| Task cần expertise riêng | Security audit, SEO analysis |
| Task có thể run song song | Review code + Write tests |
| Task cần tools khác nhau | Writer không cần Bash |
| Muốn isolate context | Debug session riêng |
| Task lặp đi lặp lại | Weekly report generation |

**❌ KHÔNG nên dùng Subagent khi:**

| Scenario | Lý do |
|----------|-------|
| Task đơn giản, 1-2 bước | Overhead không đáng |
| Cần share context liên tục | Context isolation thành blocker |
| Task exploratory | Cần flexibility |
| Chỉ có 1 task type | Main agent đủ rồi |

---

## 2. AGENT SKILLS - KHẢ NĂNG TÁI SỬ DỤNG

### 2.1 Định nghĩa

**Skill** là một khả năng/expertise được đóng gói, có thể tái sử dụng across projects và agents. Skills là **model-invoked** - Claude tự quyết định khi nào sử dụng dựa trên description.

> **Analogy MangoAds:** Skill giống như một "kỹ năng" trong CV - một người có thể có nhiều skills (Excel, SEO, Copywriting), và họ tự biết khi nào nên dùng skill nào.

### 2.2 So sánh Skill vs Subagent

| Aspect | Skill | Subagent |
|--------|-------|----------|
| **Bản chất** | Khả năng/expertise | AI personality |
| **Context** | Share với caller | Isolated |
| **Reusability** | Cao - dùng ở nhiều nơi | Thấp hơn - specific task |
| **Invocation** | Model tự quyết định | Model tự quyết định |
| **Tools** | Inherit từ caller | Có thể configure riêng |
| **Use case** | Portable expertise | Specialized worker |

### 2.3 File Structure

**Location:**
```
📁 Project skills
└── .claude/skills/
    └── utm-generator/
        ├── SKILL.md           # Required - core instructions
        ├── reference.md       # Optional - detailed docs
        ├── examples.md        # Optional - usage examples
        └── templates/         # Optional - reusable templates
            └── utm-template.txt

📁 User skills (global)
└── ~/.claude/skills/
    └── general-formatting/
        └── SKILL.md
```

### 2.4 SKILL.md Structure

**Template chuẩn:**

```markdown
---
name: skill-name
description: Clear description of what this skill does and when Claude should use it
---

# Skill Name

## Purpose
Explain the skill's purpose and when it's useful.

## Instructions
Step-by-step guidance for using this skill.

### Step 1: ...
### Step 2: ...

## Examples

### Example 1: Basic usage
Input: ...
Output: ...

### Example 2: Advanced usage
Input: ...
Output: ...

## Guidelines
- Best practice 1
- Best practice 2

## Constraints
- Limitation 1
- Limitation 2
```

### 2.5 Skill Types

#### 2.5.1 Personal Skills

```
~/.claude/skills/my-skill/SKILL.md
```

- Available trong MỌI projects
- Chỉ user hiện tại có access
- Dùng cho personal preferences/workflows

#### 2.5.2 Project Skills

```
.claude/skills/project-skill/SKILL.md
```

- Available trong project hiện tại
- Share với team qua git
- Priority cao hơn personal skills (nếu trùng name)

#### 2.5.3 Plugin Skills

```
plugin-name/skills/plugin-skill/SKILL.md
```

- Bundled trong plugin
- Distributed qua plugin marketplace
- Auto-registered khi install plugin

### 2.6 Model-Invoked vs User-Invoked

**Skill = Model-Invoked:**
```
User: "Tạo UTM link cho campaign Q1"

Claude (internally):
"Hmm, user cần tạo UTM link..."
"Tôi có skill utm-generator với description phù hợp..."
"Tôi sẽ sử dụng skill này để generate UTM"

→ Claude TỰ QUYẾT ĐỊNH dùng skill
```

**Slash Command = User-Invoked:**
```
User: "/utm q1-campaign google cpc"

→ User CHỦ ĐỘNG invoke command
```

### 2.7 Skill Discovery Mechanism

Claude Code scan skills khi startup:

```
1. Load skill name + description vào system prompt
2. KHÔNG load full content (tiết kiệm context)
3. Khi Claude quyết định dùng skill:
   → Load full SKILL.md content
   → Apply instructions
4. Skill selection = Language model decision
   (KHÔNG dùng embeddings/classifiers)
```

### 2.8 Ví dụ Skill MangoAds

**File: `.claude/skills/utm-generator/SKILL.md`**

```markdown
---
name: utm-generator
description: Generates UTM tracking links following MangoAds conventions for campaign tracking and analytics
---

# UTM Link Generator - MangoAds

## Purpose
Generate properly formatted UTM links that comply with MangoAds tracking standards for Google Analytics 4.

## When to Use
- Creating campaign tracking links
- Setting up ad URLs
- Email marketing links
- Social media campaign URLs

## UTM Parameters

### Required Parameters
| Parameter | Description | Format |
|-----------|-------------|--------|
| utm_source | Traffic source | lowercase, no spaces |
| utm_medium | Marketing medium | lowercase, predefined values |
| utm_campaign | Campaign name | kebab-case, year-quarter-name |

### Optional Parameters
| Parameter | Description | Format |
|-----------|-------------|--------|
| utm_content | Ad/link variant | kebab-case |
| utm_term | Paid keywords | lowercase, + for spaces |

## Allowed Values

### utm_source
- google, facebook, instagram, linkedin, twitter
- email, newsletter
- direct, referral

### utm_medium
- cpc, ppc (paid search)
- social, organic_social, paid_social
- email, newsletter
- display, banner
- affiliate, referral

## Instructions

### Step 1: Validate inputs
- Check all required parameters present
- Verify values are in allowed lists
- Convert to lowercase if needed

### Step 2: Format campaign name
- Use format: {year}-{quarter}-{campaign-description}
- Example: 2025-q1-product-launch

### Step 3: Build URL
- Start with base URL
- Append ? or & as needed
- Add each UTM parameter

### Step 4: Validate output
- Test URL is valid
- Check no special characters except allowed
- Verify tracking will work in GA4

## Examples

### Example 1: Basic Campaign Link
Input:
- URL: https://mangoads.com/services
- Source: google
- Medium: cpc
- Campaign: Q1 2025 Brand Awareness

Output:
```
https://mangoads.com/services?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-brand-awareness
```

### Example 2: Email Campaign with Content Variant
Input:
- URL: https://mangoads.com/ebook
- Source: email
- Medium: newsletter
- Campaign: Q1 2025 Lead Gen
- Content: header-cta

Output:
```
https://mangoads.com/ebook?utm_source=email&utm_medium=newsletter&utm_campaign=2025-q1-lead-gen&utm_content=header-cta
```

## Constraints
- NEVER use spaces in any parameter
- NEVER use Vietnamese diacritics
- ALWAYS use lowercase
- Campaign name MUST include year and quarter

## Anti-patterns
❌ utm_campaign=Q1 Product Launch (spaces, uppercase)
❌ utm_source=Google (uppercase)
❌ utm_campaign=sản-phẩm-mới (Vietnamese)
```

---

## 3. SO SÁNH SUBAGENT VS SKILL

### 3.1 Decision Matrix

```
                    ┌─────────────────────────────────────┐
                    │      CẦN CONTEXT ISOLATION?         │
                    │                                     │
                    │      YES              NO            │
                    └───────┬───────────────┬─────────────┘
                            │               │
                            ▼               ▼
                    ┌───────────────┐ ┌───────────────┐
                    │   SUBAGENT    │ │               │
                    └───────────────┘ │  CẦN TOOLS    │
                                      │  RIÊNG?       │
                                      │               │
                                      │  YES    NO    │
                                      └───┬──────┬────┘
                                          │      │
                                          ▼      ▼
                                   ┌──────────┐ ┌──────────┐
                                   │ SUBAGENT │ │  SKILL   │
                                   └──────────┘ └──────────┘
```

### 3.2 Comparison Table

| Criteria | Chọn Skill | Chọn Subagent |
|----------|------------|---------------|
| Portable across projects | ✅ | ❌ |
| Need isolated context | ❌ | ✅ |
| Need specific tools only | ❌ | ✅ |
| Reusable by multiple agents | ✅ | ❌ |
| Parallel execution | ❌ | ✅ |
| Share via team plugin | ✅ | ✅ |
| Complex multi-step task | ❌ | ✅ |
| Simple transformation | ✅ | ❌ |

### 3.3 Ví dụ Decision

**Scenario 1: Generate UTM links**
```
Q: Cần isolated context? → NO
Q: Cần specific tools? → NO (chỉ cần string manipulation)
Q: Reusable across projects? → YES

→ Answer: SKILL
```

**Scenario 2: Review code cho security issues**
```
Q: Cần isolated context? → YES (không muốn mix với main task)
Q: Cần specific tools? → YES (chỉ Read, Grep, không cần Write)
Q: Complex analysis? → YES

→ Answer: SUBAGENT
```

**Scenario 3: Format markdown tables**
```
Q: Cần isolated context? → NO
Q: Cần specific tools? → NO
Q: Simple transformation? → YES

→ Answer: SKILL
```

---

## 4. VÍ DỤ THỰC TẾ MANGOADS

### 4.1 Subagent: Weekly Report Generator

**File: `.claude/agents/weekly-report-generator.md`**

```markdown
---
name: weekly-report-generator
description: Generates weekly marketing performance reports from campaign data, including metrics analysis and recommendations
model: sonnet
tools: Read, Glob, Grep, WebFetch
permissionMode: default
skills: utm-generator, metric-calculator
---

# Weekly Report Generator - MangoAds

Bạn là chuyên gia phân tích marketing data của MangoAds. Nhiệm vụ của bạn là tạo báo cáo tuần cho các campaign.

## Nhiệm vụ chính

1. **Thu thập data**
   - Đọc file data từ thư mục reports/
   - Parse metrics: impressions, clicks, conversions, spend, revenue

2. **Phân tích performance**
   - Tính CTR, CPA, ROAS
   - So sánh với tuần trước
   - Identify top/bottom performers

3. **Tạo insights**
   - Highlight điểm đáng chú ý
   - Đề xuất optimization
   - Flag anomalies

4. **Format report**
   - Executive summary (3-5 bullet points)
   - Detailed metrics table
   - Week-over-week comparison
   - Recommendations

## Output Format

```markdown
# Weekly Report: [Campaign Name]
**Period:** [Start Date] - [End Date]
**Prepared by:** AI Assistant
**Date:** [Today]

## Executive Summary
- Key insight 1
- Key insight 2
- Key insight 3

## Performance Metrics
| Metric | This Week | Last Week | Change |
|--------|-----------|-----------|--------|
| Impressions | X | Y | +Z% |
| Clicks | X | Y | +Z% |
| CTR | X% | Y% | +Z% |
| Conversions | X | Y | +Z% |
| CPA | $X | $Y | +Z% |
| Spend | $X | $Y | +Z% |
| Revenue | $X | $Y | +Z% |
| ROAS | X | Y | +Z% |

## Top Performers
1. ...
2. ...

## Areas for Improvement
1. ...
2. ...

## Recommendations
1. ...
2. ...
```

## Constraints
- Luôn verify data trước khi report
- Không fabricate numbers
- Flag nếu data incomplete
- Use consistent number formatting (2 decimal places)
```

### 4.2 Subagent: Landing Page Reviewer

**File: `.claude/agents/landing-page-reviewer.md`**

```markdown
---
name: landing-page-reviewer
description: Reviews landing pages for UX, performance, conversion optimization, and MangoAds quality standards
model: opus
tools: Read, Glob, Grep, WebFetch
permissionMode: plan
---

# Landing Page Reviewer - MangoAds

Bạn là UX expert và CRO specialist. Nhiệm vụ: review landing pages theo standards MangoAds.

## Review Checklist

### 1. Technical Performance
- [ ] Page load time < 3s
- [ ] Mobile responsive
- [ ] No console errors
- [ ] Images optimized
- [ ] Proper meta tags

### 2. UX/UI
- [ ] Clear value proposition above fold
- [ ] Single focused CTA
- [ ] Trust signals present
- [ ] Easy-to-read typography
- [ ] Consistent branding

### 3. Conversion Optimization
- [ ] Form fields minimal
- [ ] Clear benefit statements
- [ ] Social proof elements
- [ ] Urgency/scarcity (if applicable)
- [ ] Exit intent considered

### 4. SEO Basics
- [ ] Title tag optimized
- [ ] Meta description present
- [ ] H1 contains keyword
- [ ] Alt tags on images

### 5. Tracking
- [ ] GA4 properly installed
- [ ] Conversion tracking setup
- [ ] UTM parameters working
- [ ] Heatmap tool present

## Output Format

```markdown
# Landing Page Review: [URL]

## Overall Score: X/100

## Summary
[2-3 sentence overview]

## Detailed Findings

### ✅ Strengths
1. ...
2. ...

### ⚠️ Issues Found
1. [Issue] - [Impact: High/Medium/Low] - [Recommendation]
2. ...

### 🔧 Quick Wins
1. ...
2. ...

## Priority Actions
1. [Most critical fix]
2. [Second priority]
3. [Third priority]
```
```

### 4.3 Skill: Metric Calculator

**File: `.claude/skills/metric-calculator/SKILL.md`**

```markdown
---
name: metric-calculator
description: Calculates marketing metrics like CTR, CPA, ROAS, and LTV from campaign data
---

# Metric Calculator - MangoAds

## Purpose
Calculate standard marketing metrics accurately and consistently.

## Metrics Supported

### Click Metrics
- **CTR** (Click-Through Rate) = Clicks / Impressions × 100
- **CPC** (Cost Per Click) = Spend / Clicks

### Conversion Metrics
- **CVR** (Conversion Rate) = Conversions / Clicks × 100
- **CPA** (Cost Per Acquisition) = Spend / Conversions
- **CPL** (Cost Per Lead) = Spend / Leads

### Revenue Metrics
- **ROAS** (Return on Ad Spend) = Revenue / Spend
- **ROI** (Return on Investment) = (Revenue - Spend) / Spend × 100
- **AOV** (Average Order Value) = Revenue / Orders

### Engagement Metrics
- **Engagement Rate** = (Likes + Comments + Shares) / Impressions × 100
- **Bounce Rate** = Single Page Sessions / Total Sessions × 100

## Instructions

### Step 1: Identify required metric
- Understand what metric user needs
- Check if all inputs are available

### Step 2: Apply formula
- Use formulas above
- Handle division by zero gracefully

### Step 3: Format output
- Round to 2 decimal places
- Add % symbol for percentages
- Add $ symbol for currency

### Step 4: Provide context
- Compare to benchmarks if known
- Flag if value seems unusual

## Examples

### Example 1: CTR Calculation
Input: 50,000 impressions, 1,250 clicks
Calculation: 1,250 / 50,000 × 100 = 2.5%
Output: CTR = 2.5% (Industry avg: 1-3%)

### Example 2: ROAS Calculation
Input: $5,000 spend, $25,000 revenue
Calculation: $25,000 / $5,000 = 5.0
Output: ROAS = 5.0x (Every $1 spent returns $5)

## Benchmarks (MangoAds Reference)

| Metric | Poor | Average | Good | Excellent |
|--------|------|---------|------|-----------|
| CTR (Search) | <1% | 1-2% | 2-4% | >4% |
| CTR (Display) | <0.1% | 0.1-0.3% | 0.3-0.5% | >0.5% |
| CVR (Landing) | <1% | 1-3% | 3-5% | >5% |
| ROAS | <2x | 2-4x | 4-6x | >6x |
| CPA | Varies by industry |

## Constraints
- Never divide by zero (return "N/A - insufficient data")
- Always show calculation steps if asked
- Flag metrics that seem outside normal range
```

---

## 5. BEST PRACTICES & ANTI-PATTERNS

### 5.1 Subagent Best Practices

**✅ DO:**

```markdown
1. Description rõ ràng
   description: Reviews React components for performance issues and suggests memoization opportunities
   (Không phải: "Helps with React")

2. Least privilege tools
   tools: Read, Glob, Grep  # Read-only cho reviewer
   (Không phải: tools để trống → inherit all)

3. Specific system prompt
   Bạn là senior React developer với expertise về performance optimization...
   (Không phải: "Bạn là AI assistant")

4. Clear output format
   ## Output Format
   - Summary
   - Issues found
   - Recommendations
```

**❌ DON'T (Anti-patterns):**

```markdown
1. Subagent làm mọi thứ
   name: super-agent
   description: Does everything
   → Không có focus, không biết khi nào invoke

2. Không restrict tools
   # Subagent reviewer có quyền Write/Bash
   → Risk: reviewer có thể modify code

3. Description quá ngắn
   description: Reviews code
   → Claude không biết dùng khi nào

4. Copy-paste từ main agent
   # Subagent giống hệt main agent
   → Tại sao cần subagent?
```

### 5.2 Skill Best Practices

**✅ DO:**

```markdown
1. Single responsibility
   name: utm-generator  # Chỉ làm 1 việc
   (Không phải: marketing-everything)

2. Clear examples
   ### Example 1: Basic usage
   Input: ...
   Output: ...

3. Constraints section
   ## Constraints
   - Never use spaces
   - Always lowercase

4. Reusable across contexts
   # Skill có thể dùng bởi main agent hoặc bất kỳ subagent nào
```

**❌ DON'T (Anti-patterns):**

```markdown
1. Skill quá phức tạp
   # 500+ lines SKILL.md với 50 features
   → Chia thành multiple skills

2. Hard-coded project specifics
   utm_campaign=mangoads-q1-2025
   → Làm skill không portable

3. Thiếu examples
   # Instructions mà không có ví dụ cụ thể
   → Khó understand và dễ sai

4. Overlap với built-in capabilities
   # Skill "đọc file"
   → Claude đã có Read tool rồi
```

### 5.3 Checklist Tạo Subagent/Skill

```markdown
## Checklist Subagent

### Definition
- [ ] Name là kebab-case, descriptive
- [ ] Description rõ ràng khi nào dùng
- [ ] Model phù hợp với task complexity

### Configuration
- [ ] Tools chỉ những cần thiết
- [ ] Permission mode appropriate
- [ ] Skills list (nếu cần)

### System Prompt
- [ ] Role definition rõ ràng
- [ ] Task instructions cụ thể
- [ ] Output format defined
- [ ] Constraints listed

### Testing
- [ ] Test với các scenarios khác nhau
- [ ] Verify Claude invoke đúng lúc
- [ ] Check output quality

---

## Checklist Skill

### Definition
- [ ] Name unique và descriptive
- [ ] Description rõ use case
- [ ] Single responsibility

### Content
- [ ] Purpose section
- [ ] Step-by-step instructions
- [ ] Multiple examples
- [ ] Constraints/limitations

### Quality
- [ ] Portable (không hard-code project specifics)
- [ ] Tested across different inputs
- [ ] Documented edge cases
```

---

## TÓM TẮT PART 2

### Đã cover trong Part 2:
- [x] Subagents - Định nghĩa, cấu trúc, YAML fields
- [x] Context isolation và /agents command
- [x] Agent Skills - SKILL.md structure
- [x] Personal vs Project vs Plugin skills
- [x] Model-invoked vs User-invoked
- [x] So sánh Subagent vs Skill (Decision matrix)
- [x] Ví dụ thực tế MangoAds (Weekly Report, Landing Page Reviewer, Metric Calculator)
- [x] Best practices và Anti-patterns

### Part 3 sẽ cover:
- [ ] Hooks - Lifecycle hooks trong Claude Code
- [ ] PreToolUse, Stop, SubagentStop events
- [ ] Matcher và type (command vs prompt)
- [ ] Slash Commands chi tiết
- [ ] MCP slash commands
- [ ] Ví dụ thực tế MangoAds

---

**Tiếp theo:** [Part 3: Hooks & Slash Commands](./part-03-hooks-commands.md)
