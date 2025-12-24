# PHẦN 5: TEMPLATES & CHEATSHEET

## Tài liệu dành cho: Frontend, Backend, QA, DevOps Teams

---

## MỤC LỤC

1. [Production-Ready Templates](#1-production-ready-templates)
2. [CLAUDE.md Template](#2-claudemd-template)
3. [Agent Templates](#3-agent-templates)
4. [Hook Templates](#4-hook-templates)
5. [MCP Configuration Templates](#5-mcp-configuration-templates)
6. [Quick Reference Cheatsheet](#6-quick-reference-cheatsheet)
7. [Troubleshooting Guide](#7-troubleshooting-guide)
8. [Governance & Assessment](#8-governance--assessment)

---

## 1. PRODUCTION-READY TEMPLATES

### 1.1 Complete Project Structure

```
project-root/
├── .claude/
│   ├── settings.json          # MCP & project settings
│   ├── agents/                # Subagent definitions
│   │   ├── code-reviewer.yml
│   │   ├── test-generator.yml
│   │   ├── api-builder.yml
│   │   ├── frontend-dev.yml
│   │   ├── db-architect.yml
│   │   ├── security-auditor.yml
│   │   └── docs-writer.yml
│   ├── commands/              # Slash commands
│   │   ├── review.md
│   │   ├── test.md
│   │   ├── deploy.md
│   │   └── debug.md
│   ├── hooks/                 # Event hooks
│   │   ├── pre-commit.js
│   │   ├── post-edit.js
│   │   └── session-start.js
│   ├── rules/                 # Scoped rules
│   │   ├── api-standards.md
│   │   ├── react-patterns.md
│   │   └── testing-rules.md
│   └── cache/                 # Analysis cache
│       └── .gitkeep
├── CLAUDE.md                  # Main project memory
└── ...
```

### 1.2 Settings.json Template

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true"
      }
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
  },
  "permissions": {
    "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"],
    "deniedTools": [],
    "requireConfirmation": ["Bash"]
  }
}
```

---

## 2. CLAUDE.MD TEMPLATE

### 2.1 Complete CLAUDE.md

```markdown
# Project: [PROJECT_NAME]

## Overview
[Brief description of the project, its purpose, and main features]

## Tech Stack
- **Frontend**: Next.js 14, React 18, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes, Prisma, PostgreSQL
- **Testing**: Vitest, Playwright, React Testing Library
- **Infrastructure**: Vercel, GitHub Actions

## Project Structure
```
src/
├── app/                # Next.js App Router
│   ├── api/           # API routes
│   ├── (auth)/        # Auth pages group
│   └── (dashboard)/   # Dashboard pages group
├── components/        # React components
│   ├── ui/           # Base UI components
│   └── features/     # Feature-specific components
├── lib/              # Utilities and helpers
│   ├── api/          # API client
│   ├── hooks/        # Custom React hooks
│   └── utils/        # Utility functions
├── services/         # Business logic
└── types/            # TypeScript types
```

## Development Commands
```bash
pnpm dev          # Start development server
pnpm build        # Production build
pnpm test         # Run unit tests
pnpm test:e2e     # Run E2E tests
pnpm lint         # Run linter
pnpm type-check   # TypeScript check
```

## Coding Standards

### TypeScript
- Use strict mode
- Prefer interfaces over types for objects
- Use proper generics instead of `any`
- Always define return types for functions

### React
- Functional components only
- Use React Query for data fetching
- Implement proper error boundaries
- Follow component composition patterns

### API
- RESTful conventions
- Zod validation for all inputs
- Consistent error response format
- Proper HTTP status codes

### Testing
- Minimum 80% coverage for new code
- Unit tests for utilities and hooks
- Integration tests for API endpoints
- E2E tests for critical user flows

## Environment Variables
```env
DATABASE_URL=           # PostgreSQL connection string
NEXTAUTH_SECRET=        # NextAuth.js secret
NEXTAUTH_URL=           # Application URL
```

## Key Decisions
1. **State Management**: React Query for server state, Zustand for client state
2. **Styling**: Tailwind CSS with custom design system
3. **Forms**: React Hook Form with Zod validation
4. **Auth**: NextAuth.js with JWT strategy

## Common Patterns

### API Route Pattern
```typescript
// src/app/api/[resource]/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';

const schema = z.object({ ... });

export async function POST(request: Request) {
  const body = await request.json();
  const validated = schema.parse(body);
  // ... implementation
  return NextResponse.json({ data }, { status: 201 });
}
```

### Component Pattern
```typescript
// src/components/features/[Feature]/[Feature].tsx
interface FeatureProps {
  // props
}

export function Feature({ ...props }: FeatureProps) {
  // implementation
}
```

## Agents Available
- `/code-reviewer` - Review code changes
- `/test-generator` - Generate unit tests
- `/api-builder` - Create API endpoints
- `/security-audit` - Security analysis

## Notes for Claude
- Always run tests after making changes
- Check TypeScript errors before completing
- Follow existing patterns in the codebase
- Create small, focused commits
```

---

## 3. AGENT TEMPLATES

### 3.1 Code Reviewer Agent

```yaml
# .claude/agents/code-reviewer.yml
name: code-reviewer
description: |
  Review code for quality, security, and best practices.
  Use for: PR reviews, code audits, refactoring suggestions.
model: opus
tools:
  - Read
  - Glob
  - Grep
permissionMode: default
skills:
  - name: security-patterns
    path: .claude/skills/security.md
  - name: code-quality
    path: .claude/skills/quality.md
```

**Associated Skill File:**

```markdown
<!-- .claude/skills/security.md -->
# Security Review Patterns

## Check for:
1. SQL Injection - parameterized queries required
2. XSS - proper output encoding
3. CSRF - token validation
4. Auth bypass - proper middleware usage
5. Secrets exposure - no hardcoded credentials
6. Input validation - Zod schemas required
7. Rate limiting - on public endpoints

## Report Format:
- Severity: Critical/High/Medium/Low
- Location: file:line
- Issue: Description
- Fix: Recommendation
```

### 3.2 Test Generator Agent

```yaml
# .claude/agents/test-generator.yml
name: test-generator
description: |
  Generate comprehensive tests for source files.
  Supports: Unit, integration, and E2E tests.
model: sonnet
tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
permissionMode: acceptEdits
```

**Slash Command:**

```markdown
<!-- .claude/commands/test.md -->
# Generate Tests

Generate comprehensive tests for the specified file(s).

## Usage
/test [file_path] [test_type]

## Arguments
- file_path: Path to the source file (required)
- test_type: unit | integration | e2e (default: unit)

## Process
1. Read and analyze the source file
2. Identify functions and their behaviors
3. Generate test cases covering:
   - Happy path
   - Edge cases
   - Error cases
4. Write test file
5. Run tests to verify

## Output
Test file at: [source].test.ts
```

### 3.3 API Builder Agent

```yaml
# .claude/agents/api-builder.yml
name: api-builder
description: |
  Create REST API endpoints following project conventions.
  Includes validation, error handling, and documentation.
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
permissionMode: acceptEdits
skills:
  - name: api-conventions
    path: .claude/skills/api.md
```

**Associated Skill:**

```markdown
<!-- .claude/skills/api.md -->
# API Development Conventions

## Endpoint Structure
```typescript
// src/app/api/[resource]/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

// 1. Define schemas
const CreateSchema = z.object({
  name: z.string().min(1).max(100),
  // ...
});

// 2. GET - List/Read
export async function GET(request: Request) {
  const session = await getServerSession();
  if (!session) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  const data = await prisma.resource.findMany({
    where: { userId: session.user.id }
  });

  return NextResponse.json({ data });
}

// 3. POST - Create
export async function POST(request: Request) {
  const session = await getServerSession();
  if (!session) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  const body = await request.json();
  const validated = CreateSchema.safeParse(body);

  if (!validated.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: validated.error.issues },
      { status: 400 }
    );
  }

  const data = await prisma.resource.create({
    data: { ...validated.data, userId: session.user.id }
  });

  return NextResponse.json({ data }, { status: 201 });
}
```

## Error Response Format
```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": []
}
```

## HTTP Status Codes
- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error
```

### 3.4 Frontend Developer Agent

```yaml
# .claude/agents/frontend-dev.yml
name: frontend-dev
description: |
  Create React components following design system.
  Includes accessibility and responsive design.
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Bash
permissionMode: acceptEdits
skills:
  - name: react-patterns
    path: .claude/skills/react.md
  - name: accessibility
    path: .claude/skills/a11y.md
```

### 3.5 Security Auditor Agent

```yaml
# .claude/agents/security-auditor.yml
name: security-auditor
description: |
  Comprehensive security audit of codebase.
  Checks OWASP Top 10 and security best practices.
model: opus
tools:
  - Read
  - Glob
  - Grep
  - Bash
permissionMode: default
```

### 3.6 Database Architect Agent

```yaml
# .claude/agents/db-architect.yml
name: db-architect
description: |
  Design and implement database schemas.
  Handles migrations, indexes, and relationships.
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
permissionMode: acceptEdits
```

---

## 4. HOOK TEMPLATES

### 4.1 Pre-commit Quality Check

```javascript
// .claude/hooks/pre-commit.js
export default {
  event: 'PreToolUse',
  hooks: [
    {
      name: 'pre-commit-check',
      script: async (context) => {
        const { tool, params } = context;

        if (tool !== 'Bash') return;
        if (!params.command.includes('git commit')) return;

        // Check for test runs in session
        const hasTests = context.conversation.some(
          msg => msg.content?.includes('pnpm test') ||
                 msg.content?.includes('vitest')
        );

        // Check for type errors
        const hasTypeCheck = context.conversation.some(
          msg => msg.content?.includes('pnpm type-check') ||
                 msg.content?.includes('tsc')
        );

        const warnings = [];
        if (!hasTests) warnings.push('Tests not run');
        if (!hasTypeCheck) warnings.push('Type check not run');

        if (warnings.length > 0) {
          return {
            note: `Pre-commit warning: ${warnings.join(', ')}. Consider running these before commit.`
          };
        }
      }
    }
  ]
};
```

### 4.2 Post-Edit Validation

```javascript
// .claude/hooks/post-edit.js
export default {
  event: 'PostToolUse',
  hooks: [
    {
      name: 'post-edit-validate',
      script: async (context) => {
        const { tool, params, result } = context;

        if (tool !== 'Edit' && tool !== 'Write') return;

        const filePath = params.file_path;

        // TypeScript file edited
        if (filePath.endsWith('.ts') || filePath.endsWith('.tsx')) {
          return {
            note: 'TypeScript file modified. Run type-check to verify.'
          };
        }

        // Test file edited
        if (filePath.includes('.test.') || filePath.includes('.spec.')) {
          return {
            note: 'Test file modified. Run tests to verify.'
          };
        }

        // Schema file edited
        if (filePath.includes('schema.prisma')) {
          return {
            note: 'Prisma schema modified. Run `pnpm prisma generate` and create migration if needed.'
          };
        }
      }
    }
  ]
};
```

### 4.3 Session Start Setup

```javascript
// .claude/hooks/session-start.js
export default {
  event: 'SessionStart',
  hooks: [
    {
      name: 'environment-check',
      script: async (context) => {
        const checks = [];

        // Check Node version
        const nodeVersion = process.version;
        if (!nodeVersion.startsWith('v20')) {
          checks.push(`Node.js version: ${nodeVersion} (recommended: v20.x)`);
        }

        // Check git status
        const { execSync } = require('child_process');
        try {
          const status = execSync('git status --porcelain').toString();
          if (status.trim()) {
            const changedFiles = status.split('\n').length;
            checks.push(`${changedFiles} uncommitted changes`);
          }
        } catch (e) {
          // Not a git repo
        }

        if (checks.length > 0) {
          return {
            note: `Session info: ${checks.join('; ')}`
          };
        }
      }
    }
  ]
};
```

### 4.4 Security Gate Hook

```javascript
// .claude/hooks/security-gate.js
export default {
  event: 'PreToolUse',
  hooks: [
    {
      name: 'security-gate',
      script: async (context) => {
        const { tool, params } = context;

        // Block dangerous commands
        if (tool === 'Bash') {
          const dangerousPatterns = [
            /rm\s+-rf\s+\//,
            />\s*\/dev\/sd/,
            /mkfs\./,
            /dd\s+if=/,
            /:(){.*};:/,  // Fork bomb
          ];

          for (const pattern of dangerousPatterns) {
            if (pattern.test(params.command)) {
              return {
                action: 'block',
                message: 'Potentially dangerous command blocked for safety.'
              };
            }
          }
        }

        // Block writing to sensitive files
        if (tool === 'Write' || tool === 'Edit') {
          const sensitiveFiles = [
            /\.env$/,
            /credentials/i,
            /secrets/i,
            /\.pem$/,
            /\.key$/,
          ];

          for (const pattern of sensitiveFiles) {
            if (pattern.test(params.file_path)) {
              return {
                action: 'block',
                message: 'Cannot modify sensitive files. Manual edit required.'
              };
            }
          }
        }
      }
    }
  ]
};
```

---

## 5. MCP CONFIGURATION TEMPLATES

### 5.1 Full-Stack Development Setup

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true",
        "PLAYWRIGHT_BROWSER": "chromium"
      }
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
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-filesystem"],
      "env": {
        "ALLOWED_PATHS": "${PROJECT_ROOT}"
      }
    }
  }
}
```

### 5.2 QA/Testing Setup

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "false",
        "PLAYWRIGHT_SLOW_MO": "100"
      }
    },
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

### 5.3 DevOps Setup

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "docker": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-docker"]
    },
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-kubernetes"],
      "env": {
        "KUBECONFIG": "${HOME}/.kube/config"
      }
    }
  }
}
```

---

## 6. QUICK REFERENCE CHEATSHEET

### 6.1 CLI Commands

```
┌──────────────────────────────────────────────────────────────┐
│                 CLAUDE CODE CLI CHEATSHEET                    │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  BASIC COMMANDS                                               │
│  ───────────────────────────────────────────────────────────  │
│  claude                    # Start interactive session       │
│  claude "prompt"           # One-shot command                 │
│  claude -p "prompt"        # Print mode (no confirmation)    │
│  claude --help             # Show help                        │
│                                                               │
│  SESSION MANAGEMENT                                           │
│  ───────────────────────────────────────────────────────────  │
│  claude --resume           # Resume last session              │
│  claude --session-id ID    # Resume specific session          │
│  claude --new              # Force new session                │
│                                                               │
│  PERMISSION MODES                                             │
│  ───────────────────────────────────────────────────────────  │
│  claude --permission default        # Ask for each action    │
│  claude --permission acceptEdits    # Auto-accept edits      │
│  claude --permission plan           # Read-only planning     │
│  claude --permission bypass         # No confirmations       │
│                                                               │
│  MCP COMMANDS                                                 │
│  ───────────────────────────────────────────────────────────  │
│  claude mcp list           # List MCP servers                │
│  claude mcp add <name>     # Add MCP server                  │
│  claude mcp remove <name>  # Remove MCP server               │
│                                                               │
│  IN-SESSION COMMANDS                                          │
│  ───────────────────────────────────────────────────────────  │
│  /help                     # Show help                        │
│  /clear                    # Clear conversation               │
│  /compact                  # Compact conversation             │
│  /cost                     # Show token usage                 │
│  /doctor                   # Diagnose issues                  │
│  /memory                   # Show project memory              │
│  /undo                     # Undo last action                 │
│  /review                   # Trigger code review agent       │
│  /test                     # Trigger test generator          │
│                                                               │
│  PIPING                                                       │
│  ───────────────────────────────────────────────────────────  │
│  cat file.ts | claude "review this"                          │
│  git diff | claude "summarize changes"                       │
│  echo "fix bug" | claude                                     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 6.2 Agent Definition Quick Reference

```yaml
# Minimal agent
name: my-agent
description: "What this agent does"
model: sonnet
tools:
  - Read
  - Write

# Full agent
name: advanced-agent
description: |
  Multi-line description
  with details
model: opus
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - mcp__playwright__navigate
  - mcp__playwright__screenshot
permissionMode: acceptEdits  # default | acceptEdits | plan | bypassPermissions
skills:
  - name: skill-name
    path: .claude/skills/skill.md
```

### 6.3 Hook Quick Reference

```javascript
// Hook structure
export default {
  event: 'PreToolUse',  // PreToolUse | PostToolUse | Stop | SubagentStop | SessionStart
  hooks: [
    {
      name: 'hook-name',
      script: async (context) => {
        // context.tool - Tool being used
        // context.params - Tool parameters
        // context.result - Tool result (PostToolUse only)
        // context.conversation - Full conversation

        return {
          // Options:
          action: 'continue',  // continue | block | retry
          note: 'Message to show',
          message: 'Block reason',  // For block action
        };
      }
    }
  ]
};
```

### 6.4 Common Patterns

```
┌──────────────────────────────────────────────────────────────┐
│                    COMMON PATTERNS                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  SCOPED RULES                                                 │
│  ───────────────────────────────────────────────────────────  │
│  ---                                                          │
│  scope: src/api/**                                           │
│  ---                                                          │
│  # Rules for API files only                                   │
│                                                               │
│  CONDITIONAL TOOL USAGE                                       │
│  ───────────────────────────────────────────────────────────  │
│  tools:                                                       │
│    - Read                    # Always available              │
│    - Write                   # Always available              │
│    - Bash: ["npm *", "pnpm *"]  # Restricted patterns       │
│                                                               │
│  ENVIRONMENT VARIABLES                                        │
│  ───────────────────────────────────────────────────────────  │
│  "env": {                                                     │
│    "KEY": "${ENV_VAR}",      # From environment              │
│    "STATIC": "value"         # Static value                  │
│  }                                                            │
│                                                               │
│  SKILL REFERENCE                                              │
│  ───────────────────────────────────────────────────────────  │
│  skills:                                                      │
│    - name: skill-name                                         │
│      path: .claude/skills/file.md                            │
│    - name: inline-skill                                       │
│      content: |                                               │
│        Inline skill content                                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. TROUBLESHOOTING GUIDE

### 7.1 Common Issues

```
┌──────────────────────────────────────────────────────────────┐
│                  TROUBLESHOOTING GUIDE                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ISSUE: Agent không load được                                 │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Check YAML syntax (spaces, not tabs)                      │
│  ✓ Verify file path: .claude/agents/name.yml                 │
│  ✓ Check model name: haiku | sonnet | opus                   │
│  ✓ Validate tool names (case-sensitive)                      │
│                                                               │
│  ISSUE: MCP server không connect                              │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Run: claude mcp list                                      │
│  ✓ Check settings.json syntax                                │
│  ✓ Verify npx command works manually                         │
│  ✓ Check environment variables                               │
│  ✓ Try: claude mcp test <server-name>                        │
│                                                               │
│  ISSUE: Hook không trigger                                    │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Verify event name spelling                                │
│  ✓ Check hook file exports default                           │
│  ✓ Validate JavaScript syntax                                │
│  ✓ Check file path: .claude/hooks/name.js                    │
│                                                               │
│  ISSUE: Permission denied errors                              │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Check file/folder permissions                             │
│  ✓ Verify working directory                                  │
│  ✓ Try with --permission bypass (if safe)                    │
│                                                               │
│  ISSUE: Context limit exceeded                                │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Use /compact to reduce context                            │
│  ✓ Start new session for different task                      │
│  ✓ Break large tasks into smaller agents                     │
│                                                               │
│  ISSUE: Slow response                                         │
│  ───────────────────────────────────────────────────────────  │
│  ✓ Use haiku for simple tasks                                │
│  ✓ Limit tools to only needed ones                           │
│  ✓ Use specific file patterns in Glob                        │
│  ✓ Add timeouts to Bash commands                             │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 7.2 Debug Commands

```bash
# Check Claude Code version
claude --version

# Run diagnostics
claude /doctor

# Verbose mode
claude --verbose

# Check MCP servers
claude mcp list
claude mcp test playwright

# View configuration
cat .claude/settings.json
cat CLAUDE.md

# Validate YAML
npx yaml-lint .claude/agents/*.yml
```

### 7.3 Error Messages Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Tool not found: X` | Invalid tool name | Check spelling, case-sensitivity |
| `MCP server failed to start` | Server config issue | Verify command and args |
| `Permission denied` | Insufficient permissions | Check permissionMode setting |
| `Context length exceeded` | Too much conversation | Use /compact or new session |
| `Rate limit exceeded` | Too many API calls | Wait and retry, or batch operations |
| `Invalid YAML` | Syntax error in config | Validate YAML format |
| `Hook execution failed` | JavaScript error | Check hook syntax and exports |

---

## 8. GOVERNANCE & ASSESSMENT

### 8.1 Claude Code Skill Matrix

```
┌──────────────────────────────────────────────────────────────┐
│                  SKILL ASSESSMENT MATRIX                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Level 1: Beginner (1-2 tuần)                                │
│  ───────────────────────────────────────────────────────────  │
│  □ Basic CLI commands                                        │
│  □ Interactive chat với Claude                               │
│  □ Simple file operations                                    │
│  □ Hiểu CLAUDE.md cơ bản                                     │
│                                                               │
│  Level 2: Intermediate (2-4 tuần)                            │
│  ───────────────────────────────────────────────────────────  │
│  □ Tạo và sử dụng subagents                                  │
│  □ Viết slash commands                                       │
│  □ Cấu hình scoped rules                                     │
│  □ Sử dụng permission modes                                  │
│  □ Basic hook implementation                                  │
│                                                               │
│  Level 3: Advanced (1-2 tháng)                               │
│  ───────────────────────────────────────────────────────────  │
│  □ MCP server configuration                                   │
│  □ Multi-agent orchestration                                 │
│  □ Complex hook logic                                        │
│  □ CI/CD integration                                         │
│  □ Performance optimization                                   │
│                                                               │
│  Level 4: Expert (2+ tháng)                                  │
│  ───────────────────────────────────────────────────────────  │
│  □ Custom MCP server development                              │
│  □ Enterprise workflow design                                 │
│  □ Security governance                                        │
│  □ Training & mentoring others                               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 Assessment Rubric

| Criteria | Beginner (1) | Developing (2) | Proficient (3) | Expert (4) |
|----------|--------------|----------------|----------------|------------|
| **CLI Usage** | Basic commands | All common commands | Advanced flags | Custom workflows |
| **Agents** | Use existing | Modify agents | Create new agents | Orchestrate multiple |
| **Hooks** | Understand concept | Simple hooks | Complex logic | Event-driven systems |
| **MCP** | Use configured | Configure new | Troubleshoot | Develop custom |
| **Testing** | Manual tests | Agent-assisted | Automated suite | CI/CD integrated |
| **Security** | Follow guidelines | Apply patterns | Design policies | Audit & govern |

### 8.3 Code Review Checklist

```markdown
## Claude Code Usage Review Checklist

### Configuration
- [ ] CLAUDE.md đầy đủ và cập nhật
- [ ] .claude/settings.json valid
- [ ] Agents có description rõ ràng
- [ ] Tools scoped đúng cách

### Security
- [ ] Không hardcode credentials
- [ ] Permission modes phù hợp
- [ ] Hook security gates active
- [ ] Sensitive files protected

### Best Practices
- [ ] Model selection phù hợp (haiku/sonnet/opus)
- [ ] Tools minimal (least privilege)
- [ ] Error handling trong hooks
- [ ] Documentation cập nhật

### Performance
- [ ] Không dùng quá nhiều tools
- [ ] Parallel execution khi có thể
- [ ] Caching được sử dụng
- [ ] Timeouts configured
```

### 8.4 Weekly Self-Assessment

```markdown
# Weekly Claude Code Self-Assessment

## Tuần: [DATE]

### Sử dụng
- [ ] Số sessions: ___
- [ ] Tasks hoàn thành: ___
- [ ] Agents sử dụng: ___

### Learning
- [ ] Tính năng mới học: ___
- [ ] Challenges gặp phải: ___
- [ ] Solutions tìm được: ___

### Improvement
- [ ] Điều làm tốt: ___
- [ ] Điều cần cải thiện: ___
- [ ] Goals tuần sau: ___

### Sharing
- [ ] Kiến thức chia sẻ với team: ___
- [ ] Templates/Agents đóng góp: ___
```

---

## TỔNG KẾT TRAINING PROGRAM

### Key Files Checklist

```
□ CLAUDE.md - Project memory
□ .claude/settings.json - MCP & settings
□ .claude/agents/*.yml - Subagent definitions
□ .claude/commands/*.md - Slash commands
□ .claude/hooks/*.js - Event hooks
□ .claude/rules/*.md - Scoped rules
□ .claude/skills/*.md - Agent skills
```

### Quick Start Commands

```bash
# 1. Initialize project
claude

# 2. Setup CLAUDE.md
claude "Create CLAUDE.md based on this project"

# 3. Create first agent
claude "Create a code-reviewer agent in .claude/agents/"

# 4. Configure MCP
claude "Setup Playwright MCP in settings.json"

# 5. Start working
claude "Review the codebase and suggest improvements"
```

### Resources

- **Official Docs**: https://docs.anthropic.com/claude-code
- **MCP Servers**: https://github.com/anthropics/mcp-servers
- **Examples**: https://github.com/anthropics/claude-code-examples

---

**END OF DEV TRAINING PROGRAM**

© 2024 MangoAds - Internal Training Material
