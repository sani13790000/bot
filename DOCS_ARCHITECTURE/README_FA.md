# 🚀 Galaxy Vast Trading Bot™
## پلتفرم تجارتی نهادی برای MetaTrader 5

---

## 📌 خلاصهٔ فوری

یک **ربات تجاری خودکار** برای معاملهٔ ارزهای خارجی (Forex) که:

- ✅ **۲۴/۷** بازارها را نظارت می‌کند
- ✅ **خودکار** معاملات را اجرا می‌کند  
- ✅ **هوشمند** از تاریخ می‌آموزد
- ✅ **محفوظ** ریسک‌ها را کنترل می‌کند
- ✅ **شفاف** گزارش‌ها تهیه می‌کند

---

## 📚 مستندات

```
📂 Documentation Files:
│
├── 📄 INDEX_FA.md                   ← شروع اینجا!
│   └─ نقشهٔ راه و راهنمایی خواندن
│
├── 📄 SYSTEM_OVERVIEW_FA.md         ← برای مبتدیان
│   └─ نمای کلی ۱۰ جزء
│
├── 📄 FOLDER_STRUCTURE_FA.md        ← برای توسعه‌دهندگان
│   └─ ساختار دایرکتوری و فایل‌ها
│
├── 📄 DEPENDENCY_MAP_FA.md          ← برای معماران
│   └─ نقشهٔ وابستگی‌ها و رویدادها
│
└── 📄 ARCHITECTURE_FA.md            ← برای متخصصین
    └─ معماری فنی و الگوها
```

---

## 🎯 کجا شروع کنم؟

### برای مبتدیان:
```
۱. INDEX_FA.md را بخوانید (۳۰ دقیقه)
۲. SYSTEM_OVERVIEW_FA.md را بخوانید (۱.۵ ساعت)
۳. FOLDER_STRUCTURE_FA.md را بخوانید (۱ ساعت)
⏱️ کل: ۳ ساعت
```

### برای توسعه‌دهندگان:
```
۱. SYSTEM_OVERVIEW_FA.md (تمام)
۲. FOLDER_STRUCTURE_FA.md (تمام)  
۳. DEPENDENCY_MAP_FA.md (تمام)
۴. ARCHITECTURE_FA.md (برحسب نیاز)
⏱️ کل: ۸ ساعت
```

### برای معماران:
```
۱. ARCHITECTURE_FA.md (تمام)
۲. DEPENDENCY_MAP_FA.md (تمام)
۳. SYSTEM_OVERVIEW_FA.md (خلاصه)
⏱️ کل: ۶ ساعت
```

---

## 🏗️ ساختار پروژه

```
galaxy-vast-trading-bot/
│
├── core/                    # هسته سیستم
├── agents/                  # عاملین هوش مصنوعی
├── strategies/              # استراتژی‌های تجاری
├── analysis/                # موتورهای تحلیل
├── infrastructure/          # زیرساخت (DB, Cache, MT5)
├── services/                # خدمات (Licensing, Billing)
├── integrations/            # یکپارچه‌سازی‌ها (News, APIs)
├── dashboard/               # رابط کاربری (React/Vue)
├── tests/                   # آزمون‌ها
├── config/                  # تنظیمات
├── docs/                    # مستندات
├── scripts/                 # اسکریپت‌های خودکار
├── deploy/                  # فایل‌های استقرار (Docker, K8s)
└── logs/                    # گزارش‌ها
```

**برای توضیح هر فولدر:** به FOLDER_STRUCTURE_FA.md مراجعه کنید.

---

## 🔟 اجزای اصلی

۱. **هسته MetaTrader** - اتصال و اجرای معاملات
۲. **Price Action** - تحلیل الگوهای قیمتی
۳. **Smart Money Concepts** - فهم رفتار بزرگ‌تاجران
۴. **ICT Concepts** - استفاده از ریتم‌های بازار
۵. **AI Multi-Agent** - تصمیم‌گیری هوشمند
۶. **News Intelligence** - تحلیل اخبار اقتصادی
۷. **Market Scanner** - جستجوی فرصت‌ها
۸. **Currency Strength** - تحلیل قوت ارزها
۹. **Risk Management** - کنترل ریسک
۱۰. **Backtesting** - آزمایش استراتژی‌ها

---

## 🛠️ تکنولوژی‌ها

| بخش | تکنولوژی |
|-----|----------|
| Backend | Python 3.11+ |
| API | FastAPI |
| Database | PostgreSQL |
| Cache | Redis |
| Queue | RabbitMQ |
| ML/AI | TensorFlow / PyTorch |
| Frontend | React / Vue |
| Deployment | Docker + Kubernetes |

---

## ⚠️ تذکرات مهم

### 🔴 خطرناک!
- این ربات **واقعی پول** درگیر می‌کند
- آزمایش کامل کنید **قبل از Live**
- ریسک **۲-۳% هر معامله** تنظیم کنید
- **۵% روزانه** حد زیان نهایی است
- **ریسک پذیری شما** است، نه ما

### ✅ راهنمایی
- ۱. توسعه کامل کنید
- ۲. بک‌تست انجام دهید
- ۳. Paper trading شروع کنید
- ۴. **سپس** small live account
- ۵. آهسته scale بالا ببرید

---

## 📊 مثال ساده

### یک معامله چگونه اجرا می‌شود؟

```
۱. Market Data (قیمت لحظه‌ای دریافت)
   ↓
۲. Analysis (تحلیل Price Action + SMC + ICT)
   ↓
۳. Signal Generation (سیگنال تولید)
   ↓
۴. Decision Making (تصمیم‌گیر می‌تصمیم می‌گیرد)
   ↓
۵. Risk Check (بررسی ایمنی ریسک)
   ↓
۶. Order Execution (اجرای معامله)
   ↓
۷. Monitoring (نظارت دایمی)
   ↓
۸. Closing (بسته‌شدن معامله)
   ↓
۹. Learning (یادگیری از نتیجه)
```

---

## 🚀 شروع کار

### مرحلهٔ ۱: راه‌اندازی
```bash
git clone https://github.com/sani13790000/bot.git
cd bot
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

### مرحلهٔ ۲: تنظیم
```bash
# فایل .env را ویرایش کنید
# MT5 کلیدها را اضافه کنید
# Database اطلاعات را تنظیم کنید
```

### مرحلهٔ ۳: اجرا
```bash
python main.py --mode backtesting
python main.py --mode paper_trading
python main.py --mode live  # دقیق باشید!
```

---

## 💡 نکات مهم

### این معماری:
✅ **آموزشی** - برای یادگیری معماری  
✅ **قابل‌توسعه** - برای تجاری واقعی  
✅ **نهادی** - برای شرکت‌ها  
❌ **نه تجاری** - خودتان باید استراتژی بنویسید  
❌ **نه کد نهایی** - باید سفارشی کنید  

### مسئولیت:
- ⚠️ **تمام** خسارات مالی = بر عهدهٔ شما
- ⚠️ قوانین محلی = شما رعایت کنید
- ⚠️ آزمایش = شما انجام دهید

---

## 📞 پرسش‌های متداول

**Q: کتاب انگریزی هست؟**  
A: فقط فارسی (Persian). اسناد انگریزی بعدا.

**Q: چقدر زمان توسعه گیر می‌آید؟**  
A: ۲-۳ ماه برای اولین نسخهٔ زنده.

**Q: چقدر حذر باید کنم؟**  
A: خیلی! اگر مطمئن نیستید، نکنید.

**Q: API چی؟**  
A: REST + WebSocket برای داشبورد.

**Q: license چی هست؟**  
A: Proprietary. استفادهٔ تجاری = مجوز.

---

## 📈 نمودار راه توسعه

```
Month 1-2: Core Development
├─ Core Engine
├─ Infrastructure
└─ Basic Analysis

Month 2-3: Features
├─ All Analysis Engines
├─ Agents System
└─ Risk Management

Month 3-4: Advanced
├─ AI/ML Integration
├─ Backtesting System
└─ Dashboard

Month 4+: Production
├─ Testing & Optimization
├─ Deployment
└─ Monitoring
```

---

## 🎓 منابع بیرونی

### برای تجاری:
- ICT Concepts یادگیری
- SMC و Order Blocks
- Price Action Trading

### برای کد:
- Python Advanced
- Design Patterns
- Microservices

### برای ML:
- TensorFlow Tutorial
- Time Series Analysis

---

## 📝 نسخهٔ تاریخ

**نسخهٔ فعلی:** 1.0  
**تاریخ:** 1403/04/27 (2024-07-17)  
**وضعیت:** ✅ معماری نهایی

---

## 👨‍💼 درباره

**Galaxy Vast Trading Bot™**  
Institutional-Grade Trading Platform  

طراحی برای:
- 🎓 دانشجویان (یادگیری)
- 👨‍💻 توسعه‌دهندگان (اجرا)
- 🏢 شرکت‌ها (نهادی)

---

## 🤝 کمک و مشارکت

```
۱. Fork کنید
۲. شاخه بسازید
۳. تغییرات انجام دهید
۴. Commit کنید
۵. Pull Request بسازید
```

---

## ⚖️ مجوز

Galaxy Vast Trading Bot™ - تحت مجوز proprietary

```
⚠️ تنبیه اهم:
- استفادهٔ تجاری = نیاز به مجوز
- خسارات = مسئولیت شما
- قوانین = رعایت شما
- تست = انجام شما
```

---

## 📞 تماس

برای سوالات:
- 📖 ابتدا اسناد را بخوانید
- 💬 سپس GitHub Issues باز کنید
- 📧 یا ایمیل بفرستید

---

## 🎯 خلاصه

```
✨ یک معماری کامل برای ربات تجاری
📚 مستندات جامع فارسی
🏗️ ساختار تمیز و سازمان‌یافته
🔐 امنیت و قابلیت‌اطمینان
📈 قابلیت مقیاس‌پذیری

⏱️ شروع کنید با INDEX_FA.md
🚀 موفق باشید!
```

---

**آخرین بروزرسانی:** 1403/04/27
