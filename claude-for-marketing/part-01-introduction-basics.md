# CLAUDE CODE CHO MARKETING - MANGOADS
## Part 1: Giới Thiệu & Khái Niệm Cơ Bản

**Phiên bản:** 1.0
**Đối tượng:** Team Marketing MangoAds (Content, Performance, SEO, Social Media)
**Thời lượng đọc:** 30-45 phút

---

## MỤC LỤC

1. [Claude Code là gì và tại sao Marketing cần biết?](#1-claude-code-là-gì-và-tại-sao-marketing-cần-biết)
2. [Cài đặt và bắt đầu sử dụng](#2-cài-đặt-và-bắt-đầu-sử-dụng)
3. [CLAUDE.md - Bộ nhớ cho dự án của bạn](#3-claudemd---bộ-nhớ-cho-dự-án-của-bạn)
4. [Các khái niệm cơ bản cần biết](#4-các-khái-niệm-cơ-bản-cần-biết)
5. [Checklist hoàn thành Part 1](#5-checklist-hoàn-thành-part-1)

---

## 1. CLAUDE CODE LÀ GÌ VÀ TẠI SAO MARKETING CẦN BIẾT?

### 1.1 Claude Code trong 1 phút

**Claude Code** là công cụ AI của Anthropic giúp bạn:
- 🚀 Tự động hóa các tác vụ lặp đi lặp lại
- 📊 Sinh báo cáo nhanh chóng
- 🔗 Tạo UTM links theo chuẩn
- ✍️ Hỗ trợ viết content brief
- 📈 Phân tích data campaign

### 1.2 Tại sao Marketing nên dùng?

| Vấn đề hiện tại | Claude Code giải quyết |
|-----------------|------------------------|
| Làm weekly report mất 3-4 tiếng | Tự động sinh report trong 5 phút |
| UTM links không nhất quán | Skill tự động validate và sinh UTM chuẩn |
| Content brief thiếu structure | Template brief có sẵn, chỉ cần điền thông tin |
| Quên check tracking pixels | Nhắc nhở và kiểm tra tự động |

### 1.3 Bạn sẽ làm được gì sau training này?

✅ Tạo được **Weekly Report** tự động
✅ Sinh **UTM links** chuẩn MangoAds
✅ Viết **Content Brief** có structure
✅ Thiết lập **workflow** cho campaign
✅ Sử dụng **slash commands** để tiết kiệm thời gian

---

## 2. CÀI ĐẶT VÀ BẮT ĐẦU SỬ DỤNG

### 2.1 Cách mở Claude Code

**Bước 1:** Mở Terminal (Mac) hoặc Command Prompt (Windows)

**Bước 2:** Gõ lệnh:
```bash
claude
```

**Bước 3:** Bạn sẽ thấy giao diện Claude Code:
```
╭─────────────────────────────────────────╮
│  Welcome to Claude Code                 │
│  Type your message or /help             │
╰─────────────────────────────────────────╯
>
```

### 2.2 Các lệnh cơ bản cần nhớ

| Lệnh | Chức năng | Khi nào dùng |
|------|-----------|--------------|
| `/help` | Xem hướng dẫn | Khi không biết làm gì |
| `/init` | Tạo file CLAUDE.md | Khi bắt đầu dự án mới |
| `/clear` | Xóa màn hình | Khi muốn bắt đầu lại |
| `Ctrl+C` | Dừng Claude | Khi Claude chạy quá lâu |

### 2.3 Cách giao tiếp với Claude

**Viết rõ ràng, cụ thể:**

```
❌ XẤU:
"Làm report"

✅ TỐT:
"Tạo weekly report cho campaign Q1 Brand Awareness,
bao gồm: impressions, clicks, CTR, conversions,
so sánh với tuần trước"
```

**Cung cấp context:**

```
❌ XẤU:
"Tạo UTM link"

✅ TỐT:
"Tạo UTM link cho landing page mangoads.com/services
- Source: facebook
- Medium: paid_social
- Campaign: Q1 Lead Generation
- Content: header-cta"
```

---

## 3. CLAUDE.MD - BỘ NHỚ CHO DỰ ÁN CỦA BẠN

### 3.1 CLAUDE.md là gì?

**CLAUDE.md** là file đặc biệt mà Claude tự động đọc mỗi khi bạn làm việc. Nó giống như:

> 📝 **Tưởng tượng:** Bạn có một thực tập sinh mới. Mỗi sáng bạn phải nhắc lại: "Campaign này target audience là..., budget là..., KPIs là...".
>
> Với CLAUDE.md, bạn chỉ cần viết một lần, Claude sẽ nhớ mãi!

### 3.2 Tạo CLAUDE.md cho campaign

**Bước 1:** Trong Claude Code, gõ:
```
/init
```

**Bước 2:** Claude sẽ tạo file CLAUDE.md. Bạn có thể chỉnh sửa:

```markdown
# Campaign: Q1 2025 Brand Awareness

## Thông tin cơ bản
- Client: [Tên client]
- Thời gian: 01/01/2025 - 31/03/2025
- Budget: $10,000/tháng
- Objective: Brand Awareness

## Target Audience
- Độ tuổi: 25-45
- Location: Việt Nam (HCM, Hà Nội)
- Interests: Digital marketing, Business

## Channels
- Google Ads (Display)
- Facebook/Instagram
- LinkedIn

## KPIs
- Impressions: 500,000/tháng
- Reach: 200,000/tháng
- CTR: > 0.5%

## UTM Convention
- Source: google, facebook, linkedin
- Medium: display, social, paid_social
- Campaign: 2025-q1-brand-awareness

## Tracking
- GA4 Property: G-XXXXXXXXXX
- Facebook Pixel: XXXXXXXXXX

## Team
- Account Manager: @am-name
- Designer: @designer-name
```

### 3.3 Lợi ích của CLAUDE.md

| Không có CLAUDE.md | Có CLAUDE.md |
|-------------------|--------------|
| Mỗi lần phải nhắc lại campaign info | Claude tự hiểu context |
| UTM không nhất quán | UTM theo đúng convention |
| Quên KPIs khi làm report | Luôn check đúng KPIs |
| Mất thời gian explain | Tiết kiệm thời gian |

---

## 4. CÁC KHÁI NIỆM CƠ BẢN CẦN BIẾT

### 4.1 Bảng tổng hợp (Chỉ những gì Marketing cần)

| Khái niệm | Giải thích đơn giản | Ví dụ Marketing |
|-----------|---------------------|-----------------|
| **CLAUDE.md** | "Bộ nhớ" của dự án | Campaign info, KPIs, UTM convention |
| **Skill** | Khả năng đặc biệt Claude có thể dùng | Skill sinh UTM, tính metrics |
| **Command** | Shortcut bạn gõ để Claude làm việc | `/weekly-report`, `/utm` |
| **Subagent** | AI chuyên gia cho task cụ thể | Report Generator, Content Writer |

### 4.2 Skill - Khả năng Claude tự động sử dụng

**Skill là gì?**

Skill giống như "kỹ năng" bạn dạy cho Claude. Một khi đã có skill, Claude sẽ tự động dùng khi cần.

**Ví dụ:**

```
Bạn: "Tạo UTM link cho facebook campaign Q1"

Claude (đã có UTM skill):
"Tôi sẽ dùng UTM Generator skill để tạo link..."

→ Tự động validate source, medium
→ Tự động format campaign name đúng chuẩn
→ Output: https://mangoads.com?utm_source=facebook&utm_medium=paid_social&utm_campaign=2025-q1-...
```

### 4.3 Command - Shortcut cho công việc thường xuyên

**Command là gì?**

Command là lệnh tắt bạn gõ để Claude thực hiện một workflow cụ thể.

**Ví dụ:**

| Command | Chức năng |
|---------|-----------|
| `/weekly-report` | Tạo báo cáo tuần |
| `/utm` | Sinh UTM link |
| `/content-brief` | Tạo content brief template |
| `/campaign-summary` | Tóm tắt campaign performance |

**Cách dùng:**
```
> /weekly-report q1-brand-awareness

Claude: "Đang tạo weekly report cho Q1 Brand Awareness..."
```

### 4.4 Subagent - AI chuyên gia

**Subagent là gì?**

Subagent là "AI chuyên gia" cho một lĩnh vực cụ thể. Thay vì Claude làm tất cả, Claude gọi chuyên gia phù hợp.

**Ví dụ:**

```
┌─────────────────────────────────────────────────────┐
│  Bạn: "Phân tích campaign và viết weekly report"   │
│                                                     │
│  Claude:                                            │
│  "Tôi sẽ gọi 2 chuyên gia..."                      │
│                                                     │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ Performance     │  │ Report          │          │
│  │ Analyst         │  │ Writer          │          │
│  │                 │  │                 │          │
│  │ Phân tích số    │  │ Viết báo cáo    │          │
│  │ liệu campaign   │  │ từ data         │          │
│  └─────────────────┘  └─────────────────┘          │
│                                                     │
│  → Kết quả: Weekly report hoàn chỉnh               │
└─────────────────────────────────────────────────────┘
```

### 4.5 Tóm tắt: Khi nào dùng gì?

```
┌─────────────────────────────────────────────────────┐
│  DECISION GUIDE CHO MARKETING                       │
│                                                     │
│  Cần Claude nhớ thông tin campaign?                │
│  → Dùng CLAUDE.md                                  │
│                                                     │
│  Cần Claude tự động làm gì đó (như sinh UTM)?      │
│  → Đã có Skill                                     │
│                                                     │
│  Muốn shortcut cho việc thường làm?                │
│  → Dùng /command                                   │
│                                                     │
│  Cần phân tích phức tạp hoặc báo cáo chi tiết?    │
│  → Claude sẽ gọi Subagent                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 5. CHECKLIST HOÀN THÀNH PART 1

### Kiến thức

- [ ] Hiểu Claude Code là gì và lợi ích cho Marketing
- [ ] Biết cách mở Claude Code
- [ ] Hiểu CLAUDE.md là gì
- [ ] Phân biệt được Skill, Command, Subagent

### Thực hành

- [ ] Mở Claude Code thành công
- [ ] Chạy lệnh `/help`
- [ ] Tạo CLAUDE.md cho một campaign bất kỳ

### Câu hỏi tự kiểm tra

1. **CLAUDE.md dùng để làm gì?**
   - A) Viết code
   - B) Lưu thông tin campaign để Claude nhớ ✅
   - C) Gửi email

2. **Skill khác Command như thế nào?**
   - Skill: Claude tự động dùng khi cần
   - Command: Bạn phải gõ /command để trigger

3. **Khi nào nên dùng Subagent?**
   - Khi task phức tạp, cần chuyên gia AI riêng

---

## TÓM TẮT PART 1

### Đã học:
- [x] Claude Code là gì và lợi ích cho Marketing
- [x] Cách cài đặt và sử dụng cơ bản
- [x] CLAUDE.md - bộ nhớ cho campaign
- [x] Skill, Command, Subagent là gì

### Part 2 sẽ học:
- [ ] Tạo UTM Generator Skill
- [ ] Tạo Weekly Report Command
- [ ] Metric Calculator Skill
- [ ] Thực hành với campaign thật

---

**Tiếp theo:** [Part 2: Skills & Commands cho Marketing](./part-02-skills-commands.md)
