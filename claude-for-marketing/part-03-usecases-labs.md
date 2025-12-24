# CLAUDE CODE CHO MARKETING - MANGOADS
## Part 3: Use Cases & Hands-on Labs

**Phiên bản:** 1.0
**Đối tượng:** Team Marketing MangoAds
**Thời lượng:** 60-90 phút (bao gồm labs)

---

## MỤC LỤC

1. [Use Case 1: Campaign Performance Analysis](#use-case-1-campaign-performance-analysis)
2. [Use Case 2: Content Calendar Generator](#use-case-2-content-calendar-generator)
3. [Use Case 3: Competitor Ad Analysis](#use-case-3-competitor-ad-analysis)
4. [Use Case 4: Email Campaign Optimizer](#use-case-4-email-campaign-optimizer)
5. [Use Case 5: Social Media Post Generator](#use-case-5-social-media-post-generator)
6. [Use Case 6: SEO Content Audit](#use-case-6-seo-content-audit)
7. [Hands-on Lab: Complete Campaign Workflow](#hands-on-lab-complete-campaign-workflow)

---

## USE CASE 1: CAMPAIGN PERFORMANCE ANALYSIS

### Tình huống

Bạn cần phân tích performance của campaign Facebook Ads đang chạy và đưa ra khuyến nghị optimization.

### Cách thực hiện

**Bước 1:** Chuẩn bị data từ Facebook Ads Manager

```
Campaign: Q1 2025 Lead Generation
Period: Jan 1-14, 2025

Ad Set 1: Interest - Digital Marketing
- Spend: $1,500
- Impressions: 45,000
- Clicks: 900
- Leads: 45

Ad Set 2: Interest - Small Business
- Spend: $1,500
- Impressions: 52,000
- Clicks: 780
- Leads: 62

Ad Set 3: Lookalike - Website Visitors
- Spend: $1,000
- Impressions: 38,000
- Clicks: 950
- Leads: 76
```

**Bước 2:** Yêu cầu Claude phân tích

```
> Phân tích performance của 3 ad sets này và đưa ra khuyến nghị budget allocation:

[Paste data]
```

**Bước 3:** Nhận phân tích

```markdown
## 📊 Campaign Analysis: Q1 2025 Lead Generation

### Performance Summary

| Ad Set | Spend | CTR | CPL | Leads | Efficiency |
|--------|-------|-----|-----|-------|------------|
| Interest - Digital Marketing | $1,500 | 2.0% | $33.33 | 45 | 🟡 |
| Interest - Small Business | $1,500 | 1.5% | $24.19 | 62 | 🟢 |
| Lookalike - Website Visitors | $1,000 | 2.5% | $13.16 | 76 | 🟢🟢 |

### Key Insights

1. **Best Performer:** Lookalike - Website Visitors
   - Lowest CPL ($13.16)
   - Highest CTR (2.5%)
   - Best lead quality expected (similar to existing customers)

2. **Underperformer:** Interest - Digital Marketing
   - Highest CPL ($33.33)
   - Good CTR but low conversion rate
   - May be too broad targeting

### 💡 Recommendations

#### Budget Reallocation
| Ad Set | Current | Recommended | Change |
|--------|---------|-------------|--------|
| Digital Marketing | $1,500 | $500 | -67% |
| Small Business | $1,500 | $1,500 | 0% |
| Lookalike | $1,000 | $2,000 | +100% |

#### Action Items
- [ ] Shift $1,000 from Digital Marketing to Lookalike
- [ ] Create new Lookalike audience from leads (not just visitors)
- [ ] Test narrowing Digital Marketing with job titles
- [ ] A/B test new creatives for Small Business
```

---

## USE CASE 2: CONTENT CALENDAR GENERATOR

### Tình huống

Bạn cần lên content calendar cho tháng tới với 12 posts (3 posts/tuần).

### Cách thực hiện

```
> Tạo content calendar cho tháng 2/2025 với:
- 12 posts (Thứ 2, 4, 6)
- Theme: Digital Marketing Tips
- Mix: 40% educational, 30% promotional, 30% engagement
- Platform: LinkedIn
```

### Kết quả

```markdown
# Content Calendar - February 2025
## Theme: Digital Marketing Tips for SMEs

### Week 1 (Feb 3-7)

| Day | Type | Topic | CTA | Visual |
|-----|------|-------|-----|--------|
| Mon 3 | 📚 Educational | "5 Marketing Metrics CEOs Actually Care About" | Save this post | Infographic |
| Wed 5 | 🎯 Promotional | "Case Study: How [Client] Increased ROAS by 3x" | Book a call | Before/After |
| Fri 7 | 💬 Engagement | "Poll: What's your biggest marketing challenge in 2025?" | Vote & comment | Poll graphic |

### Week 2 (Feb 10-14)

| Day | Type | Topic | CTA | Visual |
|-----|------|-------|-----|--------|
| Mon 10 | 📚 Educational | "Google Ads vs Facebook Ads: Which is Right for You?" | Read more (blog) | Comparison chart |
| Wed 12 | 💬 Engagement | "❤️ Valentine's Special: Show some love to your marketing team!" | Tag your team | Hearts theme |
| Fri 14 | 🎯 Promotional | "Free Marketing Audit - Limited Spots for Q1" | Apply now | Urgency banner |

### Week 3 (Feb 17-21)

| Day | Type | Topic | CTA | Visual |
|-----|------|-------|-----|--------|
| Mon 17 | 📚 Educational | "AI in Marketing: 3 Tools We Use Daily" | Try these tools | Tool screenshots |
| Wed 19 | 🎯 Promotional | "Client Spotlight: [Success Story]" | Learn how | Video testimonial |
| Fri 21 | 💬 Engagement | "What's one marketing hack that changed your game?" | Share below | Question graphic |

### Week 4 (Feb 24-28)

| Day | Type | Topic | CTA | Visual |
|-----|------|-------|-----|--------|
| Mon 24 | 📚 Educational | "Landing Page Optimization: Quick Wins" | Get checklist | Checklist preview |
| Wed 26 | 💬 Engagement | "Hot take: [Controversial marketing opinion]. Agree?" | Debate below | Quote card |
| Fri 28 | 🎯 Promotional | "March is Coming - Book Your Q2 Strategy Session" | Book now | Calendar visual |

---

## Content Mix Summary
- 📚 Educational: 4 posts (33%)
- 🎯 Promotional: 4 posts (33%)
- 💬 Engagement: 4 posts (33%)

## Hashtags to Use
#DigitalMarketing #MarketingTips #SMEMarketing #GrowthHacking #MangoAds
```

---

## USE CASE 3: COMPETITOR AD ANALYSIS

### Tình huống

Bạn cần phân tích quảng cáo của đối thủ và rút ra insights.

### Cách thực hiện

```
> Tôi có thông tin về ad của đối thủ:

Competitor: ABC Agency
Ad 1:
- Headline: "Grow Your Business 10x in 90 Days"
- Body: "Our proven system has helped 500+ SMEs..."
- CTA: Free Consultation
- Visual: Before/after graph

Ad 2:
- Headline: "Tired of Agencies That Don't Deliver?"
- Body: "We guarantee results or your money back..."
- CTA: Get Quote
- Visual: Testimonial video thumbnail

Hãy phân tích và đưa ra gợi ý cho quảng cáo của MangoAds.
```

### Kết quả

```markdown
## 🔍 Competitor Ad Analysis: ABC Agency

### Patterns Identified

| Element | Competitor Approach | Our Opportunity |
|---------|---------------------|-----------------|
| **Headlines** | Aggressive claims (10x growth) | Credible, specific results |
| **Social Proof** | Generic "500+ clients" | Named case studies |
| **Risk Reversal** | Money-back guarantee | Free audit, no commitment |
| **Visuals** | Stock-like graphics | Real client results |

### Competitor Strengths
✅ Strong risk reversal (money-back guarantee)
✅ Clear CTA (Free Consultation)
✅ Addresses pain point (agencies that don't deliver)

### Competitor Weaknesses
❌ Claims seem exaggerated (10x in 90 days)
❌ Generic social proof
❌ No specific methodology shown

### 💡 MangoAds Ad Recommendations

#### Ad Concept 1: Specificity Wins
```
Headline: "We Helped [Client] Increase ROAS from 2.1x to 5.3x"
Body: "Not 10x promises. Real results with real clients.
See exactly how we did it in this case study."
CTA: See Case Study
Visual: Actual GA4 screenshot (blurred sensitive data)
```

#### Ad Concept 2: Address the Pain
```
Headline: "Spent $10K on Ads with Nothing to Show?"
Body: "Most agencies hide behind vanity metrics.
We focus on one thing: revenue.
Free audit reveals exactly where your ad spend is leaking."
CTA: Get Free Audit
Visual: Pie chart showing "wasted spend"
```

#### Ad Concept 3: Transparency Play
```
Headline: "Here's Our Actual Client Dashboard"
Body: "No fluff. No 'proprietary systems.'
Just clear reporting and real results."
CTA: Book Strategy Call
Visual: Real (anonymized) dashboard screenshot
```

### A/B Test Priority
1. Test Ad Concept 1 vs 2 (specificity vs pain point)
2. Winner vs Ad Concept 3
3. Iterate based on CTR and CPL
```

---

## USE CASE 4: EMAIL CAMPAIGN OPTIMIZER

### Tình huống

Email campaign có open rate thấp, cần optimize.

### Cách thực hiện

```
> Email campaign hiện tại có open rate 15% (target 25%).
Subject line đang dùng: "MangoAds Newsletter - January Edition"

Đề xuất 5 subject lines tốt hơn và phân tích tại sao.
```

### Kết quả

```markdown
## 📧 Email Subject Line Optimization

### Current Performance
- Subject: "MangoAds Newsletter - January Edition"
- Open Rate: 15%
- Issue: Generic, no value proposition, no urgency

### Why It's Underperforming
❌ "Newsletter" = inbox skip word
❌ No benefit stated
❌ No personalization
❌ No curiosity gap

### 💡 Optimized Subject Lines

| # | Subject Line | Psychology | Expected OR |
|---|--------------|------------|-------------|
| 1 | "Your competitors are doing this (are you?)" | FOMO, curiosity | 28-32% |
| 2 | "[Name], your Q1 marketing checklist" | Personalization, utility | 25-28% |
| 3 | "We analyzed 100 campaigns. Here's what works." | Data, credibility | 24-27% |
| 4 | "Quick question about your marketing budget" | Conversational, low commitment | 26-30% |
| 5 | "The $10K mistake most marketers make" | Loss aversion, curiosity | 27-31% |

### A/B Test Plan

**Test 1:** Personalization vs FOMO
- A: "[Name], your Q1 marketing checklist"
- B: "Your competitors are doing this (are you?)"
- Sample: 20% of list each
- Winner: Send to remaining 60%

**Test 2:** Winner vs Data-driven
- Test the winning subject against #3

### Additional Tips
✅ Send time: Tuesday/Thursday 10am
✅ Preview text: Extend the curiosity
✅ From name: "An from MangoAds" (personal > brand)
```

---

## USE CASE 5: SOCIAL MEDIA POST GENERATOR

### Tình huống

Cần viết posts nhanh cho announcement mới.

### Cách thực hiện

```
> Viết 3 versions của social media post cho:
- Announcement: MangoAds vừa đạt Google Premier Partner
- Platforms: LinkedIn, Facebook, Instagram
- Tone: Professional nhưng excited
```

### Kết quả

```markdown
## 🎉 Social Media Posts: Google Premier Partner Announcement

### LinkedIn Version (Professional)
```
Excited to share that MangoAds has achieved Google Premier Partner status! 🏆

This places us in the top 3% of Google Partners worldwide.

What this means for our clients:
✅ Direct access to Google product betas
✅ Dedicated Google support team
✅ Advanced training and certifications
✅ Exclusive industry insights

Thank you to our amazing team and clients who made this possible.

Here's to helping even more businesses grow through data-driven marketing in 2025! 📈

#GooglePartner #DigitalMarketing #MangoAds #AgencyLife
```

### Facebook Version (Community-focused)
```
WE DID IT! 🎉

MangoAds is now officially a Google Premier Partner - that's the top 3% of partners globally!

What does this mean for YOU?
👉 Better ad performance through early access to new features
👉 Faster support when you need it
👉 Cutting-edge strategies backed by Google

Huge thanks to everyone who's been part of this journey. Our clients, our team, our community - this is YOUR win too! ❤️

Drop a 🙌 if you're excited for what's coming next!

#GooglePremierPartner #GrowthPartners #MarketingAgency
```

### Instagram Version (Visual-first)
```
TOP 3% IN THE WORLD 🌍🏆

Swipe to see what Google Premier Partner means for your business →

[Slide 1: Big announcement graphic]
[Slide 2: What is Premier Partner?]
[Slide 3: Benefits for clients - bullet points]
[Slide 4: Thank you to team/clients]
[Slide 5: CTA - Work with us]

Caption:
We're OFFICIALLY Google Premier Partners!

This means bigger results, better support, and exclusive access for our clients.

2025 is going to be BIG. Ready to grow with us? Link in bio 🚀

#GooglePremierPartner #DigitalMarketing #MarketingAgency #Growth #MangoAds
```
```

---

## USE CASE 6: SEO CONTENT AUDIT

### Tình huống

Kiểm tra nhanh SEO của một blog post.

### Cách thực hiện

```
> Audit SEO cho blog post này:

Title: Digital Marketing Tips
URL: /blog/digital-marketing-tips
H1: Marketing Tips for Your Business
Word count: 450 words
Images: 2 (no alt text)
Internal links: 0
External links: 1
Meta description: (không có)
```

### Kết quả

```markdown
## 🔍 SEO Audit: Digital Marketing Tips

### Overall Score: 35/100 ⚠️ Needs Work

### Detailed Analysis

| Factor | Status | Issue | Fix |
|--------|--------|-------|-----|
| Title Tag | 🔴 | Too generic | Add keyword + benefit |
| Meta Description | 🔴 | Missing | Add 150-160 char description |
| H1 | 🟡 | Doesn't match title | Align with target keyword |
| Word Count | 🔴 | Too short (450) | Expand to 1,500+ |
| Images | 🟡 | No alt text | Add descriptive alt text |
| Internal Links | 🔴 | None | Add 3-5 relevant links |
| External Links | 🟢 | Has 1 | Add 1-2 more authoritative sources |
| URL | 🟡 | Okay but generic | Consider more specific URL |

### 💡 Quick Fixes (Priority Order)

**1. Add Meta Description**
```
"Discover 10 proven digital marketing tips to grow your business in 2025. From SEO to social media, learn actionable strategies that drive real results."
```

**2. Optimize Title Tag**
```
Before: "Digital Marketing Tips"
After: "10 Digital Marketing Tips That Actually Work in 2025 | MangoAds"
```

**3. Fix H1**
```
Before: "Marketing Tips for Your Business"
After: "10 Digital Marketing Tips to Grow Your Business in 2025"
```

**4. Add Alt Text to Images**
```
Image 1: "Digital marketing strategy infographic showing ROI metrics"
Image 2: "Marketing funnel diagram with conversion rates"
```

**5. Add Internal Links**
- Link to: /services/digital-marketing
- Link to: /blog/seo-guide
- Link to: /case-studies

### Content Expansion Suggestions
- Add real examples and case studies
- Include statistics and data
- Add FAQ section
- Create step-by-step actionable tips
- Target word count: 1,500-2,000 words
```

---

## HANDS-ON LAB: COMPLETE CAMPAIGN WORKFLOW

### Thời lượng: 45 phút

### Scenario

Bạn được giao launch campaign mới:
- **Client:** TechStartup ABC
- **Objective:** Lead Generation
- **Budget:** $5,000/tháng
- **Channels:** Facebook + Google
- **Target:** Tech founders tại Vietnam

### Task 1: Tạo CLAUDE.md (10 phút)

```
> /init

Sau đó customize với thông tin campaign:
- Client name
- Budget
- KPIs
- UTM convention
- Target audience
```

**Checklist:**
- [ ] Client info đầy đủ
- [ ] Budget và timeline rõ ràng
- [ ] KPIs được define
- [ ] UTM convention được set

### Task 2: Tạo UTM Links (10 phút)

Tạo UTM links cho:
1. Facebook Feed Ad
2. Facebook Story Ad
3. Google Search Ad
4. Google Display Ad

```
> Tạo 4 UTM links cho campaign TechStartup ABC với các ad placements trên
```

**Checklist:**
- [ ] 4 links được tạo
- [ ] Format đúng chuẩn
- [ ] Có thể phân biệt placement trong GA4

### Task 3: Phân tích Metrics giả định (10 phút)

Sau 1 tuần, data như sau:
```
Facebook Feed: $1,500 spend, 12,000 clicks, 180 leads
Facebook Story: $500 spend, 8,000 clicks, 40 leads
Google Search: $2,000 spend, 1,500 clicks, 200 leads
Google Display: $1,000 spend, 25,000 clicks, 80 leads
```

```
> Phân tích performance và recommend budget allocation cho tuần 2
```

**Checklist:**
- [ ] Tính được CPL cho mỗi channel
- [ ] Identify best performer
- [ ] Có recommendation cụ thể

### Task 4: Tạo Weekly Report (15 phút)

```
> /weekly-report

[Input data từ Task 3]
```

**Checklist:**
- [ ] Report có executive summary
- [ ] Có metrics table
- [ ] Có recommendations
- [ ] Có action items

---

## CHECKLIST HOÀN THÀNH PART 3

### Use Cases đã thực hành
- [ ] Campaign Performance Analysis
- [ ] Content Calendar Generator
- [ ] Competitor Ad Analysis
- [ ] Email Campaign Optimizer
- [ ] Social Media Post Generator
- [ ] SEO Content Audit

### Lab hoàn thành
- [ ] Tạo CLAUDE.md cho campaign
- [ ] Tạo 4 UTM links
- [ ] Phân tích metrics và recommend
- [ ] Tạo weekly report

---

## TÓM TẮT PART 3

### Đã học:
- [x] 6 Use Cases thực tế cho Marketing
- [x] Cách phân tích campaign performance
- [x] Content calendar workflow
- [x] Competitor analysis framework
- [x] Email và social media optimization

### Part 4 sẽ học:
- [ ] Templates chuẩn MangoAds
- [ ] Best practices
- [ ] Cheatsheet 1 trang
- [ ] Governance và security cơ bản

---

**Tiếp theo:** [Part 4: Templates & Cheatsheet](./part-04-templates-cheatsheet.md)
