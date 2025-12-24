# Claude Code Training - Developer Track

## Dành cho: Frontend, Backend, QA, DevOps Teams

---

## Giới thiệu

Tài liệu training này được thiết kế cho các Developer tại MangoAds để master Claude Code - công cụ AI coding assistant của Anthropic. Nội dung tập trung vào việc xây dựng các professional subagents phục vụ công việc phát triển phần mềm.

## Mục tiêu

Sau khi hoàn thành training, bạn sẽ có thể:

1. **Hiểu architecture** của Claude Code và các thành phần
2. **Tạo subagents** chuyên biệt cho development workflows
3. **Cấu hình MCP** để mở rộng capabilities
4. **Tự động hóa testing** với Playwright và DevTools MCP
5. **Thiết kế workflows** phức tạp với multi-agent orchestration
6. **Tích hợp CI/CD** với Claude Code

## Cấu trúc tài liệu

| Part | Nội dung | Thời lượng |
|------|----------|------------|
| [Part 1](part-01-foundation-architecture.md) | Foundation & Architecture | 2h |
| [Part 2](part-02-subagents-hooks.md) | Subagents & Hooks | 2h |
| [Part 3](part-03-mcp-testing.md) | MCP & Testing Automation | 2h |
| [Part 4](part-04-advanced-patterns-labs.md) | Advanced Patterns & Labs | 3h |
| [Part 5](part-05-templates-cheatsheet.md) | Templates & Cheatsheet | 1h |

**Tổng thời lượng**: ~10 giờ

## Chi tiết từng phần

### Part 1: Foundation & Architecture
- Claude Code là gì và tại sao sử dụng
- Kiến trúc hệ thống và các components
- Memory System (CLAUDE.md, .claude/rules/)
- CLI Commands và Sessions
- Tools và Permission Modes
- Checkpointing và Recovery

### Part 2: Subagents & Hooks
- Subagent deep dive với YAML configuration
- Skill system và tích hợp
- Model selection strategies
- Tool scoping và least privilege
- Hook system (PreToolUse, PostToolUse, Stop, SessionStart)
- Practical examples và workflows

### Part 3: MCP & Testing Automation
- MCP (Model Context Protocol) architecture
- Playwright MCP cho browser automation
- DevTools MCP cho debugging
- Testing automation strategies
- CI/CD integration patterns
- Best practices cho testing

### Part 4: Advanced Patterns & Labs
- Multi-agent orchestration patterns
- Complex workflow automation
- Performance optimization techniques
- Error handling và recovery
- **Hands-on Lab 1**: Full-stack feature development
- **Hands-on Lab 2**: Legacy code refactoring
- **Hands-on Lab 3**: CI/CD pipeline setup

### Part 5: Templates & Cheatsheet
- Production-ready CLAUDE.md template
- Agent templates (code-reviewer, test-generator, api-builder, etc.)
- Hook templates (pre-commit, post-edit, security-gate)
- MCP configuration templates
- Quick reference cheatsheet
- Troubleshooting guide
- Assessment rubric

## Prerequisites

- Kiến thức TypeScript/JavaScript
- Familiar với Git và command line
- Hiểu biết cơ bản về React/Next.js (cho frontend)
- Hiểu biết về REST APIs (cho backend)
- Có Claude Code đã cài đặt

## Quick Start

```bash
# 1. Clone project
git clone <repo-url>
cd project

# 2. Khởi động Claude Code
claude

# 3. Đọc CLAUDE.md
claude /memory

# 4. Thử command đầu tiên
claude "Analyze the project structure"
```

## Skill Assessment Levels

| Level | Tên | Thời gian | Skills |
|-------|-----|-----------|--------|
| 1 | Beginner | 1-2 tuần | CLI cơ bản, interactive chat |
| 2 | Intermediate | 2-4 tuần | Subagents, hooks, commands |
| 3 | Advanced | 1-2 tháng | MCP, multi-agent, CI/CD |
| 4 | Expert | 2+ tháng | Custom MCP, enterprise workflows |

## Recommended Learning Path

```
Week 1: Part 1 + Part 2
        ↓
Week 2: Part 3 + Lab 1
        ↓
Week 3: Part 4 (Labs)
        ↓
Week 4: Part 5 + Real Projects
```

## Support

- **Internal**: Slack channel #claude-code-training
- **Documentation**: https://docs.anthropic.com/claude-code
- **Issues**: https://github.com/anthropics/claude-code/issues

---

**Bắt đầu với [Part 1: Foundation & Architecture](part-01-foundation-architecture.md)**

---

© 2024 MangoAds - Internal Training Material
