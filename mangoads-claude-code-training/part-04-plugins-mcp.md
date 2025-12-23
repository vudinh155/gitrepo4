# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 4: Plugins & MCP (Model Context Protocol)

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1, 2, 3

---

## MỤC LỤC PART 4

1. [Plugins - Bundle & Distribution](#1-plugins---bundle--distribution)
2. [MCP (Model Context Protocol) - Giải Thích Chi Tiết](#2-mcp-model-context-protocol---giải-thích-chi-tiết)
3. [Playwright MCP - Web Testing & Automation](#3-playwright-mcp---web-testing--automation)
4. [Browser DevTools MCP - Debug & Performance](#4-browser-devtools-mcp---debug--performance)
5. [MCP Security Model](#5-mcp-security-model)
6. [Ứng Dụng MCP tại MangoAds](#6-ứng-dụng-mcp-tại-mangoads)

---

## 1. PLUGINS - BUNDLE & DISTRIBUTION

### 1.1 Định nghĩa

**Plugin** là một package đóng gói các thành phần Claude Code để:
- Share trong team
- Version control
- Distribute qua marketplace
- Reuse across projects

> **Analogy MangoAds:** Plugin giống như một "toolkit" chuyên dụng - thay vì mỗi người tự build từ đầu, team dùng chung toolkit đã được tối ưu và test kỹ.

### 1.2 Plugin vs Standalone .claude/

| Aspect | Standalone .claude/ | Plugin |
|--------|---------------------|--------|
| Scope | Single project | Shareable |
| Versioning | Manual | Semantic versioning |
| Distribution | Git clone | Plugin marketplace |
| Updates | Manual sync | Plugin update command |
| Bundling | Loose files | Packaged bundle |

### 1.3 Plugin Directory Structure

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json           # Plugin metadata (REQUIRED)
├── commands/                  # Slash commands
│   ├── report.md
│   └── analyze.md
├── agents/                    # Subagents
│   ├── content-writer.md
│   └── code-reviewer.md
├── skills/                    # Agent Skills
│   └── utm-generator/
│       └── SKILL.md
├── hooks/                     # Event handlers
│   └── hooks.json
├── .mcp.json                  # MCP server config
└── README.md
```

**⚠️ Quan trọng:** Components phải ở ROOT của plugin, KHÔNG phải trong `.claude-plugin/`. Chỉ `plugin.json` nằm trong `.claude-plugin/`.

### 1.4 plugin.json Structure

```json
{
  "name": "mangoads-marketing-toolkit",
  "version": "1.0.0",
  "description": "Marketing automation toolkit for MangoAds team",
  "author": "MangoAds Dev Team",
  "homepage": "https://github.com/mangoads/marketing-toolkit",
  "keywords": ["marketing", "utm", "reporting", "automation"],
  "dependencies": {
    "node": ">=18.0.0"
  },
  "claude": {
    "minVersion": "1.0.0"
  }
}
```

### 1.5 Plugin Installation & Management

**Install từ marketplace:**
```bash
claude
> /plugin marketplace add mangoads/marketing-toolkit
```

**Install từ local path:**
```bash
> /plugin add /path/to/plugin
```

**List installed plugins:**
```bash
> /plugin list
```

**Toggle plugin on/off:**
```bash
> /plugin toggle mangoads-marketing-toolkit
```

**Update plugin:**
```bash
> /plugin update mangoads-marketing-toolkit
```

### 1.6 Khi nào nên tạo Plugin?

**✅ NÊN tạo Plugin khi:**

| Scenario | Ví dụ MangoAds |
|----------|----------------|
| Share across multiple projects | UTM toolkit dùng cho mọi campaign |
| Team cần standardization | Code review standards |
| Version control quan trọng | Marketing automation v1, v2 |
| External distribution | Share với clients |

**❌ KHÔNG cần Plugin khi:**

| Scenario | Giải pháp thay thế |
|----------|-------------------|
| Project-specific logic | Dùng .claude/ trực tiếp |
| Experimental features | Test trong .claude/ trước |
| Single person use | Personal ~/.claude/ |

### 1.7 Ví dụ Plugin MangoAds

**MangoAds Marketing Toolkit Plugin:**

```
mangoads-marketing-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── utm.md                    # Generate UTM links
│   ├── weekly-report.md          # Weekly performance report
│   └── campaign-brief.md         # Generate campaign brief
├── agents/
│   ├── content-writer.md         # Content generation
│   ├── seo-analyzer.md           # SEO analysis
│   └── performance-analyst.md    # Campaign analysis
├── skills/
│   ├── utm-generator/
│   │   └── SKILL.md
│   ├── metric-calculator/
│   │   └── SKILL.md
│   └── content-formatter/
│       └── SKILL.md
├── hooks/
│   └── hooks.json                # Quality gates
└── README.md
```

**plugin.json:**
```json
{
  "name": "mangoads-marketing-toolkit",
  "version": "2.0.0",
  "description": "Complete marketing automation toolkit for MangoAds team. Includes UTM generation, reporting, and campaign management.",
  "author": "MangoAds Dev Team",
  "keywords": ["marketing", "utm", "reporting", "campaign", "analytics"],
  "claude": {
    "minVersion": "2.0.0"
  }
}
```

---

## 2. MCP (MODEL CONTEXT PROTOCOL) - GIẢI THÍCH CHI TIẾT

### 2.1 MCP là gì?

**MCP (Model Context Protocol)** là một open standard và framework để standardize cách LLMs tích hợp với tools, systems, và data sources bên ngoài.

> **Analogy:** MCP giống như **USB-C port cho AI** - một chuẩn kết nối universal để AI có thể "cắm vào" bất kỳ tool nào.

### 2.2 Vấn đề MCP giải quyết

**Trước MCP:**
```
┌─────────────────────────────────────────────────────────────┐
│                    TRƯỚC MCP                                 │
│                                                              │
│  ┌──────────┐     Custom API      ┌──────────┐             │
│  │  Claude  │────────────────────►│  GitHub  │             │
│  │          │                      └──────────┘             │
│  │          │     Custom API      ┌──────────┐             │
│  │          │────────────────────►│  Slack   │             │
│  │          │                      └──────────┘             │
│  │          │     Custom API      ┌──────────┐             │
│  │          │────────────────────►│  Browser │             │
│  └──────────┘                      └──────────┘             │
│                                                              │
│  Problem: Mỗi integration cần custom implementation!        │
└─────────────────────────────────────────────────────────────┘
```

**Với MCP:**
```
┌─────────────────────────────────────────────────────────────┐
│                    VỚI MCP                                   │
│                                                              │
│  ┌──────────┐                      ┌──────────┐             │
│  │  Claude  │      Standard        │  GitHub  │             │
│  │  (MCP    │◄────── MCP ─────────►│  MCP     │             │
│  │  Client) │      Protocol        │  Server  │             │
│  │          │                      └──────────┘             │
│  │          │                      ┌──────────┐             │
│  │          │◄────── MCP ─────────►│  Slack   │             │
│  │          │                      │  MCP     │             │
│  │          │                      │  Server  │             │
│  │          │                      └──────────┘             │
│  │          │                      ┌──────────┐             │
│  │          │◄────── MCP ─────────►│Playwright│             │
│  │          │                      │  MCP     │             │
│  └──────────┘                      │  Server  │             │
│                                    └──────────┘             │
│                                                              │
│  Solution: MỘT protocol chuẩn cho TẤT CẢ integrations!      │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 MCP Server vs MCP Client

```
┌─────────────────────────────────────────────────────────────┐
│                   MCP ARCHITECTURE                           │
│                                                              │
│  ┌─────────────────────┐          ┌─────────────────────┐   │
│  │     MCP CLIENT      │          │     MCP SERVER      │   │
│  │   (Claude Code)     │◄────────►│   (External Tool)   │   │
│  │                     │  JSON-RPC │                     │   │
│  │  - Sends requests   │   2.0    │  - Exposes tools    │   │
│  │  - Receives results │  over    │  - Exposes resources│   │
│  │  - Manages sessions │  stdio   │  - Exposes prompts  │   │
│  └─────────────────────┘  or HTTP │                     │   │
│                                    └─────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**MCP Client (Claude Code):**
- Là consumer của MCP services
- Gửi requests đến servers
- Nhận và process responses
- Quản lý permissions

**MCP Server (Tool Provider):**
- Standalone process
- Expose capabilities qua MCP protocol
- Handle requests từ client
- Return structured responses

### 2.4 MCP Expose: Tools, Resources, Prompts

#### 2.4.1 Tools

**Định nghĩa:** Actions có side effects (gọi APIs, write data, execute commands)

```json
{
  "tools": [
    {
      "name": "playwright_navigate",
      "description": "Navigate browser to URL",
      "inputSchema": {
        "type": "object",
        "properties": {
          "url": {"type": "string"}
        },
        "required": ["url"]
      }
    },
    {
      "name": "playwright_click",
      "description": "Click on element",
      "inputSchema": {
        "type": "object",
        "properties": {
          "selector": {"type": "string"}
        },
        "required": ["selector"]
      }
    }
  ]
}
```

**Cách Claude sử dụng:**
```
User: "Click vào nút Submit"
Claude: [Sử dụng playwright_click tool với selector phù hợp]
```

#### 2.4.2 Resources

**Định nghĩa:** Read-only data sources (không có side effects)

```json
{
  "resources": [
    {
      "uri": "github://repo/issues",
      "name": "GitHub Issues",
      "description": "List of issues in repository"
    },
    {
      "uri": "database://users/active",
      "name": "Active Users",
      "description": "Currently active user list"
    }
  ]
}
```

**Cách sử dụng trong Claude Code:**
```
User: "Show me @github:mangoads/website/issues"
Claude: [Fetch resource và hiển thị]
```

#### 2.4.3 Prompts

**Định nghĩa:** Reusable templates/workflows → trở thành slash commands

```json
{
  "prompts": [
    {
      "name": "create-test",
      "description": "Create E2E test for page",
      "arguments": [
        {"name": "url", "description": "URL to test", "required": true},
        {"name": "test-name", "description": "Name for test", "required": false}
      ]
    }
  ]
}
```

**Sử dụng:**
```
/mcp__playwright__create-test https://mangoads.com/landing-page
```

### 2.5 MCP Configuration trong Claude Code

**File: `.mcp.json` (project) hoặc `~/.claude/.mcp.json` (global)**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-playwright"],
      "env": {
        "BROWSER": "chromium"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-filesystem", "/path/to/allowed/dir"]
    }
  }
}
```

### 2.6 MCP Industry Adoption (2025)

| Company | Adoption Date | Notes |
|---------|---------------|-------|
| Anthropic | Nov 2024 | Creator of MCP |
| OpenAI | Mar 2025 | ChatGPT, Agents SDK |
| Google DeepMind | Apr 2025 | Gemini models |
| Linux Foundation | Dec 2025 | Agentic AI Foundation |

---

## 3. PLAYWRIGHT MCP - WEB TESTING & AUTOMATION

### 3.1 Playwright MCP là gì?

**Playwright MCP** là MCP server cho phép Claude Code điều khiển browser để:
- Navigate đến URLs
- Click elements
- Fill forms
- Take screenshots
- Assert UI state
- Run E2E tests

### 3.2 Cách hoạt động

```
┌─────────────────────────────────────────────────────────────┐
│                 PLAYWRIGHT MCP WORKFLOW                      │
│                                                              │
│  ┌──────────┐    MCP Protocol    ┌──────────────────────┐   │
│  │  Claude  │◄─────────────────►│  Playwright MCP      │   │
│  │  Code    │                    │  Server              │   │
│  └──────────┘                    │                      │   │
│       │                          │  ┌────────────────┐  │   │
│       │ "Navigate to URL"        │  │ Chromium       │  │   │
│       │───────────────────────►  │  │ Browser        │  │   │
│       │                          │  │                │  │   │
│       │ "Click button"           │  │ [Visible       │  │   │
│       │───────────────────────►  │  │  Window]       │  │   │
│       │                          │  │                │  │   │
│       │◄── Screenshot ───────────│  └────────────────┘  │   │
│       │                          │                      │   │
│       │◄── Accessibility Tree ───│                      │   │
│                                  │                      │   │
└─────────────────────────────────────────────────────────────┘
```

**Key features:**
- **Visible browser window:** User có thể thấy browser đang làm gì
- **Accessibility tree:** Claude dùng structured data (không phải screenshots)
- **Deterministic:** Actions chính xác, không cần vision model
- **Cookies persist:** Login state giữ trong session

### 3.3 Cài đặt Playwright MCP

**Bước 1: Thêm vào `.mcp.json`**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"]
    }
  }
}
```

**Bước 2: Verify cài đặt**

```bash
claude
> /mcp
# Should show playwright server listed
```

### 3.4 Playwright MCP Tools Available

| Tool | Description | Example |
|------|-------------|---------|
| `playwright_navigate` | Go to URL | Navigate to https://mangoads.com |
| `playwright_click` | Click element | Click the Submit button |
| `playwright_fill` | Fill input | Fill email field with test@example.com |
| `playwright_screenshot` | Take screenshot | Screenshot current page |
| `playwright_select` | Select dropdown | Select "Vietnam" from country |
| `playwright_hover` | Hover element | Hover over menu item |
| `playwright_evaluate` | Run JS | Execute custom JavaScript |
| `playwright_get_text` | Get text content | Get text from element |
| `playwright_wait` | Wait for condition | Wait for element visible |

### 3.5 Ví dụ sử dụng tại MangoAds

**Use case: Test Landing Page Form**

```
User: "Test form submission trên landing page Q1 campaign"

Claude:
1. playwright_navigate: https://mangoads.com/campaign/q1-2025
2. playwright_fill: [email input] với "test@mangoads.com"
3. playwright_fill: [name input] với "Test User"
4. playwright_click: [submit button]
5. playwright_wait: [success message]
6. playwright_screenshot: Capture result

Output: "Form submission successful. Screenshot captured."
```

**Use case: Visual Regression Check**

```
User: "So sánh homepage với baseline"

Claude:
1. playwright_navigate: https://mangoads.com
2. playwright_screenshot: [current state]
3. So sánh với baseline image
4. Report differences
```

### 3.6 Playwright MCP Prompts → Commands

Khi Playwright MCP expose prompts:

```
/mcp__playwright__create-test [url]
/mcp__playwright__capture-page [url]
/mcp__playwright__test-form [url]
```

---

## 4. BROWSER DEVTOOLS MCP - DEBUG & PERFORMANCE

### 4.1 Browser DevTools MCP là gì?

**Browser DevTools MCP** kết nối với Chrome DevTools Protocol để:
- Monitor network requests
- Read console logs
- Analyze performance
- Inspect DOM
- Execute JavaScript

### 4.2 Capabilities

| Feature | Description | Use Case MangoAds |
|---------|-------------|-------------------|
| **Network monitoring** | Track HTTP requests/responses | Debug API calls on website |
| **Console integration** | Read console logs/errors | Find JavaScript errors |
| **Performance metrics** | Timing, resources, memory | Audit page performance |
| **DOM inspection** | Explore page structure | Debug UI issues |
| **JS execution** | Run JavaScript in context | Test functionality |

### 4.3 Cài đặt

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-chrome-devtools"],
      "env": {
        "CHROME_DEBUG_PORT": "9222"
      }
    }
  }
}
```

**Yêu cầu:** Chrome phải chạy với remote debugging enabled:
```bash
chrome --remote-debugging-port=9222
```

### 4.4 Use Cases tại MangoAds

**Use case 1: Debug JavaScript Errors**

```
User: "Check console errors trên landing page"

Claude (via DevTools MCP):
1. Connect to running Chrome
2. Navigate to page
3. Collect console.error messages
4. Report findings with stack traces
```

**Use case 2: Network Analysis**

```
User: "Analyze API calls khi user submit form"

Claude:
1. Enable network monitoring
2. User submits form
3. Capture all XHR/fetch requests
4. Report:
   - Request URLs
   - Request payloads
   - Response status
   - Response times
```

**Use case 3: Performance Audit**

```
User: "Audit performance của homepage"

Claude:
1. Start performance recording
2. Load page
3. Collect metrics:
   - First Contentful Paint (FCP)
   - Largest Contentful Paint (LCP)
   - Time to Interactive (TTI)
   - Total Blocking Time (TBT)
4. Compare with benchmarks
5. Suggest optimizations
```

---

## 5. MCP SECURITY MODEL

### 5.1 Security Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP SECURITY LAYERS                       │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 1: Permission Control                            │ │
│  │ - User phải approve MCP tool usage                     │ │
│  │ - Can restrict tools per subagent                      │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 2: Environment Isolation                         │ │
│  │ - MCP servers run in separate processes                │ │
│  │ - Network access controls                              │ │
│  │ - Sandbox boundaries                                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Layer 3: Credential Separation                         │ │
│  │ - Credentials KHÔNG nằm trong Claude Code sandbox      │ │
│  │ - MCP server manage credentials riêng                  │ │
│  │ - Env vars passed at server start                      │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Permission Control

**Subagent tool restriction:**

```yaml
---
name: read-only-auditor
description: Audits code without making changes
tools: Read, Grep, Glob
# KHÔNG có MCP tools → không access browser
---
```

**MCP tool trong subagent (khi cần):**

```yaml
---
name: qa-tester
description: Runs automated tests using Playwright
tools: Read, Grep, Glob, mcp__playwright
# Chỉ có Playwright MCP, không có DevTools
---
```

### 5.3 Khi nào KHÔNG cho dùng MCP

| Scenario | Lý do | Alternative |
|----------|-------|-------------|
| Production systems | Risk of accidental changes | Read-only MCP hoặc staging |
| Sensitive data | Data exposure risk | Sanitized test data |
| Without review | Unknown MCP servers | Review source trước |
| Automated pipelines | Unsupervised execution | Human-in-the-loop |

### 5.4 MCP Security Checklist MangoAds

```markdown
## MCP Security Checklist

### Before Installation
- [ ] Review MCP server source code/documentation
- [ ] Check server maintainer reputation
- [ ] Verify no known vulnerabilities
- [ ] Understand what data server can access

### Configuration
- [ ] Use environment variables for credentials (NOT hardcoded)
- [ ] Limit MCP servers to necessary ones only
- [ ] Configure appropriate permissions
- [ ] Set up logging for MCP tool usage

### Runtime
- [ ] Never run MCP on production without approval
- [ ] Use staging/test environments
- [ ] Review MCP actions in sensitive operations
- [ ] Monitor for unusual activity

### Team Policy
- [ ] Document approved MCP servers
- [ ] Require PR review for .mcp.json changes
- [ ] Train team on MCP security best practices
- [ ] Regular audit of MCP usage
```

---

## 6. ỨNG DỤNG MCP TẠI MANGOADS

### 6.1 Web Testing & QA

**Workflow:**

```
┌─────────────────────────────────────────────────────────────┐
│           MANGOADS WEB TESTING WORKFLOW                      │
│                                                              │
│  1. Developer pushes landing page code                      │
│                    │                                         │
│                    ▼                                         │
│  2. Claude Code + Playwright MCP                            │
│     - Navigate to staging URL                               │
│     - Test all forms                                        │
│     - Check responsive layouts                              │
│     - Verify tracking pixels                                │
│                    │                                         │
│                    ▼                                         │
│  3. Generate test report                                    │
│     - Pass/fail status                                      │
│     - Screenshots                                           │
│     - Issues found                                          │
│                    │                                         │
│                    ▼                                         │
│  4. QA reviews report                                       │
│                    │                                         │
│                    ▼                                         │
│  5. Approve/request fixes                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Automation Testing

**Test Landing Page Campaign:**

```markdown
## Automated Test: Landing Page Q1 2025

### Test Cases:

1. **Page Load**
   - Navigate to URL
   - Verify page loads < 3s
   - Check no console errors

2. **Form Submission**
   - Fill all required fields
   - Submit form
   - Verify success message
   - Check data sent to backend

3. **Responsive Layout**
   - Test desktop (1920x1080)
   - Test tablet (768x1024)
   - Test mobile (375x667)

4. **Tracking Verification**
   - Check GA4 pixel fires
   - Check Facebook Pixel fires
   - Verify UTM params captured

5. **Performance Metrics**
   - LCP < 2.5s
   - FID < 100ms
   - CLS < 0.1
```

### 6.3 Visual Regression

**Workflow:**

```
1. Capture baseline screenshots
   - Homepage
   - Key landing pages
   - Forms
   - Mobile views

2. After code changes, run comparison

3. Report differences:
   - Pixel-level diff
   - Structural changes
   - New/removed elements

4. QA reviews:
   - Expected changes → Approve, update baseline
   - Unexpected changes → Bug report
```

### 6.4 Frontend Debugging

**Debug Session Example:**

```
User: "Website MangoAds bị lỗi trên mobile Safari"

Claude + Browser DevTools MCP:
1. Collect console errors
   → Found: "TypeError: undefined is not an object"
   → Source: main.js line 245

2. Analyze network requests
   → API call failing with 404
   → Endpoint: /api/mobile-detect

3. Inspect DOM
   → Missing polyfill for Safari
   → IntersectionObserver not supported

4. Recommendation:
   → Add Safari polyfill
   → Fix API endpoint
   → Test on BrowserStack
```

### 6.5 Pre-deploy Smoke Test

**Checklist tự động:**

```markdown
## Pre-deploy Smoke Test

### Using Playwright MCP:

1. [ ] Homepage loads
2. [ ] Navigation works
3. [ ] Contact form submits
4. [ ] Blog loads
5. [ ] Search works
6. [ ] Login/logout (if applicable)
7. [ ] Cart/checkout (if e-commerce)

### Using DevTools MCP:

1. [ ] No JavaScript errors
2. [ ] No 404 requests
3. [ ] No mixed content warnings
4. [ ] Performance acceptable

### Manual Verification:
1. [ ] Content accuracy
2. [ ] Brand consistency
3. [ ] Legal compliance
```

### 6.6 MCP Configuration Chuẩn MangoAds

**File: `.mcp.json`**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"],
      "env": {
        "BROWSER": "chromium",
        "HEADLESS": "false"
      }
    },
    "devtools": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-chrome-devtools"],
      "env": {
        "CHROME_DEBUG_PORT": "9222"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_TOKEN": "${SLACK_TOKEN}"
      }
    }
  }
}
```

---

## TÓM TẮT PART 4

### Đã cover trong Part 4:
- [x] Plugins - Structure, installation, management
- [x] Khi nào nên tạo Plugin
- [x] MCP (Model Context Protocol) - Giải thích chi tiết
- [x] MCP Server vs Client architecture
- [x] Tools, Resources, Prompts trong MCP
- [x] Playwright MCP - Installation và tools
- [x] Browser DevTools MCP - Capabilities
- [x] MCP Security Model
- [x] Ứng dụng MCP tại MangoAds (Testing, Debugging, Smoke test)

### Part 5 sẽ cover:
- [ ] Tools & Permission Modes chi tiết
- [ ] Tool scoping cho subagents
- [ ] CLI / Interactive / Sessions
- [ ] Checkpointing & Rollback
- [ ] Resume session workflows

---

**Tiếp theo:** [Part 5: Tools, Permissions & CLI](./part-05-tools-permissions-cli.md)
