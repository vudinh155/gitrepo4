# CHƯƠNG TRÌNH TRAINING CLAUDE CODE - MANGOADS
## Part 8: Ví Dụ Kỹ Thuật Chi Tiết

**Phiên bản:** 1.0
**Cập nhật:** Tháng 12/2025
**Prerequisite:** Đã hoàn thành Part 1-7

---

## MỤC LỤC PART 8

1. [Project Setup: Next.js + Tailwind + shadcn/ui](#1-project-setup-nextjs--tailwind--shadcnui)
2. [Campaign Brief Form với React JSON Schema Form](#2-campaign-brief-form-với-react-json-schema-form)
3. [Subagent + MCP Test Page Sinh Ra](#3-subagent--mcp-test-page-sinh-ra)
4. [Full Integration Example](#4-full-integration-example)

---

## 1. PROJECT SETUP: NEXT.JS + TAILWIND + SHADCN/UI

### 1.1 CLAUDE.md cho Project

```markdown
# CLAUDE.md - MangoAds Campaign Platform

## Project Overview
Internal platform for managing marketing campaigns.
Tech stack: Next.js 14, TypeScript, Tailwind CSS, shadcn/ui, React JSON Schema Form.

## Commands
- Dev: `pnpm dev` (port 3000)
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`
- Type check: `pnpm typecheck`

## Directory Structure
```
src/
├── app/                    # Next.js App Router
│   ├── (dashboard)/       # Dashboard routes
│   ├── api/               # API routes
│   └── layout.tsx
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── forms/             # RJSF form components
│   └── campaign/          # Campaign-specific components
├── lib/
│   ├── schemas/           # JSON schemas for forms
│   └── utils.ts           # Utility functions
└── types/                 # TypeScript types
```

## Coding Conventions
- Use TypeScript strict mode
- Components: PascalCase, named exports
- Files: kebab-case
- Use shadcn/ui for all UI components
- Form validation via JSON Schema

## Component Patterns

### Basic Component
```tsx
import { cn } from "@/lib/utils"

interface ComponentProps {
  className?: string
  children: React.ReactNode
}

export function Component({ className, children }: ComponentProps) {
  return (
    <div className={cn("base-styles", className)}>
      {children}
    </div>
  )
}
```

### Form Component (RJSF)
```tsx
import Form from "@rjsf/core"
import validator from "@rjsf/validator-ajv8"

export function CampaignForm({ schema, onSubmit }) {
  return (
    <Form
      schema={schema}
      validator={validator}
      onSubmit={onSubmit}
    />
  )
}
```

## API Patterns
- Use Next.js Route Handlers
- Return JSON with { success, data, error }
- Validate input with Zod
```

### 1.2 Rules cho shadcn/ui

**File: `.claude/rules/shadcn-ui.md`**

```markdown
---
paths: ["src/components/**/*.tsx", "src/app/**/*.tsx"]
---

# shadcn/ui Rules

## Component Usage
- Always use shadcn/ui components instead of custom implementations
- Import from @/components/ui/
- Never modify files in ui/ directory directly

## Available Components
Button, Card, Dialog, Form, Input, Label, Select, Textarea,
Tabs, Table, Toast, Tooltip, Popover, Sheet, Badge, Alert

## Styling
- Use cn() for conditional classes
- Follow Tailwind conventions
- Dark mode: Use CSS variables

## Form Components
- Use Form component with react-hook-form
- Use FormField, FormItem, FormLabel, FormControl, FormMessage

## Example
```tsx
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Card, CardHeader, CardContent } from "@/components/ui/card"

export function MyComponent() {
  return (
    <Card>
      <CardHeader>Title</CardHeader>
      <CardContent>
        <Input placeholder="Enter text" />
        <Button>Submit</Button>
      </CardContent>
    </Card>
  )
}
```
```

### 1.3 Rules cho React JSON Schema Form

**File: `.claude/rules/rjsf.md`**

```markdown
---
paths: ["src/components/forms/**/*.tsx", "src/lib/schemas/**/*.ts"]
---

# React JSON Schema Form (RJSF) Rules

## Package
Using @rjsf/core with @rjsf/validator-ajv8

## Schema Location
All JSON schemas in src/lib/schemas/

## Schema Structure
```typescript
// src/lib/schemas/campaign-brief.schema.ts
export const campaignBriefSchema = {
  type: "object",
  required: ["campaignName", "objective", "budget"],
  properties: {
    campaignName: {
      type: "string",
      title: "Campaign Name",
      minLength: 3
    },
    objective: {
      type: "string",
      title: "Objective",
      enum: ["awareness", "consideration", "conversion"]
    },
    budget: {
      type: "number",
      title: "Budget (USD)",
      minimum: 100
    }
  }
}
```

## UI Schema
```typescript
export const campaignBriefUiSchema = {
  objective: {
    "ui:widget": "select",
    "ui:placeholder": "Select objective"
  },
  budget: {
    "ui:widget": "updown"
  }
}
```

## Custom Widgets
- Use shadcn/ui components as custom widgets
- Register in widgets prop

## Form Component Pattern
```tsx
import Form from "@rjsf/core"
import validator from "@rjsf/validator-ajv8"
import { campaignBriefSchema, campaignBriefUiSchema } from "@/lib/schemas/campaign-brief.schema"

export function CampaignBriefForm({ onSubmit }) {
  return (
    <Form
      schema={campaignBriefSchema}
      uiSchema={campaignBriefUiSchema}
      validator={validator}
      onSubmit={({ formData }) => onSubmit(formData)}
    />
  )
}
```
```

---

## 2. CAMPAIGN BRIEF FORM VỚI REACT JSON SCHEMA FORM

### 2.1 JSON Schema Definition

**File: `src/lib/schemas/campaign-brief.schema.ts`**

```typescript
import { JSONSchema7 } from "json-schema"

export const campaignBriefSchema: JSONSchema7 = {
  type: "object",
  title: "Campaign Brief",
  description: "Create a new marketing campaign brief",
  required: [
    "campaignName",
    "client",
    "objective",
    "targetAudience",
    "budget",
    "startDate",
    "endDate"
  ],
  properties: {
    // Basic Info
    campaignName: {
      type: "string",
      title: "Campaign Name",
      description: "Use format: [Year]-[Quarter]-[Description]",
      minLength: 5,
      maxLength: 100,
      pattern: "^[0-9]{4}-Q[1-4]-.*$"
    },
    client: {
      type: "string",
      title: "Client",
      minLength: 2
    },

    // Objectives
    objective: {
      type: "string",
      title: "Primary Objective",
      enum: ["awareness", "consideration", "conversion", "retention"],
      enumNames: [
        "Brand Awareness",
        "Consideration",
        "Conversion",
        "Customer Retention"
      ]
    },
    secondaryObjectives: {
      type: "array",
      title: "Secondary Objectives",
      items: {
        type: "string",
        enum: ["traffic", "engagement", "leads", "sales", "app_installs"]
      },
      uniqueItems: true
    },

    // Target Audience
    targetAudience: {
      type: "object",
      title: "Target Audience",
      required: ["ageRange", "location"],
      properties: {
        ageRange: {
          type: "object",
          title: "Age Range",
          properties: {
            min: { type: "integer", minimum: 13, maximum: 65, default: 25 },
            max: { type: "integer", minimum: 18, maximum: 65, default: 45 }
          }
        },
        gender: {
          type: "string",
          title: "Gender",
          enum: ["all", "male", "female"],
          default: "all"
        },
        location: {
          type: "array",
          title: "Locations",
          items: {
            type: "string",
            enum: ["vietnam", "thailand", "singapore", "malaysia", "indonesia"]
          },
          minItems: 1
        },
        interests: {
          type: "array",
          title: "Interests",
          items: { type: "string" }
        }
      }
    },

    // Budget & Timeline
    budget: {
      type: "object",
      title: "Budget",
      required: ["total", "currency"],
      properties: {
        total: {
          type: "number",
          title: "Total Budget",
          minimum: 500
        },
        currency: {
          type: "string",
          title: "Currency",
          enum: ["USD", "VND", "THB", "SGD"],
          default: "USD"
        },
        dailyBudget: {
          type: "number",
          title: "Daily Budget (optional)",
          minimum: 10
        }
      }
    },
    startDate: {
      type: "string",
      title: "Start Date",
      format: "date"
    },
    endDate: {
      type: "string",
      title: "End Date",
      format: "date"
    },

    // Channels
    channels: {
      type: "array",
      title: "Marketing Channels",
      items: {
        type: "string",
        enum: [
          "google_search",
          "google_display",
          "facebook",
          "instagram",
          "linkedin",
          "tiktok",
          "youtube",
          "email"
        ]
      },
      minItems: 1,
      uniqueItems: true
    },

    // KPIs
    kpis: {
      type: "array",
      title: "Key Performance Indicators",
      items: {
        type: "object",
        required: ["metric", "target"],
        properties: {
          metric: {
            type: "string",
            enum: ["impressions", "clicks", "ctr", "conversions", "cpa", "roas"]
          },
          target: {
            type: "number"
          },
          unit: {
            type: "string"
          }
        }
      }
    },

    // Additional Info
    notes: {
      type: "string",
      title: "Additional Notes"
    }
  }
}

export const campaignBriefUiSchema = {
  campaignName: {
    "ui:placeholder": "2025-Q1-Product-Launch",
    "ui:help": "Format: YYYY-QN-Description"
  },
  objective: {
    "ui:widget": "radio"
  },
  secondaryObjectives: {
    "ui:widget": "checkboxes"
  },
  targetAudience: {
    location: {
      "ui:widget": "checkboxes"
    },
    interests: {
      "ui:options": {
        addable: true
      }
    }
  },
  budget: {
    total: {
      "ui:widget": "updown"
    }
  },
  startDate: {
    "ui:widget": "date"
  },
  endDate: {
    "ui:widget": "date"
  },
  channels: {
    "ui:widget": "checkboxes"
  },
  kpis: {
    "ui:options": {
      orderable: true
    }
  },
  notes: {
    "ui:widget": "textarea",
    "ui:options": {
      rows: 4
    }
  }
}
```

### 2.2 Form Component với shadcn/ui Widgets

**File: `src/components/forms/campaign-brief-form.tsx`**

```tsx
"use client"

import { useState } from "react"
import Form, { IChangeEvent } from "@rjsf/core"
import validator from "@rjsf/validator-ajv8"
import {
  campaignBriefSchema,
  campaignBriefUiSchema
} from "@/lib/schemas/campaign-brief.schema"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue
} from "@/components/ui/select"
import { Textarea } from "@/components/ui/textarea"
import { Checkbox } from "@/components/ui/checkbox"
import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group"
import { cn } from "@/lib/utils"

// Custom widgets using shadcn/ui
const CustomTextWidget = (props: any) => {
  return (
    <Input
      type="text"
      id={props.id}
      value={props.value || ""}
      onChange={(e) => props.onChange(e.target.value)}
      placeholder={props.placeholder}
      required={props.required}
      className={cn(props.rawErrors?.length > 0 && "border-red-500")}
    />
  )
}

const CustomSelectWidget = (props: any) => {
  return (
    <Select
      value={props.value}
      onValueChange={props.onChange}
    >
      <SelectTrigger>
        <SelectValue placeholder={props.placeholder || "Select..."} />
      </SelectTrigger>
      <SelectContent>
        {props.options.enumOptions?.map((option: any) => (
          <SelectItem key={option.value} value={option.value}>
            {option.label}
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  )
}

const CustomTextareaWidget = (props: any) => {
  return (
    <Textarea
      id={props.id}
      value={props.value || ""}
      onChange={(e) => props.onChange(e.target.value)}
      placeholder={props.placeholder}
      rows={props.options?.rows || 3}
    />
  )
}

const CustomCheckboxWidget = (props: any) => {
  return (
    <div className="flex items-center space-x-2">
      <Checkbox
        id={props.id}
        checked={props.value || false}
        onCheckedChange={props.onChange}
      />
      <Label htmlFor={props.id}>{props.label}</Label>
    </div>
  )
}

// Custom field template
const CustomFieldTemplate = (props: any) => {
  const { id, label, required, rawErrors, children, help, description } = props

  return (
    <div className="mb-4">
      {label && (
        <Label htmlFor={id} className="mb-2 block">
          {label}
          {required && <span className="text-red-500 ml-1">*</span>}
        </Label>
      )}
      {description && (
        <p className="text-sm text-muted-foreground mb-2">{description}</p>
      )}
      {children}
      {help && (
        <p className="text-xs text-muted-foreground mt-1">{help}</p>
      )}
      {rawErrors?.length > 0 && (
        <ul className="text-sm text-red-500 mt-1">
          {rawErrors.map((error: string, i: number) => (
            <li key={i}>{error}</li>
          ))}
        </ul>
      )}
    </div>
  )
}

// Widgets map
const widgets = {
  TextWidget: CustomTextWidget,
  SelectWidget: CustomSelectWidget,
  TextareaWidget: CustomTextareaWidget,
  CheckboxWidget: CustomCheckboxWidget,
}

// Form templates
const templates = {
  FieldTemplate: CustomFieldTemplate,
}

interface CampaignBriefFormProps {
  initialData?: any
  onSubmit: (data: any) => void
  onCancel?: () => void
}

export function CampaignBriefForm({
  initialData,
  onSubmit,
  onCancel
}: CampaignBriefFormProps) {
  const [formData, setFormData] = useState(initialData || {})

  const handleChange = (e: IChangeEvent) => {
    setFormData(e.formData)
  }

  const handleSubmit = (e: IChangeEvent) => {
    onSubmit(e.formData)
  }

  return (
    <Card className="w-full max-w-4xl mx-auto">
      <CardHeader>
        <CardTitle>Campaign Brief</CardTitle>
      </CardHeader>
      <CardContent>
        <Form
          schema={campaignBriefSchema}
          uiSchema={campaignBriefUiSchema}
          formData={formData}
          validator={validator}
          widgets={widgets}
          templates={templates}
          onChange={handleChange}
          onSubmit={handleSubmit}
          className="space-y-6"
        >
          <div className="flex justify-end space-x-4 pt-6 border-t">
            {onCancel && (
              <Button type="button" variant="outline" onClick={onCancel}>
                Cancel
              </Button>
            )}
            <Button type="submit">
              Create Campaign Brief
            </Button>
          </div>
        </Form>
      </CardContent>
    </Card>
  )
}
```

### 2.3 Page Component

**File: `src/app/(dashboard)/campaigns/new/page.tsx`**

```tsx
"use client"

import { useRouter } from "next/navigation"
import { CampaignBriefForm } from "@/components/forms/campaign-brief-form"
import { toast } from "@/components/ui/use-toast"

export default function NewCampaignPage() {
  const router = useRouter()

  const handleSubmit = async (data: any) => {
    try {
      const response = await fetch("/api/campaigns", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      })

      if (!response.ok) throw new Error("Failed to create campaign")

      const result = await response.json()

      toast({
        title: "Campaign created",
        description: `Campaign "${data.campaignName}" has been created successfully.`
      })

      router.push(`/campaigns/${result.data.id}`)
    } catch (error) {
      toast({
        title: "Error",
        description: "Failed to create campaign. Please try again.",
        variant: "destructive"
      })
    }
  }

  const handleCancel = () => {
    router.back()
  }

  return (
    <div className="container py-8">
      <CampaignBriefForm
        onSubmit={handleSubmit}
        onCancel={handleCancel}
      />
    </div>
  )
}
```

---

## 3. SUBAGENT + MCP TEST PAGE SINH RA

### 3.1 Subagent: Page Generator

**File: `.claude/agents/page-generator.md`**

```yaml
---
name: page-generator
description: Generates Next.js pages with shadcn/ui components and RJSF forms following MangoAds conventions
model: opus
tools: Read, Write, Edit, Glob, Grep
permissionMode: acceptEdits
skills: component-generator
---

# Page Generator - MangoAds

You are a senior Next.js developer specializing in creating pages with shadcn/ui and React JSON Schema Form.

## Capabilities
1. Generate complete page components
2. Create corresponding API routes
3. Define JSON schemas for forms
4. Set up proper TypeScript types

## File Structure to Generate
For a new feature "[feature]":

```
src/
├── app/(dashboard)/[feature]/
│   ├── page.tsx              # List view
│   ├── new/page.tsx          # Create form
│   └── [id]/
│       ├── page.tsx          # Detail view
│       └── edit/page.tsx     # Edit form
├── app/api/[feature]/
│   ├── route.ts              # GET (list), POST (create)
│   └── [id]/route.ts         # GET, PUT, DELETE
├── components/[feature]/
│   ├── [feature]-form.tsx    # RJSF form component
│   ├── [feature]-list.tsx    # List component
│   └── [feature]-card.tsx    # Card component
├── lib/schemas/
│   └── [feature].schema.ts   # JSON Schema + UI Schema
└── types/
    └── [feature].ts          # TypeScript types
```

## Conventions
- Use App Router (not Pages Router)
- Server Components by default
- "use client" only when needed
- shadcn/ui for all UI
- RJSF for dynamic forms
- Zod for API validation

## Output Format
For each file, provide:
1. File path
2. Complete code
3. Brief explanation
```

### 3.2 Subagent: Page Tester (với MCP)

**File: `.claude/agents/page-tester.md`**

```yaml
---
name: page-tester
description: Tests generated pages using Playwright MCP for form submission, navigation, and UI verification
model: sonnet
tools: Read, Glob, Grep, mcp__playwright
permissionMode: default
---

# Page Tester - MangoAds

You test generated pages thoroughly using Playwright.

## Test Categories

### 1. Navigation Tests
- Page loads without errors
- Navigation links work
- Breadcrumbs correct
- Back/forward navigation

### 2. Form Tests
- Required field validation
- Format validation
- Successful submission
- Error handling

### 3. UI Tests
- Components render correctly
- Responsive layout
- Accessibility basics
- Loading states

### 4. API Integration Tests
- Create operation
- Read operation
- Update operation
- Delete operation

## Test Flow

```
1. playwright_navigate → Go to page
2. playwright_wait → Wait for page load
3. playwright_screenshot → Capture initial state
4. playwright_fill → Fill form fields
5. playwright_click → Submit form
6. playwright_wait → Wait for response
7. playwright_get_text → Verify result
8. playwright_screenshot → Capture final state
```

## Output Format

```markdown
# Test Report: [Page Name]

## Summary
- Total tests: X
- Passed: Y
- Failed: Z

## Test Results

### Navigation Tests
| Test | Status | Notes |
|------|--------|-------|
| ... | ... | ... |

### Form Tests
| Test | Status | Notes |
|------|--------|-------|
| ... | ... | ... |

## Screenshots
[Attached screenshots]

## Issues Found
1. [Issue description + severity]
```
```

### 3.3 Workflow: Generate + Test

```
┌─────────────────────────────────────────────────────────────┐
│           GENERATE + TEST WORKFLOW                           │
│                                                              │
│  User: "Create a campaign management page"                  │
│                                                              │
│  ┌─────────────────┐                                        │
│  │  page-generator │ Subagent                               │
│  │                 │                                        │
│  │  1. Create schema                                        │
│  │  2. Generate form component                              │
│  │  3. Create page components                               │
│  │  4. Set up API routes                                    │
│  │  5. Add TypeScript types                                 │
│  │                                                          │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  Hook: Stop     │                                        │
│  │  Run tests      │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │  page-tester    │ Subagent + MCP                         │
│  │                 │                                        │
│  │  1. Start dev server                                     │
│  │  2. Navigate to new page                                 │
│  │  3. Test form submission                                 │
│  │  4. Verify CRUD operations                               │
│  │  5. Generate test report                                 │
│  │                                                          │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  Output: Generated page + Test report                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.4 Hook Configuration

**File: `.claude/settings.json`**

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If the completed subagent was 'page-generator' and files were created/modified, invoke 'page-tester' subagent to test the generated pages. Skip if this was already a testing subagent."
          }
        ]
      }
    ]
  }
}
```

---

## 4. FULL INTEGRATION EXAMPLE

### 4.1 Complete Project Structure

```
mangoads-campaign-platform/
├── .claude/
│   ├── CLAUDE.md
│   ├── rules/
│   │   ├── shadcn-ui.md
│   │   └── rjsf.md
│   ├── agents/
│   │   ├── page-generator.md
│   │   └── page-tester.md
│   ├── skills/
│   │   └── component-generator/
│   │       └── SKILL.md
│   ├── commands/
│   │   ├── generate-page.md
│   │   └── test-page.md
│   └── settings.json
├── .mcp.json
├── src/
│   ├── app/
│   │   ├── (dashboard)/
│   │   │   └── campaigns/
│   │   │       ├── page.tsx
│   │   │       ├── new/page.tsx
│   │   │       └── [id]/
│   │   │           ├── page.tsx
│   │   │           └── edit/page.tsx
│   │   ├── api/
│   │   │   └── campaigns/
│   │   │       ├── route.ts
│   │   │       └── [id]/route.ts
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/                    # shadcn/ui
│   │   └── forms/
│   │       └── campaign-brief-form.tsx
│   ├── lib/
│   │   ├── schemas/
│   │   │   └── campaign-brief.schema.ts
│   │   └── utils.ts
│   └── types/
│       └── campaign.ts
├── package.json
└── tsconfig.json
```

### 4.2 MCP Configuration

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
    }
  }
}
```

### 4.3 Slash Command: Generate Page

**File: `.claude/commands/generate-page.md`**

```markdown
---
description: Generate a complete CRUD page with form, list, and API routes
allowed-tools: Read, Write, Edit, Glob, Grep
argument-hint: "feature-name [--with-test]"
model: opus
---

# Generate Page Command

Generate a complete feature page following MangoAds conventions.

## Arguments
- feature-name: Name of the feature (kebab-case)
- --with-test: Also run tests after generation

## Generated Files
1. JSON Schema + UI Schema
2. Form component (RJSF + shadcn/ui)
3. List component
4. Detail component
5. Page routes (App Router)
6. API routes
7. TypeScript types

## Process
1. Invoke page-generator subagent
2. Generate all required files
3. If --with-test, invoke page-tester subagent

## Example
```
/generate-page campaign-analytics --with-test
```
```

### 4.4 Slash Command: Test Page

**File: `.claude/commands/test-page.md`**

```markdown
---
description: Test a page using Playwright MCP
allowed-tools: Read, Glob, Grep, mcp__playwright
argument-hint: "url [--full]"
model: sonnet
---

# Test Page Command

Run automated tests on a page using Playwright.

## Arguments
- url: Page URL to test
- --full: Run comprehensive test suite

## Test Types
- Basic: Navigation, load time, no errors
- Full: + Form submission, CRUD, responsive

## Process
1. Invoke page-tester subagent
2. Navigate to URL
3. Run test suite
4. Generate report

## Example
```
/test-page http://localhost:3000/campaigns/new --full
```
```

### 4.5 Example Usage Session

```
User: /generate-page campaign-management --with-test

Claude:
1. Invoking page-generator subagent...

[page-generator subagent]:
Creating campaign management feature...

Generated files:
- src/lib/schemas/campaign-management.schema.ts
- src/components/campaign-management/campaign-form.tsx
- src/components/campaign-management/campaign-list.tsx
- src/app/(dashboard)/campaigns/page.tsx
- src/app/(dashboard)/campaigns/new/page.tsx
- src/app/api/campaigns/route.ts

2. SubagentStop hook triggered...

3. Invoking page-tester subagent...

[page-tester subagent]:
Starting dev server: pnpm dev

Testing http://localhost:3000/campaigns...

Test Results:
✅ Page loads successfully (1.2s)
✅ Navigation to /campaigns/new works
✅ Form renders with all fields
✅ Required field validation works
✅ Form submission successful
✅ New campaign appears in list
⚠️ Mobile viewport: button slightly cut off

Test Report Generated:
- 6 tests passed
- 1 warning (non-blocking)
- Screenshots saved to /test-results/

Recommendation: Fix button padding for mobile viewport.

User: Fix the mobile button issue

Claude:
Updating campaign-form.tsx...

```tsx
// Before
<Button type="submit">Create Campaign</Button>

// After
<Button type="submit" className="w-full sm:w-auto">
  Create Campaign
</Button>
```

Re-running mobile test...
✅ Mobile viewport: button displays correctly

All tests pass!
```

---

## TÓM TẮT PART 8

### Đã cover trong Part 8:
- [x] Project setup với CLAUDE.md cho Next.js + Tailwind + shadcn/ui
- [x] Rules cho shadcn/ui và RJSF
- [x] Campaign Brief Form với React JSON Schema Form
- [x] Custom widgets với shadcn/ui components
- [x] Page Generator Subagent
- [x] Page Tester Subagent với Playwright MCP
- [x] Hook integration cho auto-testing
- [x] Full integration example với complete project structure

### Part 9 sẽ cover:
- [ ] Templates chuẩn (Subagent, Skill, Hook, Command, Plugin)
- [ ] Hands-on Labs (3+ bài)
- [ ] Lab 1: Weekly Report Subagent
- [ ] Lab 2: UTM Skill
- [ ] Lab 3: MCP Playwright Web Test

---

**Tiếp theo:** [Part 9: Templates & Hands-on Labs](./part-09-templates-labs.md)
