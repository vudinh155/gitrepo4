# PHẦN 4: ADVANCED PATTERNS & HANDS-ON LABS

## Tài liệu dành cho: Frontend, Backend, QA, DevOps Teams

---

## MỤC LỤC

1. [Multi-Agent Orchestration](#1-multi-agent-orchestration)
2. [Complex Workflow Automation](#2-complex-workflow-automation)
3. [Performance Optimization](#3-performance-optimization)
4. [Error Handling & Recovery](#4-error-handling--recovery)
5. [Hands-on Lab 1: Full-Stack Feature Development](#5-hands-on-lab-1-full-stack-feature-development)
6. [Hands-on Lab 2: Legacy Code Refactoring](#6-hands-on-lab-2-legacy-code-refactoring)
7. [Hands-on Lab 3: CI/CD Pipeline Setup](#7-hands-on-lab-3-cicd-pipeline-setup)

---

## 1. MULTI-AGENT ORCHESTRATION

### 1.1 Agent Hierarchy Patterns

```
┌──────────────────────────────────────────────────────────────┐
│                AGENT HIERARCHY PATTERNS                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Pattern 1: Sequential Pipeline                               │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐          │
│  │ Agent  │──►│ Agent  │──►│ Agent  │──►│ Agent  │          │
│  │   A    │   │   B    │   │   C    │   │   D    │          │
│  └────────┘   └────────┘   └────────┘   └────────┘          │
│                                                               │
│  Pattern 2: Parallel Fan-out                                  │
│                    ┌────────┐                                │
│                 ┌─►│ Agent  │                                │
│  ┌────────┐    │  │   B    │                                │
│  │ Agent  │────┼─►├────────┤                                │
│  │   A    │    │  │ Agent  │                                │
│  └────────┘    │  │   C    │                                │
│                └─►├────────┤                                │
│                   │ Agent  │                                │
│                   │   D    │                                │
│                   └────────┘                                │
│                                                               │
│  Pattern 3: Hierarchical Delegation                          │
│                   ┌────────┐                                 │
│                   │Orchestr│                                 │
│                   │  ator  │                                 │
│                   └───┬────┘                                 │
│              ┌────────┼────────┐                             │
│              ▼        ▼        ▼                             │
│         ┌────────┐┌────────┐┌────────┐                      │
│         │Frontend││Backend ││  QA    │                      │
│         │ Agent  ││ Agent  ││ Agent  │                      │
│         └───┬────┘└───┬────┘└───┬────┘                      │
│             │         │         │                            │
│         ┌───┴───┐ ┌───┴───┐ ┌───┴───┐                       │
│         ▼       ▼ ▼       ▼ ▼       ▼                       │
│       ┌───┐ ┌───┐┌───┐ ┌───┐┌───┐ ┌───┐                    │
│       │UI │ │API││DB │ │API││Unit│ │E2E│                    │
│       └───┘ └───┘└───┘ └───┘└───┘ └───┘                    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 Orchestrator Agent Pattern

```yaml
# .claude/agents/orchestrator.yml
name: orchestrator
description: "Điều phối các agents khác để hoàn thành complex tasks"
model: opus
tools:
  - Task
  - Read
  - Glob
  - TodoWrite
permissionMode: default
```

**Orchestrator Workflow:**

```markdown
# Feature Implementation Orchestrator

## Input
Feature request: "Add user profile editing functionality"

## Orchestration Steps

### Step 1: Analysis Phase
Spawn: code-analyzer agent
Task: Analyze existing user-related code and identify integration points

### Step 2: Parallel Development
Spawn simultaneously:
- frontend-agent: Create ProfileEdit component
- backend-agent: Create /api/users/profile endpoint

### Step 3: Integration
Wait for Step 2 completion
Spawn: integration-tester agent
Task: Verify frontend-backend integration

### Step 4: Quality Assurance
Spawn: qa-agent
Task: Run full test suite and report coverage

### Step 5: Documentation
Spawn: docs-agent
Task: Update API documentation and component storybook
```

### 1.3 Agent Communication Protocol

```
┌──────────────────────────────────────────────────────────────┐
│              AGENT COMMUNICATION FLOW                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐                                             │
│  │ Orchestrator│                                             │
│  │    Agent    │                                             │
│  └──────┬──────┘                                             │
│         │                                                     │
│         │ 1. Spawn with detailed prompt                      │
│         ▼                                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                    Task Tool                          │    │
│  │  prompt: "Analyze src/users/ for profile endpoints"   │    │
│  │  subagent_type: "Explore"                            │    │
│  └──────────────────────────────────────────────────────┘    │
│         │                                                     │
│         │ 2. Agent executes autonomously                     │
│         ▼                                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                  Child Agent                          │    │
│  │  - Reads files                                        │    │
│  │  - Searches codebase                                  │    │
│  │  - Analyzes patterns                                  │    │
│  └──────────────────────────────────────────────────────┘    │
│         │                                                     │
│         │ 3. Return structured result                        │
│         ▼                                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                 Result Object                         │    │
│  │  {                                                    │    │
│  │    "files_found": [...],                             │    │
│  │    "endpoints": [...],                               │    │
│  │    "recommendations": [...]                          │    │
│  │  }                                                    │    │
│  └──────────────────────────────────────────────────────┘    │
│         │                                                     │
│         │ 4. Orchestrator processes and continues            │
│         ▼                                                     │
│  ┌─────────────┐                                             │
│  │ Orchestrator│  Next step based on result                 │
│  └─────────────┘                                             │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 1.4 Parallel Agent Execution

```yaml
# .claude/agents/parallel-executor.yml
name: parallel-executor
description: "Thực thi nhiều agents song song"
model: sonnet
tools:
  - Task
  - TodoWrite
```

**Parallel Execution Example:**

```markdown
# Parallel Code Review

## Task
Review PR #456 covering multiple areas

## Parallel Agents

### Agent 1: Security Review
```yaml
subagent_type: general-purpose
prompt: |
  Review PR #456 for security issues:
  - SQL injection
  - XSS vulnerabilities
  - Authentication bypass
  - Sensitive data exposure
```

### Agent 2: Performance Review
```yaml
subagent_type: general-purpose
prompt: |
  Review PR #456 for performance:
  - N+1 queries
  - Memory leaks
  - Inefficient algorithms
  - Bundle size impact
```

### Agent 3: Code Quality Review
```yaml
subagent_type: general-purpose
prompt: |
  Review PR #456 for code quality:
  - Code style consistency
  - DRY violations
  - Complex functions
  - Missing tests
```

## Aggregation
Combine results from all 3 agents into comprehensive review
```

---

## 2. COMPLEX WORKFLOW AUTOMATION

### 2.1 Feature Development Workflow

```
┌──────────────────────────────────────────────────────────────┐
│            FEATURE DEVELOPMENT WORKFLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                   1. PLANNING                        │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐       │     │
│  │  │  Analyze  │─►│  Design   │─►│  Create   │       │     │
│  │  │ Requirements│ │ Solution │  │   Todos   │       │     │
│  │  └───────────┘  └───────────┘  └───────────┘       │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                2. IMPLEMENTATION                     │     │
│  │                                                      │     │
│  │  ┌─────────────────────────────────────────────┐    │     │
│  │  │              Parallel Work                   │    │     │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐     │    │     │
│  │  │  │Frontend │  │ Backend │  │Database │     │    │     │
│  │  │  │  Code   │  │   API   │  │ Schema  │     │    │     │
│  │  │  └─────────┘  └─────────┘  └─────────┘     │    │     │
│  │  └─────────────────────────────────────────────┘    │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                  3. TESTING                          │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐       │     │
│  │  │   Unit    │─►│Integration│─►│    E2E    │       │     │
│  │  │   Tests   │  │   Tests   │  │   Tests   │       │     │
│  │  └───────────┘  └───────────┘  └───────────┘       │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                  4. REVIEW                           │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐       │     │
│  │  │  Code     │─►│  Create   │─►│   Merge   │       │     │
│  │  │  Review   │  │    PR     │  │           │       │     │
│  │  └───────────┘  └───────────┘  └───────────┘       │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Automated Bug Fix Workflow

```yaml
# .claude/agents/bug-fixer.yml
name: bug-fixer
description: "End-to-end bug fixing automation"
model: opus
tools:
  - Read
  - Edit
  - Write
  - Bash
  - Grep
  - Glob
  - mcp__devtools__get_console_logs
  - mcp__devtools__get_network_requests
permissionMode: acceptEdits
```

**Bug Fix Workflow:**

```markdown
# Automated Bug Fix: Issue #789

## Phase 1: Reproduce
1. Read issue description và reproduction steps
2. Setup local environment
3. Reproduce bug và capture:
   - Console errors
   - Network failures
   - Screenshots

## Phase 2: Diagnose
1. Analyze error stack traces
2. Search codebase for related code
3. Identify root cause
4. Document findings

## Phase 3: Fix
1. Create fix branch
2. Implement solution
3. Add regression test
4. Verify fix locally

## Phase 4: Validate
1. Run full test suite
2. Check for regressions
3. Performance impact check
4. Create PR with detailed description
```

### 2.3 Database Migration Workflow

```
┌──────────────────────────────────────────────────────────────┐
│              DATABASE MIGRATION WORKFLOW                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. ANALYZE                                                   │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Current schema analysis                            │    │
│  │  - Identify breaking changes                          │    │
│  │  - Data migration requirements                        │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  2. GENERATE MIGRATION                                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Create migration file                              │    │
│  │  - Add rollback logic                                 │    │
│  │  - Data transformation scripts                        │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  3. TEST MIGRATION                                            │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Backup test database                               │    │
│  │  - Run migration on test DB                           │    │
│  │  - Verify data integrity                              │    │
│  │  - Test rollback                                      │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  4. UPDATE APPLICATION                                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Update TypeScript types                            │    │
│  │  - Update Prisma schema                               │    │
│  │  - Update API endpoints                               │    │
│  │  - Update frontend forms                              │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  5. DEPLOY                                                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Create PR with all changes                         │    │
│  │  - Document migration steps                           │    │
│  │  - Coordinate deployment timing                       │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 2.4 Monorepo Dependency Update

```yaml
# .claude/agents/dep-updater.yml
name: dep-updater
description: "Update dependencies across monorepo packages"
model: sonnet
tools:
  - Bash
  - Read
  - Edit
  - Glob
  - Grep
```

**Dependency Update Script:**

```markdown
# Update React to v19 across Monorepo

## Scope
Packages: apps/web, apps/admin, packages/ui, packages/shared

## Steps

### 1. Inventory
- List all packages using React
- Document current versions
- Identify breaking changes from v18 to v19

### 2. Update Root
```bash
pnpm add react@19 react-dom@19 -w
pnpm add @types/react@19 @types/react-dom@19 -wD
```

### 3. Update Each Package
For each package:
1. Update package.json
2. Run type check
3. Fix type errors
4. Run tests
5. Document changes

### 4. Validate
- Full test suite
- Build all packages
- E2E tests
- Visual regression tests

### 5. Create PR
- Comprehensive changelog
- Migration notes
- Performance benchmarks
```

---

## 3. PERFORMANCE OPTIMIZATION

### 3.1 Agent Performance Tuning

```
┌──────────────────────────────────────────────────────────────┐
│              AGENT PERFORMANCE MATRIX                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Model Selection:                                             │
│  ┌────────────┬──────────┬───────────┬──────────────┐        │
│  │   Model    │  Speed   │   Cost    │   Use Case   │        │
│  ├────────────┼──────────┼───────────┼──────────────┤        │
│  │   Haiku    │  Fast    │   Low     │ Simple tasks │        │
│  │   Sonnet   │ Medium   │  Medium   │ Most tasks   │        │
│  │   Opus     │  Slow    │   High    │ Complex/QA   │        │
│  └────────────┴──────────┴───────────┴──────────────┘        │
│                                                               │
│  Tool Optimization:                                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  ✓ Limit tools to only what's needed                 │    │
│  │  ✓ Use specific Glob patterns over broad searches    │    │
│  │  ✓ Prefer Grep with file type filters               │    │
│  │  ✓ Read specific line ranges for large files        │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
│  Context Optimization:                                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  ✓ Provide focused, relevant context                 │    │
│  │  ✓ Avoid dumping entire files                        │    │
│  │  ✓ Use structured prompts                            │    │
│  │  ✓ Clear success criteria                            │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 3.2 Caching Strategies

```yaml
# .claude/rules/caching-patterns.md
---
scope: project
---

# Caching Patterns

## Analysis Cache
Khi phân tích codebase:
1. Lưu kết quả vào .claude/cache/analysis.json
2. Check cache trước khi re-analyze
3. Invalidate khi files thay đổi

## Test Results Cache
Khi chạy tests:
1. Cache test results với file hash
2. Only re-run tests for changed files
3. Store in .claude/cache/test-results.json

## Dependency Graph Cache
1. Build dependency graph once
2. Update incrementally on file changes
3. Store in .claude/cache/deps.json
```

### 3.3 Batch Operations

```typescript
// Pattern: Batch file operations
// Thay vì:
for (const file of files) {
  await read(file);
  await edit(file, changes);
}

// Sử dụng:
const contents = await Promise.all(files.map(f => read(f)));
// Analyze all at once
const changes = analyzeAll(contents);
// Apply changes in batch
await batchEdit(files, changes);
```

### 3.4 Incremental Processing

```
┌──────────────────────────────────────────────────────────────┐
│              INCREMENTAL PROCESSING PATTERN                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Full Processing (Slow):                                      │
│  ┌────────────────────────────────────────────────────┐      │
│  │  All Files ────► Process All ────► Complete Result │      │
│  └────────────────────────────────────────────────────┘      │
│                                                               │
│  Incremental Processing (Fast):                               │
│  ┌────────────────────────────────────────────────────┐      │
│  │                                                     │      │
│  │  ┌──────────┐                                      │      │
│  │  │ Changed  │──► Process ──┐                       │      │
│  │  │  Files   │              │                       │      │
│  │  └──────────┘              ▼                       │      │
│  │                     ┌─────────────┐                │      │
│  │  ┌──────────┐       │   Merge     │                │      │
│  │  │ Cached   │──────►│   Results   │───► Output     │      │
│  │  │ Results  │       └─────────────┘                │      │
│  │  └──────────┘                                      │      │
│  │                                                     │      │
│  └────────────────────────────────────────────────────┘      │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. ERROR HANDLING & RECOVERY

### 4.1 Error Classification

```
┌──────────────────────────────────────────────────────────────┐
│                  ERROR CLASSIFICATION                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Recoverable Errors:                                          │
│  ├── Network timeout → Retry with backoff                    │
│  ├── File not found → Search for alternatives               │
│  ├── Syntax error → Auto-fix attempt                        │
│  └── Test failure → Analyze and fix                         │
│                                                               │
│  Non-Recoverable Errors:                                      │
│  ├── Permission denied → Report to user                     │
│  ├── Invalid credentials → Request new credentials          │
│  ├── Disk full → Cannot proceed                             │
│  └── Critical system error → Abort safely                   │
│                                                               │
│  Partially Recoverable:                                       │
│  ├── Partial test failure → Continue with warnings          │
│  ├── Deprecated API → Use alternative + warn                │
│  └── Memory limit → Reduce batch size                       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Retry Strategies

```javascript
// .claude/hooks/retry-logic.js
export default {
  event: 'SubagentStop',
  hooks: [
    {
      name: 'auto-retry',
      script: async (context) => {
        const { result, retryCount = 0 } = context;

        // Check if failed and retryable
        if (result.error && isRetryable(result.error) && retryCount < 3) {
          const backoff = Math.pow(2, retryCount) * 1000; // 1s, 2s, 4s

          return {
            action: 'retry',
            delay: backoff,
            context: { retryCount: retryCount + 1 }
          };
        }

        return { action: 'continue' };
      }
    }
  ]
};

function isRetryable(error) {
  const retryablePatterns = [
    /timeout/i,
    /ECONNRESET/,
    /ENOTFOUND/,
    /rate limit/i,
    /503/,
    /429/
  ];
  return retryablePatterns.some(p => p.test(error.message));
}
```

### 4.3 Checkpoint & Rollback

```
┌──────────────────────────────────────────────────────────────┐
│              CHECKPOINT & ROLLBACK FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Normal Flow:                                                 │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐             │
│  │ Start  │─►│Checkpoint│─►│  Work  │─►│Complete│             │
│  └────────┘  │   #1    │  └────────┘  └────────┘             │
│              └────────┘       │                               │
│                               ▼                               │
│                          ┌────────┐  ┌────────┐              │
│                          │Checkpoint│─►│  Work  │─►...        │
│                          │   #2    │  └────────┘              │
│                          └────────┘                           │
│                                                               │
│  Error Recovery:                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐             │
│  │ Start  │─►│   CP   │─►│  Work  │─►│ ERROR! │             │
│  └────────┘  │   #1   │  └────────┘  └───┬────┘             │
│              └───▲────┘                   │                   │
│                  │                        │                   │
│                  └────────────────────────┘                   │
│                      Rollback to CP #1                        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 4.4 Graceful Degradation

```yaml
# .claude/agents/resilient-worker.yml
name: resilient-worker
description: "Worker với graceful degradation"
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob

# Fallback strategies
fallbacks:
  - condition: "mcp_unavailable"
    action: "use_builtin_tools"
  - condition: "test_timeout"
    action: "skip_slow_tests"
  - condition: "build_failure"
    action: "partial_build"
```

---

## 5. HANDS-ON LAB 1: FULL-STACK FEATURE DEVELOPMENT

### 5.1 Lab Overview

**Objective**: Implement a complete user notification system

**Duration**: 2-3 hours

**Prerequisites**:
- Next.js project setup
- PostgreSQL database
- Basic understanding of Claude Code agents

### 5.2 Lab Structure

```
┌──────────────────────────────────────────────────────────────┐
│           LAB 1: NOTIFICATION SYSTEM                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Phase 1: Database (30 min)                                   │
│  ├── Create notifications table schema                       │
│  ├── Create Prisma model                                     │
│  └── Run migration                                           │
│                                                               │
│  Phase 2: Backend API (45 min)                                │
│  ├── GET /api/notifications                                  │
│  ├── POST /api/notifications                                 │
│  ├── PATCH /api/notifications/:id/read                       │
│  └── DELETE /api/notifications/:id                           │
│                                                               │
│  Phase 3: Frontend (45 min)                                   │
│  ├── NotificationBell component                              │
│  ├── NotificationList component                              │
│  ├── NotificationItem component                              │
│  └── useNotifications hook                                   │
│                                                               │
│  Phase 4: Real-time (30 min)                                  │
│  ├── WebSocket integration                                   │
│  └── Live notification updates                               │
│                                                               │
│  Phase 5: Testing (30 min)                                    │
│  ├── Unit tests                                              │
│  ├── Integration tests                                       │
│  └── E2E test                                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 5.3 Phase 1: Database Setup

**Agent Configuration:**

```yaml
# .claude/agents/db-architect.yml
name: db-architect
description: "Design and implement database schemas"
model: sonnet
tools:
  - Read
  - Write
  - Bash
  - Glob
permissionMode: acceptEdits
```

**Prompt:**

```markdown
# Create Notification System Database

## Requirements
1. Tạo Prisma schema cho notifications table với fields:
   - id: UUID
   - userId: Foreign key to User
   - type: Enum (INFO, WARNING, ERROR, SUCCESS)
   - title: String
   - message: String
   - isRead: Boolean (default false)
   - createdAt: DateTime
   - readAt: DateTime (nullable)

2. Tạo migration file
3. Apply migration
4. Generate Prisma client

## Files to create/modify
- prisma/schema.prisma
- Run: pnpm prisma migrate dev --name add_notifications
```

**Expected Output:**

```prisma
// prisma/schema.prisma
enum NotificationType {
  INFO
  WARNING
  ERROR
  SUCCESS
}

model Notification {
  id        String           @id @default(uuid())
  userId    String
  user      User             @relation(fields: [userId], references: [id])
  type      NotificationType @default(INFO)
  title     String
  message   String
  isRead    Boolean          @default(false)
  createdAt DateTime         @default(now())
  readAt    DateTime?

  @@index([userId, isRead])
  @@index([createdAt])
}
```

### 5.4 Phase 2: Backend API

**Agent Configuration:**

```yaml
# .claude/agents/api-builder.yml
name: api-builder
description: "Build REST API endpoints"
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

**Prompt:**

```markdown
# Create Notification API Endpoints

## Endpoints

### GET /api/notifications
- Query params: limit, offset, unreadOnly
- Return: { notifications: [], total: number, unreadCount: number }
- Auth: Required

### POST /api/notifications
- Body: { userId, type, title, message }
- Return: Created notification
- Auth: Admin only

### PATCH /api/notifications/:id/read
- Mark notification as read
- Set readAt timestamp
- Auth: Owner only

### DELETE /api/notifications/:id
- Soft delete or hard delete
- Auth: Owner only

## Files to create
- src/app/api/notifications/route.ts
- src/app/api/notifications/[id]/route.ts
- src/app/api/notifications/[id]/read/route.ts
- src/lib/services/notification.service.ts

## Validation
- Use zod for request validation
- Proper error handling
- TypeScript types
```

### 5.5 Phase 3: Frontend Components

**Prompt:**

```markdown
# Create Notification UI Components

## Components

### NotificationBell (src/components/notifications/NotificationBell.tsx)
- Shows bell icon with unread badge
- Click opens NotificationList dropdown
- Polling every 30s for new notifications

### NotificationList (src/components/notifications/NotificationList.tsx)
- Dropdown list of notifications
- "Mark all as read" button
- Infinite scroll
- Empty state

### NotificationItem (src/components/notifications/NotificationItem.tsx)
- Icon based on type
- Title, message, time ago
- Click to mark as read
- Delete button

### useNotifications hook (src/hooks/useNotifications.ts)
- Fetch notifications
- Mark as read mutation
- Delete mutation
- Real-time updates support

## Tech Stack
- React Query for data fetching
- Tailwind CSS for styling
- Lucide icons
- date-fns for time formatting
```

### 5.6 Lab Completion Checklist

```
□ Database
  □ Prisma schema created
  □ Migration applied successfully
  □ Prisma client generated

□ Backend API
  □ GET endpoint returns paginated notifications
  □ POST endpoint creates notification
  □ PATCH endpoint marks as read
  □ DELETE endpoint removes notification
  □ All endpoints have proper auth

□ Frontend
  □ NotificationBell shows unread count
  □ NotificationList displays notifications
  □ NotificationItem handles interactions
  □ Real-time updates working

□ Testing
  □ Unit tests for service layer
  □ API integration tests
  □ Component tests with RTL
  □ E2E test for notification flow

□ Documentation
  □ API documentation updated
  □ Component storybook stories
```

---

## 6. HANDS-ON LAB 2: LEGACY CODE REFACTORING

### 6.1 Lab Overview

**Objective**: Refactor legacy JavaScript module to modern TypeScript

**Duration**: 2 hours

**Scenario**: Bạn có một module xử lý payments được viết bằng JavaScript cũ, cần refactor sang TypeScript với modern patterns.

### 6.2 Legacy Code Sample

```javascript
// src/legacy/payment-processor.js (BEFORE)
var PaymentProcessor = function(config) {
  this.apiKey = config.apiKey;
  this.sandbox = config.sandbox || false;
  this.retries = config.retries || 3;
};

PaymentProcessor.prototype.process = function(payment, callback) {
  var self = this;
  var attempts = 0;

  function tryProcess() {
    attempts++;
    var url = self.sandbox
      ? 'https://sandbox.payment.com/api'
      : 'https://api.payment.com/api';

    fetch(url + '/charge', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer ' + self.apiKey,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        amount: payment.amount,
        currency: payment.currency || 'USD',
        customer: payment.customerId,
        description: payment.description
      })
    })
    .then(function(response) {
      if (!response.ok) {
        throw new Error('Payment failed: ' + response.status);
      }
      return response.json();
    })
    .then(function(result) {
      callback(null, result);
    })
    .catch(function(error) {
      if (attempts < self.retries) {
        setTimeout(tryProcess, 1000 * attempts);
      } else {
        callback(error);
      }
    });
  }

  tryProcess();
};

PaymentProcessor.prototype.refund = function(transactionId, amount, callback) {
  // Similar legacy code...
};

module.exports = PaymentProcessor;
```

### 6.3 Refactoring Steps

```
┌──────────────────────────────────────────────────────────────┐
│              REFACTORING WORKFLOW                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Step 1: Analysis                                             │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Identify all public methods                        │    │
│  │  - Document current behavior                          │    │
│  │  - Find all usages in codebase                        │    │
│  │  - Create test cases for current behavior             │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  Step 2: Type Definitions                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Create interfaces for config, payment, result      │    │
│  │  - Define error types                                 │    │
│  │  - Create type guards                                 │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  Step 3: Core Refactoring                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Convert to ES6 class                               │    │
│  │  - Add TypeScript types                               │    │
│  │  - Convert callbacks to async/await                   │    │
│  │  - Add proper error handling                          │    │
│  └──────────────────────────────────────────────────────┘    │
│                           │                                   │
│                           ▼                                   │
│  Step 4: Validation                                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  - Run existing tests                                 │    │
│  │  - Type check passes                                  │    │
│  │  - No breaking changes to consumers                   │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 6.4 Refactored Code

```typescript
// src/services/payment-processor.ts (AFTER)

// Types
interface PaymentConfig {
  apiKey: string;
  sandbox?: boolean;
  retries?: number;
  timeout?: number;
}

interface PaymentRequest {
  amount: number;
  currency?: string;
  customerId: string;
  description?: string;
  metadata?: Record<string, unknown>;
}

interface PaymentResult {
  id: string;
  status: 'succeeded' | 'pending' | 'failed';
  amount: number;
  currency: string;
  createdAt: Date;
}

interface RefundResult {
  id: string;
  transactionId: string;
  amount: number;
  status: 'succeeded' | 'pending' | 'failed';
}

class PaymentError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode?: number
  ) {
    super(message);
    this.name = 'PaymentError';
  }
}

// Main class
export class PaymentProcessor {
  private readonly apiKey: string;
  private readonly baseUrl: string;
  private readonly retries: number;
  private readonly timeout: number;

  constructor(config: PaymentConfig) {
    this.apiKey = config.apiKey;
    this.baseUrl = config.sandbox
      ? 'https://sandbox.payment.com/api'
      : 'https://api.payment.com/api';
    this.retries = config.retries ?? 3;
    this.timeout = config.timeout ?? 30000;
  }

  async process(payment: PaymentRequest): Promise<PaymentResult> {
    return this.withRetry(() => this.executeCharge(payment));
  }

  async refund(
    transactionId: string,
    amount?: number
  ): Promise<RefundResult> {
    return this.withRetry(() =>
      this.executeRefund(transactionId, amount)
    );
  }

  private async executeCharge(
    payment: PaymentRequest
  ): Promise<PaymentResult> {
    const response = await this.fetch('/charge', {
      method: 'POST',
      body: JSON.stringify({
        amount: payment.amount,
        currency: payment.currency ?? 'USD',
        customer: payment.customerId,
        description: payment.description,
        metadata: payment.metadata,
      }),
    });

    return this.parseResponse<PaymentResult>(response);
  }

  private async executeRefund(
    transactionId: string,
    amount?: number
  ): Promise<RefundResult> {
    const response = await this.fetch('/refund', {
      method: 'POST',
      body: JSON.stringify({
        transactionId,
        amount,
      }),
    });

    return this.parseResponse<RefundResult>(response);
  }

  private async fetch(
    endpoint: string,
    options: RequestInit
  ): Promise<Response> {
    const controller = new AbortController();
    const timeoutId = setTimeout(
      () => controller.abort(),
      this.timeout
    );

    try {
      return await fetch(`${this.baseUrl}${endpoint}`, {
        ...options,
        headers: {
          Authorization: `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json',
          ...options.headers,
        },
        signal: controller.signal,
      });
    } finally {
      clearTimeout(timeoutId);
    }
  }

  private async parseResponse<T>(response: Response): Promise<T> {
    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new PaymentError(
        error.message ?? 'Payment request failed',
        error.code ?? 'UNKNOWN_ERROR',
        response.status
      );
    }
    return response.json();
  }

  private async withRetry<T>(
    operation: () => Promise<T>,
    attempt = 1
  ): Promise<T> {
    try {
      return await operation();
    } catch (error) {
      if (attempt >= this.retries || !this.isRetryable(error)) {
        throw error;
      }

      await this.delay(1000 * attempt);
      return this.withRetry(operation, attempt + 1);
    }
  }

  private isRetryable(error: unknown): boolean {
    if (error instanceof PaymentError) {
      return error.statusCode !== undefined &&
        error.statusCode >= 500;
    }
    return error instanceof Error &&
      error.name === 'AbortError';
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### 6.5 Lab Completion Checklist

```
□ Analysis Complete
  □ All methods documented
  □ All usages found
  □ Baseline tests created

□ Types Created
  □ PaymentConfig interface
  □ PaymentRequest interface
  □ PaymentResult interface
  □ PaymentError class

□ Refactoring Complete
  □ Converted to ES6 class
  □ TypeScript types added
  □ Async/await implemented
  □ Proper error handling

□ Validation
  □ All tests pass
  □ No type errors
  □ No breaking changes
  □ Code review complete
```

---

## 7. HANDS-ON LAB 3: CI/CD PIPELINE SETUP

### 7.1 Lab Overview

**Objective**: Setup complete CI/CD pipeline với Claude Code integration

**Duration**: 1.5 hours

### 7.2 Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                    TRIGGERS                          │     │
│  │  Push to main │ Pull Request │ Manual │ Schedule    │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                   BUILD STAGE                        │     │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │     │
│  │  │ Install │─►│  Lint   │─►│  Build  │             │     │
│  │  │  Deps   │  │         │  │         │             │     │
│  │  └─────────┘  └─────────┘  └─────────┘             │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                   TEST STAGE                         │     │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │     │
│  │  │  Unit   │  │ Integra │  │   E2E   │ (parallel)  │     │
│  │  │  Tests  │  │  tion   │  │  Tests  │             │     │
│  │  └─────────┘  └─────────┘  └─────────┘             │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                  REVIEW STAGE                        │     │
│  │  ┌─────────────┐  ┌─────────────┐                   │     │
│  │  │Claude Code  │  │  Security   │                   │     │
│  │  │   Review    │  │    Scan     │                   │     │
│  │  └─────────────┘  └─────────────┘                   │     │
│  └─────────────────────────────────────────────────────┘     │
│                           │                                   │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │                  DEPLOY STAGE                        │     │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │     │
│  │  │ Staging │─►│  Smoke  │─►│  Prod   │             │     │
│  │  │ Deploy  │  │  Tests  │  │ Deploy  │             │     │
│  │  └─────────┘  └─────────┘  └─────────┘             │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 7.3 GitHub Actions Configuration

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  # ============================================
  # BUILD STAGE
  # ============================================
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Type check
        run: pnpm type-check

      - name: Build
        run: pnpm build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: .next/

  # ============================================
  # TEST STAGE (Parallel)
  # ============================================
  unit-tests:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm test:unit --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4

  integration-tests:
    needs: build
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
      - run: pnpm test:integration

  e2e-tests:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps

      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: build
          path: .next/

      - run: pnpm test:e2e

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # ============================================
  # REVIEW STAGE
  # ============================================
  claude-review:
    needs: [unit-tests, integration-tests, e2e-tests]
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed
        run: |
          echo "files=$(git diff --name-only origin/main...HEAD | tr '\n' ' ')" >> $GITHUB_OUTPUT

      - name: Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          claude --print "Review these changed files for code quality, security, and best practices: ${{ steps.changed.outputs.files }}"

  security-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  # ============================================
  # DEPLOY STAGE
  # ============================================
  deploy-staging:
    needs: [claude-review, security-scan]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Staging
        run: |
          # Deployment script
          echo "Deploying to staging..."

  deploy-production:
    needs: [claude-review, security-scan]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Production
        run: |
          # Deployment script
          echo "Deploying to production..."
```

### 7.4 Pre-commit Hook với Claude Code

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: claude-check
        name: Claude Code Quick Check
        entry: claude --print "Quick check for obvious issues in staged changes"
        language: system
        stages: [commit]
        pass_filenames: false
```

### 7.5 Lab Completion Checklist

```
□ GitHub Actions Setup
  □ Build job configured
  □ Test jobs running in parallel
  □ Claude review integration
  □ Security scanning enabled
  □ Deploy stages configured

□ Branch Protection
  □ Require PR reviews
  □ Require status checks
  □ Require Claude review pass

□ Secrets Configured
  □ ANTHROPIC_API_KEY
  □ Deploy credentials
  □ Database URLs

□ Monitoring
  □ Build status badges
  □ Coverage reports
  □ Security alerts
```

---

## TỔNG KẾT PHẦN 4

### Key Takeaways

1. **Multi-agent orchestration** cho phép complex task decomposition
2. **Parallel execution** tăng efficiency đáng kể
3. **Error handling patterns** đảm bảo reliability
4. **CI/CD integration** automates quality gates
5. **Hands-on labs** provide practical experience

### Competency Matrix

| Skill | Beginner | Intermediate | Advanced |
|-------|----------|--------------|----------|
| Single Agent | ✓ | | |
| Multi-Agent | | ✓ | |
| Orchestration | | | ✓ |
| CI/CD Integration | | ✓ | |
| Custom MCP | | | ✓ |

### Phần Tiếp Theo

**Part 5: Templates & Cheatsheet** sẽ cung cấp:
- Production-ready templates
- Quick reference cheatsheet
- Troubleshooting guide
- Best practices summary
