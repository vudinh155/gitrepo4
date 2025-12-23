# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 1: Overview & Project Memory / Rules

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Đối tượng:** Nhân viên MangoAds (Digital Marketing & Web/Software Development)

---

## MỤC LỤC PART 1

1. [Giới thiệu chương trình training](#1-giới-thiệu-chương-trình-training)
2. [Tổng quan Claude Code](#2-tổng-quan-claude-code)
3. [Phân track học viên](#3-phân-track-học-viên)
4. [Project Memory / Rules](#4-project-memory--rules)
   - CLAUDE.md
   - .claude/rules/
   - YAML frontmatter
   - /init command
5. [Checklist & Anti-patterns](#5-checklist--anti-patterns)

---

## 1. GIỚI THIỆU CHƯƠNG TRÌNH TRAINING

### 1.1 Mục tiêu

Sau khi hoàn thành chương trình training này, nhân viên MangoAds sẽ có khả năng:

**Hiểu biết nền tảng:**
- [ ] Hiểu rõ kiến trúc và các thành phần của Claude Code
- [ ] Phân biệt được Memory, Skill, Subagent, Hook, Command, Plugin, MCP
- [ ] Nắm vững khi nào sử dụng thành phần nào

**Kỹ năng thực hành:**
- [ ] Thiết lập và tối ưu CLAUDE.md cho dự án MangoAds
- [ ] Tạo Subagent chuyên biệt cho từng loại task
- [ ] Viết Skill tái sử dụng trong team
- [ ] Cấu hình Hooks để kiểm soát workflow
- [ ] Kết nối MCP servers (Playwright, Browser DevTools)
- [ ] Tự động hóa test web với MCP

**Áp dụng thực tế:**
- [ ] Ứng dụng Claude Code vào quy trình Digital Marketing
- [ ] Tích hợp Claude Code vào pipeline Web Development
- [ ] Xây dựng automation cho QA/Testing

### 1.2 Học xong làm được gì?

| Track | Sau training có thể làm |
|-------|------------------------|
| **Marketing** | Tạo subagent viết content, phân tích campaign, sinh UTM, tạo weekly report tự động |
| **Dev/QA** | Tạo subagent review code, test automation với Playwright MCP, debug với Browser DevTools MCP |
| **Chung** | Thiết kế workflow chuẩn, share skills trong team, audit & governance |

---

## 2. TỔNG QUAN CLAUDE CODE

### 2.1 Claude Code là gì?

Claude Code là CLI (Command Line Interface) chính thức của Anthropic, cho phép tương tác với Claude AI trực tiếp trong terminal. Đây là công cụ mạnh mẽ để:

- **Coding Assistant**: Viết, review, debug code
- **Task Automation**: Tự động hóa các tác vụ lặp đi lặp lại
- **Project Management**: Quản lý context và workflow dự án
- **Integration Hub**: Kết nối với tools bên ngoài qua MCP

### 2.2 Các thành phần chính (Overview)

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLAUDE CODE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Memory    │  │  Subagents  │  │   Skills    │              │
│  │ (CLAUDE.md) │  │             │  │             │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Hooks     │  │  Commands   │  │   Plugins   │              │
│  │             │  │             │  │             │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐            │
│  │                 MCP (Model Context Protocol)     │            │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │            │
│  │  │Playwright│  │ Browser │  │ GitHub  │  ...    │            │
│  │  └─────────┘  └─────────┘  └─────────┘         │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Bảng so sánh nhanh các thành phần

| Thành phần | Mục đích | Ai kích hoạt | Ví dụ MangoAds |
|------------|----------|--------------|----------------|
| **Memory** | Context dự án | Tự động load | Coding standards của team |
| **Subagent** | AI chuyên biệt | Claude tự chọn | Subagent review landing page |
| **Skill** | Khả năng tái sử dụng | Claude tự chọn | Skill sinh UTM link |
| **Hook** | Kiểm soát lifecycle | Tự động trigger | Block deploy chưa test |
| **Command** | Shortcut workflow | User gõ /command | /weekly-report |
| **Plugin** | Bundle đóng gói | Import từ team | MangoAds SEO Plugin |
| **MCP** | Kết nối tool ngoài | Claude gọi tool | Playwright test web |

---

## 3. PHÂN TRACK HỌC VIÊN

### 3.1 Track Marketing

**Đối tượng:** Content, Performance, SEO, Social Media team

**Focus chính:**
- CLAUDE.md cho campaign workflow
- Subagent viết content, phân tích data
- Skill sinh UTM, format report
- Slash commands cho tasks thường xuyên

**Labs ưu tiên:**
- Lab 1: Weekly report subagent
- Lab 2: UTM skill
- Ứng dụng thực tế: Automation báo cáo campaign

### 3.2 Track Development/Software

**Đối tượng:** Frontend, Backend, QA, DevOps team

**Focus chính:**
- CLAUDE.md cho coding standards
- Subagent review code, debug
- MCP integration (Playwright, Browser DevTools)
- Hooks cho CI/CD pipeline

**Labs ưu tiên:**
- Lab 3: MCP Playwright web test
- Ứng dụng thực tế: Visual regression testing

### 3.3 Nội dung chung bắt buộc

Cả hai track đều phải nắm vững:
- [ ] Project Memory & Rules
- [ ] Cơ bản về Subagent & Skill
- [ ] Decision Tree chọn thành phần
- [ ] Governance & Security

---

## 4. PROJECT MEMORY / RULES

### 4.1 CLAUDE.md - "Hiến pháp" của Project

#### 4.1.1 Định nghĩa

**CLAUDE.md** là file markdown chứa các hướng dẫn, context, và quy tắc mà Claude sẽ tự động load và tuân theo trong suốt session làm việc.

> **Analogy MangoAds:** Nếu project là một công ty con, thì CLAUDE.md chính là "Handbook" của công ty đó - mọi nhân viên (Claude) khi vào làm đều phải đọc và tuân theo.

#### 4.1.2 Vị trí file (Hierarchy)

Claude Code tìm kiếm CLAUDE.md theo thứ tự từ thấp lên cao:

```
📁 ~/                           # Global (áp dụng mọi project)
├── .claude/
│   └── CLAUDE.md              # Priority: Thấp nhất
│
📁 /project-root/               # Project root
├── CLAUDE.md                  # Priority: Cao hơn
├── .claude/
│   └── CLAUDE.md              # Priority: Cao hơn
├── CLAUDE.local.md            # Priority: Cao nhất (gitignore)
│
📁 /project-root/src/           # Subfolder
└── CLAUDE.md                  # Priority: Áp dụng khi ở folder này
```

**Quy tắc priority:**
- File cụ thể hơn (thư mục con) override file chung hơn
- `CLAUDE.local.md` luôn có priority cao nhất (dùng cho cá nhân, không commit)
- Tất cả CLAUDE.md được tìm thấy đều được load và merge

#### 4.1.3 Nội dung nên có trong CLAUDE.md

**Template chuẩn MangoAds:**

```markdown
# CLAUDE.md - [Tên Project]

## Project Overview
- Mô tả ngắn gọn project
- Tech stack: Next.js, Tailwind, shadcn/ui
- Môi trường: Node 20, pnpm

## Commands thường dùng
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`
- Dev: `pnpm dev`

## Coding Conventions
- Sử dụng TypeScript strict mode
- Component naming: PascalCase
- File naming: kebab-case
- Import order: React > Third-party > Local

## Cấu trúc thư mục
```
src/
├── components/     # UI components
├── hooks/          # Custom hooks
├── lib/            # Utilities
├── pages/          # Next.js pages
└── styles/         # Global styles
```

## Workflow
1. Tạo branch từ `develop`
2. Commit theo Conventional Commits
3. PR cần ít nhất 1 approve
4. Merge vào develop, CI tự deploy staging

## Lưu ý quan trọng
- KHÔNG commit .env files
- KHÔNG sửa trực tiếp production database
- Luôn chạy test trước khi commit

## Team Contacts
- Tech Lead: @lead-name
- QA: @qa-name
```

#### 4.1.4 Khi nào dùng CLAUDE.md

| Tình huống | Dùng CLAUDE.md? | Lý do |
|------------|-----------------|-------|
| Project mới | ✅ Bắt buộc | Thiết lập context từ đầu |
| Project có nhiều người | ✅ Bắt buộc | Đồng bộ hiểu biết trong team |
| Task đơn giản, 1 lần | ❌ Không cần | Overkill |
| Debug nhanh | ❌ Không cần | Chỉ cần context tức thời |

#### 4.1.5 Best Practices

**DO - Nên làm:**
```markdown
✅ Viết cụ thể, actionable
   "Component phải có PropTypes hoặc TypeScript interface"

✅ Include commands thực tế
   "Test: pnpm test -- --coverage"

✅ Cập nhật khi workflow thay đổi

✅ Review CLAUDE.md trong code review
```

**DON'T - Anti-patterns:**
```markdown
❌ Viết quá dài, không ai đọc hết
   → Chia thành rules/ files

❌ Viết mơ hồ
   "Code phải đẹp" → Đẹp là gì?

❌ Chứa secrets
   KHÔNG BAO GIỜ để API key, password trong CLAUDE.md

❌ Copy-paste không customize
   Template phải adapt cho project cụ thể
```

#### 4.1.6 Ví dụ thực tế MangoAds

**Project: Landing Page Campaign**

```markdown
# CLAUDE.md - MangoAds Landing Page Campaign

## Project Info
- Client: [Client Name]
- Campaign: Q1 2025 Product Launch
- Tech: Next.js 14, Tailwind CSS, shadcn/ui

## Commands
- Dev: `pnpm dev` (port 3000)
- Build: `pnpm build`
- Test: `pnpm test`
- Lighthouse: `pnpm lighthouse`

## Performance Requirements
- LCP < 2.5s
- FID < 100ms
- CLS < 0.1
- Mobile PageSpeed Score > 90

## UTM Convention
- Source: google, facebook, linkedin
- Medium: cpc, social, email
- Campaign: q1-2025-product-launch

## Content Guidelines
- Tone: Professional nhưng friendly
- CTA: Action-oriented (Đăng ký ngay, Tìm hiểu thêm)
- Không dùng: "Click here", passive voice

## Checklist trước deploy
- [ ] Test trên mobile (iOS Safari, Android Chrome)
- [ ] Verify tracking pixels (GA4, Facebook Pixel)
- [ ] Check form submission
- [ ] Validate UTM parameters
```

### 4.2 .claude/rules/ - Modular Rules

#### 4.2.1 Định nghĩa

Thay vì nhồi mọi thứ vào một CLAUDE.md, bạn có thể chia rules thành nhiều file nhỏ trong thư mục `.claude/rules/`.

#### 4.2.2 Cấu trúc thư mục

```
.claude/
├── rules/
│   ├── general.md           # Rules chung
│   ├── frontend/
│   │   ├── react.md         # Rules cho React
│   │   └── tailwind.md      # Rules cho Tailwind
│   ├── backend/
│   │   ├── api.md           # Rules cho API
│   │   └── database.md      # Rules cho DB
│   └── testing/
│       └── e2e.md           # Rules cho E2E test
```

#### 4.2.3 YAML Frontmatter - Scope Rules

Dùng YAML frontmatter để giới hạn scope áp dụng của rule:

**Ví dụ 1: Rule cho tất cả TypeScript files**

```markdown
---
paths: "**/*.ts"
---

# TypeScript Rules

- Sử dụng strict mode
- Không dùng `any`, prefer `unknown`
- Export interface trước khi export implementation
```

**Ví dụ 2: Rule cho API routes**

```markdown
---
paths: ["src/api/**/*.ts", "src/pages/api/**/*.ts"]
---

# API Development Rules

- Mọi endpoint phải có input validation
- Response format chuẩn: { success: boolean, data?: T, error?: string }
- Logging với request ID
- Rate limiting cho public endpoints
```

**Ví dụ 3: Rule cho components**

```markdown
---
paths: "src/components/**/*.tsx"
---

# React Component Rules

- Mỗi component một file
- Props interface đặt tên: ComponentNameProps
- Dùng forwardRef cho interactive components
- Export named, không export default
```

#### 4.2.4 Khi nào dùng rules/ thay vì CLAUDE.md đơn

| Tình huống | CLAUDE.md đơn | .claude/rules/ |
|------------|---------------|----------------|
| Project nhỏ, < 10 rules | ✅ | ❌ |
| Project lớn, nhiều rules | ❌ | ✅ |
| Rules khác nhau cho frontend/backend | ❌ | ✅ |
| Team muốn modular | ❌ | ✅ |
| Cần scope theo file pattern | ❌ | ✅ |

#### 4.2.5 Ví dụ thực tế MangoAds

**File: `.claude/rules/marketing/utm.md`**

```markdown
---
paths: ["src/**/*campaign*", "src/**/*tracking*"]
---

# UTM Tracking Rules - MangoAds

## Chuẩn UTM Parameters
- utm_source: Nguồn traffic (google, facebook, email)
- utm_medium: Kênh (cpc, organic, social)
- utm_campaign: Tên campaign (dùng kebab-case)
- utm_content: Phân biệt ad/link cùng campaign
- utm_term: Keywords (cho paid search)

## Naming Convention
- Lowercase only
- Dùng dấu gạch ngang (-) thay space
- Không dấu tiếng Việt
- Format: {year}-{quarter}-{campaign-name}

## Ví dụ
✅ utm_campaign=2025-q1-product-launch
❌ utm_campaign=2025 Q1 Product Launch
❌ utm_campaign=2025_q1_sản_phẩm_mới

## Validation
Trước khi generate UTM link, kiểm tra:
- [ ] Không chứa space
- [ ] Không chứa ký tự đặc biệt ngoài - và _
- [ ] Lowercase toàn bộ
```

### 4.3 /init Command

#### 4.3.1 Định nghĩa

`/init` là command tự động phân tích codebase và sinh ra CLAUDE.md phù hợp.

#### 4.3.2 Cách sử dụng

```bash
# Mở Claude Code trong project
cd /path/to/project
claude

# Chạy /init
> /init
```

#### 4.3.3 /init làm gì?

```
1. Scan codebase
   ├── package.json → Dependencies, scripts
   ├── tsconfig.json → TypeScript config
   ├── README.md → Project description
   ├── .gitignore → Files to ignore
   └── Code structure → Directory layout

2. Detect patterns
   ├── Framework (Next.js, Express, etc.)
   ├── Styling (Tailwind, CSS Modules, etc.)
   ├── Testing (Jest, Vitest, etc.)
   └── Linting (ESLint, Prettier, etc.)

3. Generate CLAUDE.md
   └── Tailored cho project cụ thể
```

#### 4.3.4 Best Practices với /init

**Workflow chuẩn MangoAds:**

```bash
# 1. Clone project mới
git clone https://github.com/mangoads/new-project
cd new-project

# 2. Mở Claude Code
claude

# 3. Chạy /init để sinh CLAUDE.md
> /init

# 4. Review và customize output
# - Thêm workflow MangoAds
# - Thêm team contacts
# - Thêm project-specific notes

# 5. Commit CLAUDE.md
git add CLAUDE.md
git commit -m "chore: add CLAUDE.md for AI context"
```

#### 4.3.5 Khi nào dùng /init

| Tình huống | Dùng /init? |
|------------|-------------|
| Project mới, chưa có CLAUDE.md | ✅ |
| Onboard vào project có sẵn | ✅ |
| Update CLAUDE.md sau refactor lớn | ✅ |
| CLAUDE.md đã hoàn chỉnh | ❌ |

### 4.4 Memory như "Constitution"

#### 4.4.1 Khái niệm

Project Memory (CLAUDE.md + rules/) hoạt động như "hiến pháp" của project:

```
┌─────────────────────────────────────────────────┐
│              PROJECT MEMORY                      │
│                                                  │
│  ┌──────────────────────────────────────────┐   │
│  │           CLAUDE.md (Constitution)        │   │
│  │  - Nguyên tắc cốt lõi                    │   │
│  │  - Workflow chính                        │   │
│  │  - Commands quan trọng                   │   │
│  └──────────────────────────────────────────┘   │
│                      │                           │
│                      ▼                           │
│  ┌──────────────────────────────────────────┐   │
│  │         .claude/rules/ (Laws)            │   │
│  │  - Chi tiết cho từng domain              │   │
│  │  - Scoped theo file patterns             │   │
│  │  - Modular, dễ maintain                  │   │
│  └──────────────────────────────────────────┘   │
│                                                  │
└─────────────────────────────────────────────────┘
```

#### 4.4.2 Tại sao gọi là "Constitution"?

| Đặc điểm | Constitution thật | CLAUDE.md |
|----------|-------------------|-----------|
| Tự động áp dụng | Mọi công dân tuân theo | Claude tự động load |
| Persistent | Không thay đổi thường xuyên | Survive qua sessions |
| Authoritative | Luật cao nhất | Priority cao với Claude |
| Shared | Áp dụng cho mọi người | Team dùng chung |

#### 4.4.3 Import syntax với @

CLAUDE.md hỗ trợ import file khác:

```markdown
# CLAUDE.md

## Project Overview
Đây là landing page campaign Q1 2025.

## Technical Stack
@docs/tech-stack.md

## Coding Standards
@.claude/rules/coding-standards.md

## API Documentation
@docs/api/README.md
```

**Lưu ý:**
- Path relative từ vị trí CLAUDE.md
- File được import sẽ được load vào context
- Không nên import quá nhiều (context limit)

---

## 5. CHECKLIST & ANTI-PATTERNS

### 5.1 Checklist thiết lập Memory cho project MangoAds

```markdown
## Checklist CLAUDE.md

### Cơ bản
- [ ] Có mô tả project overview
- [ ] List đầy đủ tech stack
- [ ] Có commands thường dùng (build, test, dev)
- [ ] Có coding conventions

### MangoAds-specific
- [ ] Có workflow (branch, PR, deploy)
- [ ] Có performance requirements (nếu là web project)
- [ ] Có tracking/UTM conventions (nếu là marketing project)
- [ ] Có team contacts

### Best Practices
- [ ] Không chứa secrets
- [ ] Đã review bởi team lead
- [ ] Được commit vào repo
- [ ] CLAUDE.local.md cho settings cá nhân (gitignored)
```

### 5.2 Anti-patterns cần tránh

#### Anti-pattern 1: CLAUDE.md quá dài

```markdown
❌ SAI
# CLAUDE.md dài 500+ dòng, chứa mọi thứ

✅ ĐÚNG
# Chia thành:
- CLAUDE.md: Overview + essentials (< 100 dòng)
- .claude/rules/: Chi tiết theo domain
```

#### Anti-pattern 2: Copy-paste template không customize

```markdown
❌ SAI
# Copy template từ internet, không sửa gì

✅ ĐÚNG
# Customize cho project cụ thể:
- Sửa tech stack thực tế
- Thêm commands thực tế
- Thêm workflow team đang dùng
```

#### Anti-pattern 3: Không cập nhật khi project thay đổi

```markdown
❌ SAI
# CLAUDE.md viết từ 6 tháng trước, không update

✅ ĐÚNG
# Review CLAUDE.md khi:
- Refactor lớn
- Thay đổi tech stack
- Thay đổi workflow
- Thêm/bớt team members
```

#### Anti-pattern 4: Chứa sensitive information

```markdown
❌ SAI
## API Keys
OPENAI_API_KEY=sk-xxxxx
DATABASE_URL=postgres://user:pass@host/db

✅ ĐÚNG
## Environment Variables
Xem .env.example cho danh sách env vars cần thiết.
KHÔNG commit .env files.
```

---

## TÓM TẮT PART 1

### Đã cover trong Part 1:
- [x] Giới thiệu chương trình training MangoAds
- [x] Tổng quan các thành phần Claude Code
- [x] Phân track Marketing vs Development
- [x] CLAUDE.md chi tiết
- [x] .claude/rules/ và YAML frontmatter
- [x] /init command
- [x] Memory như "Constitution"
- [x] Checklist & Anti-patterns

### Part 2 sẽ cover:
- [ ] Subagents - AI personality chuyên biệt
- [ ] Agent Skills - Khả năng tái sử dụng
- [ ] Phân biệt Subagent vs Skill
- [ ] YAML fields chi tiết
- [ ] Ví dụ thực tế MangoAds

---

**Tiếp theo:** [Part 2: Subagents & Agent Skills](./part-02-subagents-skills.md)
