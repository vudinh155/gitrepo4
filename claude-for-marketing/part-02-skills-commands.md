# CLAUDE CODE CHO MARKETING - MANGOADS
## Part 2: Skills & Commands Thực Hành

**Phiên bản:** 1.0
**Đối tượng:** Team Marketing MangoAds
**Thời lượng:** 45-60 phút

---

## MỤC LỤC

1. [UTM Generator Skill](#1-utm-generator-skill)
2. [Metric Calculator Skill](#2-metric-calculator-skill)
3. [Weekly Report Command](#3-weekly-report-command)
4. [Content Brief Command](#4-content-brief-command)
5. [Thực hành](#5-thực-hành)

---

## 1. UTM GENERATOR SKILL

### 1.1 UTM là gì? (Ôn nhanh)

**UTM (Urchin Tracking Module)** là các tham số thêm vào URL để tracking nguồn traffic trong Google Analytics.

```
https://mangoads.com/services
                ↓
https://mangoads.com/services?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-brand
```

### 1.2 Vấn đề thường gặp

| Vấn đề | Hậu quả |
|--------|---------|
| UTM không nhất quán | Data trong GA4 bị phân tán |
| Typo trong parameter | Tracking sai hoàn toàn |
| Quên UTM | Không biết traffic từ đâu |
| Format khác nhau mỗi người | Report không chính xác |

### 1.3 UTM Skill giải quyết như thế nào?

Skill này sẽ **tự động validate** và **sinh UTM đúng chuẩn MangoAds**.

### 1.4 Quy chuẩn UTM MangoAds

| Parameter | Format | Ví dụ |
|-----------|--------|-------|
| `utm_source` | lowercase, predefined | google, facebook, linkedin |
| `utm_medium` | lowercase, predefined | cpc, social, email |
| `utm_campaign` | năm-quý-tên | 2025-q1-brand-awareness |
| `utm_content` | kebab-case (optional) | header-cta, sidebar-banner |
| `utm_term` | lowercase (optional) | marketing+agency |

**Allowed Sources:**
```
google, facebook, instagram, linkedin, twitter,
tiktok, youtube, email, newsletter, direct, referral
```

**Allowed Mediums:**
```
cpc, ppc, social, paid_social, organic_social,
email, newsletter, display, banner, video, affiliate
```

### 1.5 Cách sử dụng

**Cách 1: Yêu cầu trực tiếp**
```
Bạn: Tạo UTM link cho trang mangoads.com/ebook
     Source: facebook
     Medium: paid_social
     Campaign: Q1 Lead Gen
     Content: header-cta

Claude: Đây là UTM link theo chuẩn MangoAds:

https://mangoads.com/ebook?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-lead-gen&utm_content=header-cta
```

**Cách 2: Dùng /utm command**
```
> /utm mangoads.com/ebook facebook paid_social "Q1 Lead Gen" header-cta

→ https://mangoads.com/ebook?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-lead-gen&utm_content=header-cta
```

### 1.6 Skill tự động validate

```
Bạn: Tạo UTM với source là "Social Media"

Claude: ⚠️ "Social Media" không phải source hợp lệ.
        Gợi ý: facebook, instagram, linkedin, twitter

        Bạn muốn dùng source nào?
```

---

## 2. METRIC CALCULATOR SKILL

### 2.1 Các metrics quan trọng

| Metric | Công thức | Ý nghĩa |
|--------|-----------|---------|
| **CTR** | Clicks / Impressions × 100 | % người click sau khi thấy ad |
| **CVR** | Conversions / Clicks × 100 | % người convert sau khi click |
| **CPC** | Spend / Clicks | Chi phí mỗi click |
| **CPA** | Spend / Conversions | Chi phí mỗi conversion |
| **ROAS** | Revenue / Spend | Return trên mỗi đồng chi |

### 2.2 Cách sử dụng

**Ví dụ 1: Tính CTR**
```
Bạn: Campaign có 50,000 impressions và 1,250 clicks. CTR là bao nhiêu?

Claude: CTR = 1,250 / 50,000 × 100 = 2.5%

        📊 Đánh giá: TỐT
        (Benchmark: CTR > 2% được coi là tốt cho display ads)
```

**Ví dụ 2: Phân tích campaign**
```
Bạn: Phân tích performance:
     - Impressions: 100,000
     - Clicks: 2,500
     - Conversions: 125
     - Spend: $5,000
     - Revenue: $20,000

Claude: 📊 Phân tích Campaign:

        | Metric | Giá trị | Đánh giá |
        |--------|---------|----------|
        | CTR    | 2.5%    | 🟢 Tốt   |
        | CVR    | 5.0%    | 🟢 Tốt   |
        | CPC    | $2.00   | 🟡 Trung bình |
        | CPA    | $40.00  | 🟡 Trung bình |
        | ROAS   | 4.0x    | 🟢 Tốt   |

        💡 Nhận xét:
        - CTR và CVR rất tốt, cho thấy ad và landing page hiệu quả
        - ROAS 4x nghĩa là mỗi $1 chi ra thu về $4
        - Có thể tăng budget vì ROAS đang tốt
```

### 2.3 Benchmark tham khảo

```
┌─────────────────────────────────────────────────────┐
│  BENCHMARK METRICS                                  │
│                                                     │
│  CTR (Click-Through Rate)                          │
│  ├── Search Ads: 2-5% (Tốt: >3%)                   │
│  ├── Display Ads: 0.5-1% (Tốt: >0.5%)              │
│  └── Social Ads: 1-2% (Tốt: >1.5%)                 │
│                                                     │
│  CVR (Conversion Rate)                             │
│  ├── E-commerce: 2-4%                              │
│  ├── Lead Gen: 5-10%                               │
│  └── SaaS: 3-7%                                    │
│                                                     │
│  ROAS (Return on Ad Spend)                         │
│  ├── Break-even: 1.0x                              │
│  ├── Good: 3-4x                                    │
│  └── Excellent: >5x                                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 3. WEEKLY REPORT COMMAND

### 3.1 Vấn đề với report thủ công

- ⏰ Mất 3-4 tiếng mỗi tuần
- 📉 Format không nhất quán
- 🔢 Dễ tính sai số liệu
- 😴 Công việc lặp đi lặp lại nhàm chán

### 3.2 Weekly Report Command

Gõ `/weekly-report` và Claude sẽ:
1. Đọc data từ file hoặc bạn cung cấp
2. Tính toán tất cả metrics
3. So sánh với tuần trước
4. Đưa ra nhận xét và khuyến nghị
5. Format thành report đẹp

### 3.3 Cách sử dụng

**Bước 1:** Chuẩn bị data (copy từ Google Ads/Facebook Ads)

```
Campaign: Q1 Brand Awareness
Week: Jan 6-12, 2025

This Week:
- Impressions: 125,000
- Clicks: 2,875
- Conversions: 145
- Spend: $5,000
- Revenue: $21,000

Last Week:
- Impressions: 108,695
- Clicks: 2,174
- Conversions: 120
- Spend: $4,500
- Revenue: $17,500
```

**Bước 2:** Gọi command

```
> /weekly-report

[Paste data ở trên]
```

**Bước 3:** Nhận report

```markdown
# Weekly Report: Q1 Brand Awareness
**Period:** Jan 6-12, 2025
**Prepared by:** AI Assistant

## 📋 Executive Summary
- 🟢 Impressions tăng 15% so với tuần trước
- 🟢 CTR tăng từ 2.0% lên 2.3%
- 🟢 ROAS tăng từ 3.9x lên 4.2x
- 💡 Recommend: Tăng budget 20% cho tuần tới

## 📊 Performance Metrics

| Metric | This Week | Last Week | Change | Status |
|--------|-----------|-----------|--------|--------|
| Impressions | 125,000 | 108,695 | +15% | 🟢 |
| Clicks | 2,875 | 2,174 | +32% | 🟢 |
| CTR | 2.30% | 2.00% | +15% | 🟢 |
| Conversions | 145 | 120 | +21% | 🟢 |
| CVR | 5.04% | 5.52% | -9% | 🟡 |
| Spend | $5,000 | $4,500 | +11% | - |
| Revenue | $21,000 | $17,500 | +20% | 🟢 |
| CPA | $34.48 | $37.50 | -8% | 🟢 |
| ROAS | 4.20x | 3.89x | +8% | 🟢 |

## 💡 Insights & Recommendations

### Điểm tốt
1. CTR tăng 15% cho thấy creative đang hiệu quả
2. ROAS vượt target (4.0x), có thể scale

### Cần cải thiện
1. CVR giảm nhẹ - kiểm tra landing page load speed
2. CPC tăng - cân nhắc điều chỉnh bidding strategy

### Action Items
- [ ] Tăng budget 20% cho tuần tới
- [ ] A/B test landing page mới
- [ ] Review search terms report
```

---

## 4. CONTENT BRIEF COMMAND

### 4.1 Content Brief là gì?

Content Brief là tài liệu hướng dẫn cho content writer biết:
- Viết về gì
- Viết cho ai
- Tone như thế nào
- Keywords cần có
- CTA là gì

### 4.2 Sử dụng /content-brief

**Input:**
```
> /content-brief

Topic: "Digital Marketing Trends 2025"
Type: Blog post
Target: Marketing Managers tại SME
Goal: Lead generation (download ebook)
```

**Output:**
```markdown
# Content Brief: Digital Marketing Trends 2025

## Overview
| Field | Value |
|-------|-------|
| Content Type | Blog Post |
| Target Audience | Marketing Managers at SMEs |
| Goal | Lead Generation |
| Word Count | 1,500-2,000 words |
| Deadline | [TBD] |

## Target Audience Profile
- Job Title: Marketing Manager, Digital Marketing Lead
- Company Size: 50-500 employees
- Pain Points: Limited budget, need ROI proof
- Goals: Stay updated, find practical tips

## SEO Requirements
### Primary Keyword
`digital marketing trends 2025`

### Secondary Keywords
- AI in marketing 2025
- social media trends 2025
- content marketing trends
- marketing automation

### Meta Description (150-160 chars)
"Discover the top digital marketing trends for 2025. Learn how AI, social commerce, and personalization will shape your marketing strategy."

## Content Structure

### H1: Digital Marketing Trends 2025: What You Need to Know

### Introduction (150 words)
- Hook: Surprising statistic about marketing changes
- Problem: Staying ahead of trends
- Promise: 7 actionable trends for 2025

### H2: 1. AI-Powered Marketing Automation
- Key points...
- Example...

### H2: 2. Social Commerce Growth
- Key points...
- Example...

[Continue for all 7 trends...]

### Conclusion (150 words)
- Summary of key points
- CTA: Download full ebook

## CTA
- Primary: "Download our 2025 Marketing Playbook"
- Secondary: "Subscribe for weekly marketing tips"

## References to Include
- Industry reports (Gartner, HubSpot)
- Statistics from reputable sources
- Case studies if available

## Tone & Style
- Professional but accessible
- Data-driven
- Actionable tips
- No jargon

## Visual Elements Needed
- Featured image
- Infographic: "7 Trends at a Glance"
- Screenshots or examples
```

---

## 5. THỰC HÀNH

### Lab 1: Tạo UTM Links (15 phút)

**Bài tập:** Tạo UTM links cho các trường hợp sau:

| # | URL | Source | Medium | Campaign | Content |
|---|-----|--------|--------|----------|---------|
| 1 | mangoads.com/services | google | cpc | Q1 Services | - |
| 2 | mangoads.com/ebook | facebook | paid_social | Q1 Lead Gen | header-banner |
| 3 | mangoads.com/blog/post | newsletter | email | Weekly Digest W1 | - |

**Cách làm:**
```
> Tạo UTM link cho mangoads.com/services, source google, medium cpc, campaign Q1 Services
```

**Kết quả mong đợi:**
```
1. https://mangoads.com/services?utm_source=google&utm_medium=cpc&utm_campaign=2025-q1-services

2. https://mangoads.com/ebook?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-lead-gen&utm_content=header-banner

3. https://mangoads.com/blog/post?utm_source=newsletter&utm_medium=email&utm_campaign=2025-q1-weekly-digest-w1
```

### Lab 2: Phân tích Campaign Metrics (15 phút)

**Bài tập:** Phân tích performance của campaign với data:

```
Campaign: Facebook Lead Gen
Period: Jan 1-7, 2025

- Impressions: 80,000
- Clicks: 1,600
- Form Submissions: 120
- Spend: $2,400
- (Giá trị mỗi lead: $50)
```

**Cách làm:**
```
> Phân tích campaign với data sau:
[Paste data]
Giá trị mỗi lead là $50
```

**Kết quả mong đợi:**
- CTR: 2.0% (Tốt)
- CVR: 7.5% (Rất tốt cho lead gen)
- CPL: $20 (Tốt, thấp hơn giá trị lead)
- ROAS: 2.5x ($6,000 revenue / $2,400 spend)

### Lab 3: Tạo Weekly Report (20 phút)

**Bài tập:** Tạo weekly report cho campaign với data thực tế của bạn (hoặc dùng data mẫu):

```
> /weekly-report

[Input your campaign data]
```

**Checklist kết quả:**
- [ ] Có Executive Summary
- [ ] Có bảng metrics với Week-over-Week comparison
- [ ] Có status indicators (🟢🟡🔴)
- [ ] Có Insights & Recommendations
- [ ] Có Action Items

---

## CHECKLIST HOÀN THÀNH PART 2

### Kiến thức

- [ ] Hiểu chuẩn UTM MangoAds
- [ ] Biết các metrics quan trọng và cách tính
- [ ] Biết cách dùng /weekly-report
- [ ] Biết cách dùng /content-brief

### Thực hành

- [ ] Tạo được 3 UTM links đúng chuẩn
- [ ] Phân tích được campaign metrics
- [ ] Tạo được 1 weekly report

---

## TÓM TẮT PART 2

### Đã học:
- [x] UTM Generator Skill - sinh link tracking
- [x] Metric Calculator Skill - tính và đánh giá metrics
- [x] Weekly Report Command - báo cáo tuần tự động
- [x] Content Brief Command - tạo brief cho content

### Part 3 sẽ học:
- [ ] Use Cases thực tế tại MangoAds
- [ ] Campaign Analysis workflow
- [ ] SEO Audit basics
- [ ] Social Media Content Calendar

---

**Tiếp theo:** [Part 3: Use Cases & Labs Thực Tế](./part-03-usecases-labs.md)
