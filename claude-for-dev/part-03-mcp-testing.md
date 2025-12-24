# PHẦN 3: MCP & TESTING AUTOMATION

## Tài liệu dành cho: Frontend, Backend, QA, DevOps Teams

---

## MỤC LỤC

1. [MCP - Model Context Protocol](#1-mcp---model-context-protocol)
2. [Cài đặt và Cấu hình MCP](#2-cài-đặt-và-cấu-hình-mcp)
3. [Playwright MCP - Browser Automation](#3-playwright-mcp---browser-automation)
4. [DevTools MCP - Debugging](#4-devtools-mcp---debugging)
5. [Testing Automation với Claude Code](#5-testing-automation-với-claude-code)
6. [CI/CD Integration](#6-cicd-integration)
7. [Best Practices & Patterns](#7-best-practices--patterns)

---

## 1. MCP - MODEL CONTEXT PROTOCOL

### 1.1 MCP là gì?

MCP (Model Context Protocol) là một protocol mở cho phép Claude Code kết nối với các external tools và data sources. Đây là cách để mở rộng khả năng của Claude Code vượt ra ngoài các tools mặc định.

```
┌─────────────────────────────────────────────────────────────┐
│                      CLAUDE CODE                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   MCP Client                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                  │
│         ┌─────────────────┼─────────────────┐               │
│         ▼                 ▼                 ▼               │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐       │
│  │ Playwright  │   │  DevTools   │   │   Custom    │       │
│  │    MCP      │   │    MCP      │   │    MCP      │       │
│  └─────────────┘   └─────────────┘   └─────────────┘       │
│         │                 │                 │               │
└─────────┼─────────────────┼─────────────────┼───────────────┘
          ▼                 ▼                 ▼
   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
   │   Browser   │   │   Chrome    │   │  External   │
   │  Automation │   │   DevTools  │   │   Service   │
   └─────────────┘   └─────────────┘   └─────────────┘
```

### 1.2 MCP Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     MCP ARCHITECTURE                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐        Protocol         ┌─────────────┐     │
│  │             │◄───────JSON-RPC────────►│             │     │
│  │  MCP Host   │                         │  MCP Server │     │
│  │ (Claude)    │    ┌─────────────┐      │  (Tool)     │     │
│  │             │◄───│   stdio/    │─────►│             │     │
│  └─────────────┘    │   HTTP/SSE  │      └─────────────┘     │
│                     └─────────────┘                          │
│                                                               │
│  Capabilities:                                                │
│  ├── Tools: Functions Claude can call                        │
│  ├── Resources: Data sources Claude can read                 │
│  ├── Prompts: Pre-defined prompt templates                   │
│  └── Sampling: Server can request LLM completions            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 1.3 MCP vs Built-in Tools

| Aspect | Built-in Tools | MCP Tools |
|--------|---------------|-----------|
| **Scope** | File, Bash, Search | Unlimited external services |
| **Setup** | Ready to use | Requires configuration |
| **Permissions** | Managed by Claude Code | Custom per MCP server |
| **Use Cases** | General development | Specialized workflows |
| **Examples** | Read, Write, Glob | Playwright, Database, APIs |

---

## 2. CÀI ĐẶT VÀ CẤU HÌNH MCP

### 2.1 Cấu trúc Configuration

MCP được cấu hình trong file settings của Claude Code:

```json
// ~/.claude/settings.json (global)
// hoặc .claude/settings.json (project-level)
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-name"],
      "env": {
        "API_KEY": "your-api-key"
      },
      "disabled": false
    }
  }
}
```

### 2.2 Các Transport Types

```
┌──────────────────────────────────────────────────────────────┐
│                    MCP TRANSPORT TYPES                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. STDIO Transport (Most Common)                            │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Claude ◄──stdin/stdout──► Child Process (Server)   │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                               │
│  2. HTTP + SSE Transport                                     │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Claude ◄──HTTP POST──► Server                       │     │
│  │         ◄──SSE Events──►                             │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                               │
│  3. Streamable HTTP Transport                                │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Claude ◄──HTTP Stream──► Server                     │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2.3 Project-level MCP Configuration

```json
// .claude/settings.json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### 2.4 Kiểm tra MCP Status

```bash
# Liệt kê các MCP servers đang active
claude mcp list

# Xem chi tiết một server
claude mcp inspect playwright

# Test connection
claude mcp test postgres
```

---

## 3. PLAYWRIGHT MCP - BROWSER AUTOMATION

### 3.1 Giới thiệu Playwright MCP

Playwright MCP cho phép Claude Code điều khiển browser để:
- Chạy E2E tests
- Screenshot và visual testing
- Web scraping
- Form automation
- Performance testing

### 3.2 Cài đặt Playwright MCP

```json
// .claude/settings.json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true",
        "PLAYWRIGHT_BROWSER": "chromium"
      }
    }
  }
}
```

### 3.3 Playwright MCP Tools

```
┌──────────────────────────────────────────────────────────────┐
│                  PLAYWRIGHT MCP TOOLS                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Navigation:                                                  │
│  ├── mcp__playwright__navigate(url)                          │
│  ├── mcp__playwright__go_back()                              │
│  ├── mcp__playwright__go_forward()                           │
│  └── mcp__playwright__reload()                               │
│                                                               │
│  Interaction:                                                 │
│  ├── mcp__playwright__click(selector)                        │
│  ├── mcp__playwright__fill(selector, value)                  │
│  ├── mcp__playwright__select(selector, value)                │
│  ├── mcp__playwright__hover(selector)                        │
│  └── mcp__playwright__press(key)                             │
│                                                               │
│  Capture:                                                     │
│  ├── mcp__playwright__screenshot(options)                    │
│  ├── mcp__playwright__pdf(options)                           │
│  └── mcp__playwright__get_content()                          │
│                                                               │
│  Evaluation:                                                  │
│  ├── mcp__playwright__evaluate(script)                       │
│  ├── mcp__playwright__wait_for_selector(selector)            │
│  └── mcp__playwright__get_attribute(selector, attr)          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 3.4 Ví dụ: E2E Test với Playwright MCP

**Tạo Subagent cho E2E Testing:**

```yaml
# .claude/agents/e2e-tester.yml
name: e2e-tester
description: "Chạy E2E tests sử dụng Playwright MCP"
model: sonnet
tools:
  - mcp__playwright__navigate
  - mcp__playwright__click
  - mcp__playwright__fill
  - mcp__playwright__screenshot
  - mcp__playwright__wait_for_selector
  - mcp__playwright__get_content
  - mcp__playwright__evaluate
  - Read
  - Write
permissionMode: acceptEdits
```

**Prompt mẫu cho E2E Testing:**

```markdown
# Chạy E2E Test cho Login Flow

## Mục tiêu
Test login flow của ứng dụng tại http://localhost:3000

## Test Cases

### TC1: Login thành công
1. Navigate đến /login
2. Điền email: test@example.com
3. Điền password: Password123
4. Click nút Login
5. Verify redirect đến /dashboard
6. Screenshot kết quả

### TC2: Login thất bại - sai password
1. Navigate đến /login
2. Điền email: test@example.com
3. Điền password: WrongPassword
4. Click nút Login
5. Verify error message hiển thị
6. Screenshot kết quả

## Output
- Tạo file test-results/login-flow.md với kết quả
- Lưu screenshots vào test-results/screenshots/
```

### 3.5 Visual Regression Testing

```yaml
# .claude/agents/visual-tester.yml
name: visual-tester
description: "Visual regression testing với screenshot comparison"
model: sonnet
tools:
  - mcp__playwright__navigate
  - mcp__playwright__screenshot
  - mcp__playwright__wait_for_selector
  - Read
  - Write
  - Bash
```

**Workflow Visual Testing:**

```
┌─────────────────────────────────────────────────────────────┐
│              VISUAL REGRESSION WORKFLOW                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Capture Baseline                                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Navigate → Wait for load → Screenshot → Save as     │   │
│  │  baseline/page-name.png                              │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  2. Capture Current                                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Navigate → Wait for load → Screenshot → Save as     │   │
│  │  current/page-name.png                               │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  3. Compare & Report                                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  pixelmatch compare → Generate diff → Report         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.6 Performance Testing với Playwright

```typescript
// scripts/perf-test.ts
// Claude Code có thể chạy và phân tích kết quả

import { chromium } from 'playwright';

async function measurePerformance(url: string) {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  // Enable performance metrics
  await page.goto(url);

  const metrics = await page.evaluate(() => ({
    // Core Web Vitals
    LCP: performance.getEntriesByType('largest-contentful-paint')[0]?.startTime,
    FID: performance.getEntriesByType('first-input')[0]?.processingStart,
    CLS: performance.getEntriesByType('layout-shift')
      .reduce((sum, entry) => sum + entry.value, 0),

    // Other metrics
    TTFB: performance.timing.responseStart - performance.timing.requestStart,
    DOMContentLoaded: performance.timing.domContentLoadedEventEnd -
                      performance.timing.navigationStart,
    FullLoad: performance.timing.loadEventEnd -
              performance.timing.navigationStart,
  }));

  await browser.close();
  return metrics;
}
```

---

## 4. DEVTOOLS MCP - DEBUGGING

### 4.1 DevTools MCP Overview

DevTools MCP kết nối Claude Code với Chrome DevTools Protocol, cho phép:
- Real-time debugging
- Performance profiling
- Network monitoring
- Console log access
- Memory analysis

### 4.2 Cài đặt DevTools MCP

```json
// .claude/settings.json
{
  "mcpServers": {
    "devtools": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-devtools"],
      "env": {
        "CHROME_DEBUG_PORT": "9222"
      }
    }
  }
}
```

**Khởi động Chrome với Debug Port:**

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222

# Windows
chrome.exe --remote-debugging-port=9222
```

### 4.3 DevTools MCP Tools

```
┌──────────────────────────────────────────────────────────────┐
│                   DEVTOOLS MCP TOOLS                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Console:                                                     │
│  ├── mcp__devtools__get_console_logs()                       │
│  ├── mcp__devtools__clear_console()                          │
│  └── mcp__devtools__evaluate(expression)                     │
│                                                               │
│  Network:                                                     │
│  ├── mcp__devtools__get_network_requests()                   │
│  ├── mcp__devtools__get_request_details(requestId)           │
│  └── mcp__devtools__block_url(pattern)                       │
│                                                               │
│  Performance:                                                 │
│  ├── mcp__devtools__start_profiling()                        │
│  ├── mcp__devtools__stop_profiling()                         │
│  └── mcp__devtools__get_metrics()                            │
│                                                               │
│  DOM:                                                         │
│  ├── mcp__devtools__get_dom_tree()                           │
│  ├── mcp__devtools__query_selector(selector)                 │
│  └── mcp__devtools__get_computed_styles(nodeId)              │
│                                                               │
│  Memory:                                                      │
│  ├── mcp__devtools__take_heap_snapshot()                     │
│  └── mcp__devtools__get_memory_usage()                       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 4.4 Debugging Subagent

```yaml
# .claude/agents/debugger.yml
name: debugger
description: "Debug application issues sử dụng DevTools MCP"
model: opus
tools:
  - mcp__devtools__get_console_logs
  - mcp__devtools__get_network_requests
  - mcp__devtools__evaluate
  - mcp__devtools__get_metrics
  - Read
  - Grep
```

**Prompt mẫu cho Debugging:**

```markdown
# Debug Performance Issue

## Vấn đề
Trang /products load chậm, mất hơn 5 giây

## Yêu cầu
1. Thu thập console logs để tìm errors/warnings
2. Phân tích network requests:
   - Requests nào mất nhiều thời gian nhất?
   - Có requests nào failed không?
   - Có nhiều requests redundant không?
3. Đo performance metrics (LCP, FID, CLS)
4. Đề xuất optimizations dựa trên findings

## Output
Tạo báo cáo chi tiết với:
- Root cause analysis
- Network waterfall analysis
- Recommended fixes với priority
```

### 4.5 Real-time Error Monitoring

```yaml
# .claude/agents/error-monitor.yml
name: error-monitor
description: "Monitor và phân tích runtime errors"
model: haiku
tools:
  - mcp__devtools__get_console_logs
  - mcp__devtools__evaluate
  - Read
  - Write
```

**Error Monitoring Workflow:**

```
┌─────────────────────────────────────────────────────────────┐
│                ERROR MONITORING FLOW                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                           │
│  │ Get Console  │                                           │
│  │    Logs      │                                           │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐     ┌──────────────┐                      │
│  │ Filter for   │────►│ Parse Stack  │                      │
│  │   Errors     │     │    Traces    │                      │
│  └──────────────┘     └──────┬───────┘                      │
│                              │                               │
│         ┌────────────────────┴────────────────────┐         │
│         ▼                                         ▼         │
│  ┌──────────────┐                         ┌──────────────┐  │
│  │ Read Source  │                         │   Identify   │  │
│  │    Files     │                         │  Root Cause  │  │
│  └──────┬───────┘                         └──────┬───────┘  │
│         │                                        │          │
│         └────────────────────┬───────────────────┘          │
│                              ▼                               │
│                       ┌──────────────┐                      │
│                       │ Generate Fix │                      │
│                       │  Suggestion  │                      │
│                       └──────────────┘                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. TESTING AUTOMATION VỚI CLAUDE CODE

### 5.1 Testing Strategy Overview

```
┌──────────────────────────────────────────────────────────────┐
│                   TESTING PYRAMID                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│                        ▲                                      │
│                       /│\        E2E Tests                    │
│                      / │ \       (Playwright MCP)             │
│                     /  │  \      Slow, Expensive              │
│                    /───┼───\                                  │
│                   /    │    \    Integration Tests            │
│                  /     │     \   (API, DB)                    │
│                 /──────┼──────\                               │
│                /       │       \ Unit Tests                   │
│               /        │        \ (Vitest, Jest)              │
│              /─────────┼─────────\ Fast, Cheap                │
│                                                               │
│  Claude Code có thể tự động hóa tất cả các tầng này          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 Unit Testing Subagent

```yaml
# .claude/agents/unit-tester.yml
name: unit-tester
description: "Viết và chạy unit tests"
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
permissionMode: acceptEdits
```

**Prompt Template:**

```markdown
# Generate Unit Tests

## Target File
src/utils/validation.ts

## Requirements
1. Đọc source file và hiểu logic
2. Tạo test file tương ứng: src/utils/validation.test.ts
3. Cover các cases:
   - Happy path
   - Edge cases
   - Error cases
   - Boundary conditions
4. Sử dụng Vitest syntax
5. Mock external dependencies nếu cần
6. Target coverage: 80%+

## Chạy tests
Sau khi viết xong, chạy: pnpm test src/utils/validation.test.ts
```

### 5.3 Integration Testing

```yaml
# .claude/agents/integration-tester.yml
name: integration-tester
description: "Viết và chạy integration tests cho API endpoints"
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - mcp__playwright__navigate
  - mcp__playwright__get_content
```

**API Integration Test Example:**

```typescript
// tests/integration/api/users.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { createTestServer, closeTestServer } from '../helpers/server';
import { createTestDatabase, cleanupTestDatabase } from '../helpers/db';

describe('Users API', () => {
  let server: TestServer;
  let db: TestDatabase;

  beforeAll(async () => {
    db = await createTestDatabase();
    server = await createTestServer({ database: db });
  });

  afterAll(async () => {
    await closeTestServer(server);
    await cleanupTestDatabase(db);
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const response = await fetch(`${server.url}/api/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: 'test@example.com',
          name: 'Test User',
        }),
      });

      expect(response.status).toBe(201);
      const user = await response.json();
      expect(user).toMatchObject({
        email: 'test@example.com',
        name: 'Test User',
      });
    });

    it('should return 400 for invalid email', async () => {
      const response = await fetch(`${server.url}/api/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: 'invalid-email',
          name: 'Test User',
        }),
      });

      expect(response.status).toBe(400);
      const error = await response.json();
      expect(error.message).toContain('email');
    });
  });
});
```

### 5.4 Test Generation Hook

```javascript
// .claude/hooks/auto-test.js
// Tự động suggest tests khi code mới được viết

export default {
  event: 'Stop',
  hooks: [
    {
      name: 'suggest-tests',
      script: async (context) => {
        const { toolResults } = context;

        // Tìm các file vừa được tạo/sửa
        const modifiedFiles = toolResults
          .filter(r => r.tool === 'Write' || r.tool === 'Edit')
          .map(r => r.params.file_path)
          .filter(f => f.endsWith('.ts') && !f.includes('.test.'));

        if (modifiedFiles.length > 0) {
          return {
            note: `Files modified: ${modifiedFiles.join(', ')}. Consider running unit-tester agent to generate tests.`
          };
        }
      }
    }
  ]
};
```

### 5.5 Test Coverage Monitoring

```yaml
# .claude/agents/coverage-analyzer.yml
name: coverage-analyzer
description: "Phân tích test coverage và đề xuất improvements"
model: sonnet
tools:
  - Bash
  - Read
  - Glob
  - Grep
```

**Coverage Analysis Workflow:**

```
┌─────────────────────────────────────────────────────────────┐
│              COVERAGE ANALYSIS WORKFLOW                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Run Coverage Report                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  pnpm test --coverage --reporter=json                │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  2. Parse Coverage Data                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Read coverage/coverage-summary.json                 │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  3. Identify Low Coverage Files                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Filter files with < 80% coverage                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  4. Analyze Uncovered Lines                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Read source files, identify uncovered branches      │   │
│  └──────────────────────────────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  5. Generate Test Suggestions                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Create specific test cases for uncovered code       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. CI/CD INTEGRATION

### 6.1 GitHub Actions với Claude Code

```yaml
# .github/workflows/claude-code-tests.yml
name: Claude Code Test Suite

on:
  pull_request:
    branches: [main, develop]

jobs:
  claude-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Install Playwright browsers
        run: pnpm exec playwright install --with-deps chromium

      - name: Run Claude Code Tests
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # Chạy unit tests
          pnpm test --coverage

          # Chạy E2E tests với Playwright
          pnpm exec playwright test

          # Chạy Claude Code validation
          claude --print "Verify all tests pass and coverage is adequate"

      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
```

### 6.2 Pre-commit Testing Hook

```javascript
// .claude/hooks/pre-commit-tests.js
export default {
  event: 'PreToolUse',
  hooks: [
    {
      name: 'pre-commit-test-reminder',
      script: async (context) => {
        const { tool, params } = context;

        if (tool === 'Bash' && params.command.includes('git commit')) {
          // Kiểm tra xem tests đã chạy chưa
          const testRan = context.conversation.some(
            msg => msg.content?.includes('pnpm test') ||
                   msg.content?.includes('npm test')
          );

          if (!testRan) {
            return {
              note: 'Warning: No tests were run in this session. Consider running tests before committing.'
            };
          }
        }
      }
    }
  ]
};
```

### 6.3 Automated PR Review

```yaml
# .claude/agents/pr-reviewer.yml
name: pr-reviewer
description: "Review pull requests tự động"
model: opus
tools:
  - Bash
  - Read
  - Glob
  - Grep
```

**PR Review Workflow:**

```markdown
# Review Pull Request #123

## Checklist
1. Đọc PR description và linked issues
2. Review từng file changed:
   - Code quality và style
   - Potential bugs
   - Security vulnerabilities
   - Performance implications
3. Kiểm tra test coverage cho code mới
4. Verify CI/CD status
5. Đề xuất improvements nếu cần

## Output Format
- **Summary**: Tóm tắt changes
- **Concerns**: Các vấn đề cần address
- **Suggestions**: Đề xuất cải thiện
- **Verdict**: Approve / Request Changes / Comment
```

---

## 7. BEST PRACTICES & PATTERNS

### 7.1 MCP Security Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│                 MCP SECURITY CHECKLIST                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  □ Sử dụng environment variables cho sensitive data          │
│    ✗ "API_KEY": "sk-123abc..."                               │
│    ✓ "API_KEY": "${API_KEY}"                                 │
│                                                               │
│  □ Giới hạn MCP tools trong subagent definitions             │
│    tools:                                                     │
│      - mcp__playwright__navigate  # Chỉ tools cần thiết      │
│      - mcp__playwright__screenshot                           │
│                                                               │
│  □ Sử dụng project-level config thay vì global               │
│    .claude/settings.json > ~/.claude/settings.json           │
│                                                               │
│  □ Review MCP server source trước khi sử dụng                │
│    Chỉ dùng official hoặc trusted MCP servers                │
│                                                               │
│  □ Enable logging cho audit trail                            │
│    "PLAYWRIGHT_DEBUG": "true"                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 7.2 Testing Patterns

**Pattern 1: Arrange-Act-Assert**

```typescript
it('should validate user email', () => {
  // Arrange
  const validator = new EmailValidator();
  const email = 'test@example.com';

  // Act
  const result = validator.validate(email);

  // Assert
  expect(result.isValid).toBe(true);
});
```

**Pattern 2: Given-When-Then (BDD)**

```typescript
describe('User Registration', () => {
  describe('given valid user data', () => {
    const userData = { email: 'test@example.com', password: 'Secure123!' };

    describe('when registering', () => {
      it('then should create user successfully', async () => {
        const result = await registerUser(userData);
        expect(result.success).toBe(true);
      });
    });
  });
});
```

**Pattern 3: Test Fixtures**

```typescript
// tests/fixtures/users.ts
export const validUser = {
  email: 'valid@example.com',
  password: 'ValidPass123!',
  name: 'Valid User',
};

export const invalidUser = {
  email: 'invalid-email',
  password: '123', // too short
  name: '',
};

// Usage in tests
import { validUser, invalidUser } from '../fixtures/users';
```

### 7.3 MCP Error Handling

```typescript
// Robust MCP tool usage pattern
async function safePlaywrightNavigate(url: string) {
  try {
    const result = await mcp__playwright__navigate({ url });
    return { success: true, data: result };
  } catch (error) {
    if (error.code === 'TIMEOUT') {
      // Retry once với longer timeout
      return await mcp__playwright__navigate({
        url,
        timeout: 60000
      });
    }
    if (error.code === 'CONNECTION_REFUSED') {
      throw new Error('Browser not running. Start Chrome with --remote-debugging-port=9222');
    }
    throw error;
  }
}
```

### 7.4 Testing Documentation Template

```markdown
# Test Documentation: [Feature Name]

## Overview
Brief description of what is being tested.

## Test Environment
- Node.js: v20.x
- Framework: Vitest
- MCP Servers: Playwright, DevTools

## Test Categories

### Unit Tests
| Test File | Coverage | Status |
|-----------|----------|--------|
| validation.test.ts | 95% | ✅ |
| utils.test.ts | 88% | ✅ |

### Integration Tests
| Test Suite | Endpoints | Status |
|------------|-----------|--------|
| Users API | /api/users/* | ✅ |
| Auth API | /api/auth/* | ✅ |

### E2E Tests
| Flow | Steps | Status |
|------|-------|--------|
| Login | 5 | ✅ |
| Checkout | 8 | ⚠️ Flaky |

## Known Issues
- E2E checkout test occasionally fails due to payment gateway timeout

## Running Tests
```bash
# Unit tests
pnpm test

# Integration tests
pnpm test:integration

# E2E tests
pnpm test:e2e

# All tests with coverage
pnpm test:all --coverage
```
```

---

## TỔNG KẾT PHẦN 3

### Key Takeaways

1. **MCP mở rộng capabilities** của Claude Code vượt xa built-in tools
2. **Playwright MCP** là công cụ mạnh cho browser automation và E2E testing
3. **DevTools MCP** cho phép debugging real-time và performance profiling
4. **Testing automation** với Claude Code covers toàn bộ testing pyramid
5. **CI/CD integration** đảm bảo quality gates trong development workflow

### Checklist Áp Dụng

- [ ] Cấu hình Playwright MCP trong project
- [ ] Tạo E2E testing subagent
- [ ] Setup DevTools MCP cho debugging
- [ ] Tích hợp testing vào CI/CD pipeline
- [ ] Thiết lập coverage monitoring
- [ ] Áp dụng testing patterns phù hợp

### Phần Tiếp Theo

**Part 4: Advanced Patterns & Hands-on Labs** sẽ cover:
- Multi-agent orchestration patterns
- Complex workflow automation
- Hands-on labs với real-world scenarios
- Performance optimization techniques
