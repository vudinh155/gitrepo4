# CLAUDE CODE CHO MARKETING - MANGOADS
## Part 4: Templates & Cheatsheet

**Phiên bản:** 1.0
**Đối tượng:** Team Marketing MangoAds
**Thời lượng đọc:** 20-30 phút

---

## MỤC LỤC

1. [Templates Chuẩn Marketing](#1-templates-chuẩn-marketing)
2. [Best Practices](#2-best-practices)
3. [FAQ - Câu Hỏi Thường Gặp](#3-faq---câu-hỏi-thường-gặp)
4. [Cheatsheet 1 Trang](#4-cheatsheet-1-trang)

---

## 1. TEMPLATES CHUẨN MARKETING

### 1.1 Template: CLAUDE.md cho Campaign

Sao chép và điền thông tin cho mỗi campaign mới:

```markdown
# Campaign: [TÊN CAMPAIGN]

## Thông tin cơ bản
- **Client:** [Tên client]
- **Campaign Manager:** @[tên bạn]
- **Thời gian:** [DD/MM/YYYY] - [DD/MM/YYYY]
- **Budget:** $[số tiền]/tháng
- **Objective:** [Awareness/Consideration/Conversion]

## Target Audience
- **Độ tuổi:** [range]
- **Giới tính:** [All/Male/Female]
- **Location:** [Locations]
- **Interests:** [List interests]
- **Job titles:** [nếu có]

## Channels
- [ ] Google Search
- [ ] Google Display
- [ ] Facebook
- [ ] Instagram
- [ ] LinkedIn
- [ ] TikTok
- [ ] YouTube
- [ ] Email

## KPIs
| Metric | Target | Current |
|--------|--------|---------|
| Impressions | [target] | - |
| Clicks | [target] | - |
| CTR | [target]% | - |
| Conversions | [target] | - |
| CPA | $[target] | - |
| ROAS | [target]x | - |

## UTM Convention
- **Source:** [allowed sources]
- **Medium:** [allowed mediums]
- **Campaign format:** [năm]-[quý]-[tên-campaign]
- **Example:** 2025-q1-[campaign-name]

## Tracking Setup
- **GA4 Property:** [ID]
- **Facebook Pixel:** [ID]
- **Google Ads Conversion:** [ID]
- **UTM Tracking:** ✅ Setup

## Team Contacts
- Account Manager: @[name]
- Designer: @[name]
- Client Contact: [email]

## Notes
[Thêm notes quan trọng về campaign]
```

### 1.2 Template: Weekly Report Request

Sử dụng template này khi yêu cầu Claude tạo report:

```
Tạo weekly report với thông tin sau:

**Campaign:** [Tên campaign]
**Period:** [Start date] - [End date]

**This Week:**
- Impressions: [số]
- Clicks: [số]
- Conversions: [số]
- Spend: $[số]
- Revenue: $[số] (nếu có)

**Last Week:**
- Impressions: [số]
- Clicks: [số]
- Conversions: [số]
- Spend: $[số]
- Revenue: $[số] (nếu có)

**Channels breakdown:**
- [Channel 1]: Spend $X, Conversions Y
- [Channel 2]: Spend $X, Conversions Y

**Notes:** [Bất kỳ context quan trọng nào]
```

### 1.3 Template: UTM Request

```
Tạo UTM link:
- URL: [base URL]
- Source: [google/facebook/instagram/linkedin/email/...]
- Medium: [cpc/social/email/display/...]
- Campaign: [mô tả campaign]
- Content: [variant nếu có]
- Term: [keywords nếu có]
```

### 1.4 Template: Content Brief Request

```
Tạo content brief:
- Topic: [chủ đề]
- Type: [blog/social/email/landing page]
- Target audience: [mô tả]
- Goal: [awareness/traffic/leads/sales]
- Primary keyword: [keyword chính]
- Word count: [số từ mong muốn]
- Tone: [professional/casual/friendly/...]
- CTA: [hành động mong muốn]
```

### 1.5 Template: Campaign Analysis Request

```
Phân tích campaign performance:

**Campaign:** [Tên]
**Period:** [Thời gian]

**Data:**
| Channel/Ad Set | Spend | Impressions | Clicks | Conversions |
|----------------|-------|-------------|--------|-------------|
| [Name 1] | $X | X | X | X |
| [Name 2] | $X | X | X | X |
| [Name 3] | $X | X | X | X |

**Questions:**
1. Ad set/channel nào perform tốt nhất?
2. Nên allocate budget như thế nào?
3. Có issues gì cần fix không?
```

---

## 2. BEST PRACTICES

### 2.1 Dos and Don'ts

#### ✅ NÊN LÀM

| # | Practice | Ví dụ |
|---|----------|-------|
| 1 | Cung cấp context đầy đủ | "Campaign cho client ABC, target SMEs..." |
| 2 | Dùng số liệu cụ thể | "CTR 2.3%, spend $5,000" |
| 3 | Nêu rõ output mong muốn | "Tạo report dạng markdown với table" |
| 4 | Check lại kết quả | Verify số liệu trước khi gửi client |
| 5 | Lưu templates hay | Tạo library prompts hiệu quả |

#### ❌ KHÔNG NÊN LÀM

| # | Practice | Tại sao |
|---|----------|---------|
| 1 | Yêu cầu mơ hồ | Claude sẽ assume sai |
| 2 | Skip verification | Số liệu có thể sai |
| 3 | Share sensitive data | Client data cần protect |
| 4 | Trust 100% output | Luôn double-check |
| 5 | Dùng cho final copy | Claude draft, bạn refine |

### 2.2 Workflow Tối Ưu

```
┌─────────────────────────────────────────────────────┐
│  MARKETING WORKFLOW VỚI CLAUDE CODE                 │
│                                                     │
│  1. SETUP (1 lần/campaign)                         │
│     └── Tạo CLAUDE.md với campaign info            │
│                                                     │
│  2. DAILY TASKS                                     │
│     ├── Check metrics                              │
│     ├── Generate UTM links                         │
│     └── Draft social posts                         │
│                                                     │
│  3. WEEKLY TASKS                                    │
│     ├── /weekly-report                             │
│     ├── Analyze performance                        │
│     └── Optimize budget allocation                 │
│                                                     │
│  4. MONTHLY TASKS                                   │
│     ├── Content calendar                           │
│     ├── Competitor analysis                        │
│     └── Strategy review                            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2.3 Security Reminders

```
⚠️ SECURITY CHECKLIST

□ KHÔNG share password hoặc API keys với Claude
□ KHÔNG paste full client data (anonymize nếu cần)
□ KHÔNG share sensitive financial information
□ Luôn review output trước khi gửi client
□ Dùng staging/test data khi có thể
```

---

## 3. FAQ - CÂU HỎI THƯỜNG GẶP

### Q1: Claude có thể access Google Analytics không?

**A:** Không trực tiếp. Bạn cần:
1. Export data từ GA4
2. Paste vào Claude
3. Yêu cầu phân tích

### Q2: Làm sao để Claude nhớ thông tin campaign?

**A:** Tạo file CLAUDE.md với thông tin campaign. Claude sẽ tự động đọc mỗi khi bạn làm việc.

### Q3: Output của Claude có chính xác 100% không?

**A:** Không. Luôn verify:
- Số liệu tính toán
- Facts và statistics
- Recommendations (dùng judgment của bạn)

### Q4: Có thể dùng Claude để viết content cuối cùng không?

**A:** Claude tốt cho:
- Draft đầu tiên
- Ideas và structure
- Variations để test

Nhưng bạn nên:
- Review và refine
- Thêm brand voice
- Fact-check

### Q5: Claude có thể schedule posts không?

**A:** Không. Claude chỉ tạo content. Bạn vẫn cần dùng tools như Buffer, Hootsuite để schedule.

### Q6: Làm sao khi Claude trả lời sai?

**A:**
1. Clarify yêu cầu của bạn
2. Cung cấp thêm context
3. Chỉ ra lỗi cụ thể: "CTR này tính sai, đáng lẽ phải là..."

### Q7: Có thể dùng tiếng Việt không?

**A:** Có! Claude hiểu tiếng Việt rất tốt. Bạn có thể:
- Yêu cầu bằng tiếng Việt
- Nhận output bằng tiếng Việt
- Mix 2 ngôn ngữ

---

## 4. CHEATSHEET 1 TRANG

```
╔══════════════════════════════════════════════════════════════════════════════╗
║              CLAUDE CODE CHO MARKETING - CHEATSHEET                          ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  🚀 KHỞI ĐỘNG                                                                ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ claude          │ Mở Claude Code                                      │ ║
║  │ /init           │ Tạo CLAUDE.md cho project                           │ ║
║  │ /help           │ Xem hướng dẫn                                       │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  📊 COMMANDS MARKETING                                                       ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ /weekly-report  │ Tạo báo cáo tuần                                    │ ║
║  │ /utm            │ Sinh UTM link                                       │ ║
║  │ /content-brief  │ Tạo content brief                                   │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  📈 METRICS FORMULAS                                                         ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ CTR = Clicks / Impressions × 100                                      │ ║
║  │ CVR = Conversions / Clicks × 100                                      │ ║
║  │ CPC = Spend / Clicks                                                  │ ║
║  │ CPA = Spend / Conversions                                             │ ║
║  │ ROAS = Revenue / Spend                                                │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  🎯 BENCHMARK (THAM KHẢO)                                                    ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ CTR Search: 2-5%    │ CTR Display: 0.5-1%   │ CTR Social: 1-2%        │ ║
║  │ CVR Lead Gen: 5-10% │ CVR E-com: 2-4%       │ ROAS Good: 3-5x         │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  🔗 UTM CONVENTION MANGOADS                                                  ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ utm_source   │ google, facebook, instagram, linkedin, email           │ ║
║  │ utm_medium   │ cpc, social, paid_social, email, display               │ ║
║  │ utm_campaign │ 2025-q1-[campaign-name] (lowercase, hyphens)           │ ║
║  │ utm_content  │ [variant-name] (optional)                              │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  📝 PROMPTS HIỆU QUẢ                                                        ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ "Phân tích campaign [X] với data: [paste data]"                       │ ║
║  │ "Tạo UTM cho [URL], source [X], medium [Y], campaign [Z]"            │ ║
║  │ "So sánh performance tuần này vs tuần trước: [data]"                  │ ║
║  │ "Viết 3 variations cho social post về [topic]"                        │ ║
║  │ "Optimize email subject line: [current subject]"                      │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  ⚠️ NHẮC NHỞ                                                                ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ ✓ Luôn verify số liệu trước khi gửi client                           │ ║
║  │ ✓ Không share passwords hoặc API keys                                 │ ║
║  │ ✓ CLAUDE.md = bộ nhớ campaign                                        │ ║
║  │ ✓ Claude draft → Bạn refine                                          │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
║  🆘 CẦN HELP?                                                               ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ Trong Claude Code: /help                                              │ ║
║  │ Team support: #claude-code-marketing (Slack)                          │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## TỔNG KẾT CHƯƠNG TRÌNH CLAUDE FOR MARKETING

### Bạn đã học:

**Part 1: Giới Thiệu & Cơ Bản**
- [x] Claude Code là gì
- [x] Cách sử dụng cơ bản
- [x] CLAUDE.md - bộ nhớ campaign

**Part 2: Skills & Commands**
- [x] UTM Generator
- [x] Metric Calculator
- [x] Weekly Report Command
- [x] Content Brief Command

**Part 3: Use Cases & Labs**
- [x] 6 Use Cases thực tế
- [x] Campaign Analysis workflow
- [x] Hands-on lab hoàn chỉnh

**Part 4: Templates & Cheatsheet**
- [x] Templates chuẩn MangoAds
- [x] Best Practices
- [x] FAQ
- [x] Cheatsheet 1 trang

### Bước tiếp theo:

1. **Thực hành ngay** với campaign hiện tại
2. **Tạo CLAUDE.md** cho ít nhất 1 campaign
3. **Thử weekly report** với data thật
4. **Share feedback** với team

---

*Chương trình này được thiết kế cho Team Marketing MangoAds*
*Cập nhật: Tháng 12/2025*
