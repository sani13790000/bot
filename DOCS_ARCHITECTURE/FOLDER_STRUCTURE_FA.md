# 📁 ساختار دایرکتوری Galaxy Vast Trading Bot™

**نسخه:** 1.0  
**تاریخ:** 1403/04/27  
**وضعیت:** ✅ نهایی

---

## 🗂️ نمای کلی ساختار

```
galaxy-vast-trading-bot/
│
├── 📂 core/                          # هسته سیستم
├── 📂 agents/                        # عاملین هوش مصنوعی
├── 📂 strategies/                    # استراتژی‌های تجاری
├── 📂 analysis/                      # موتورهای تحلیل
├── 📂 infrastructure/                # زیرساخت
├── 📂 services/                      # خدمات
├── 📂 integrations/                  # یکپارچه‌سازی‌ها
├── 📂 dashboard/                     # داشبورد
├── 📂 tests/                         # آزمون‌ها
├── 📂 config/                        # تنظیمات
├── 📂 docs/                          # مستندات
├── 📂 scripts/                       # اسکریپت‌ها
├── 📂 deploy/                        # استقرار
└── 📂 logs/                          # گزارش‌ها
```

---

## 📂 توضیح هر پوشه (تفصیلی)

### 1. 📂 `core/` - هسته سیستم

**چرا وجود دارد:**
- اجرای تمام عملیات اساسی
- مدیریت جریان داده‌ها
- هماهنگی‌سازی بین ماژول‌ها

**چه می‌کند:**
```
core/
├── engine.py              # موتور اصلی
├── config_manager.py      # مدیر تنظیمات
├── event_dispatcher.py    # توزیع‌کننده رویدادها
├── logger.py              # ثبت گزارش‌ها
├── constants.py           # ثابت‌های سیستم
└── exceptions.py          # تعریف خطاها
```

**فایل‌های اصلی:**

| فایل | مسئولیت | مثال |
|------|---------|------|
| `engine.py` | راه‌اندازی و اجرای سیستم | شروع ربات، متوقف کردن ربات |
| `config_manager.py` | بارگذاری و مدیریت تنظیمات | محدودیت‌های ریسک، کلیدهای API |
| `event_dispatcher.py` | اطلاع‌رسانی ماژول‌ها از رویدادها | "قیمت تغیر کرد"، "معامله بسته شد" |
| `logger.py` | ثبت تمام رویدادها | هر معاملات، هر خطا |
| `constants.py` | تمام مقادیر ثابت | "حداکثر ریسک = 2%"، "حداکثر معاملات = 5" |
| `exceptions.py` | تعریف خطاهای سفارشی | "NoValidSignal"، "InsufficientBalance" |

**مثال کد:**
```python
# engine.py - هسته اصلی
class TradingEngine:
    def __init__(self, config):
        self.config = config
        self.mt5_connection = MT5Connection()
        self.agents = []
        self.is_running = False
    
    def start(self):
        # شروع تمام ماژول‌ها
        self.is_running = True
        self.logger.info("Trading engine started")
    
    def process_market_data(self, tick):
        # پردازش هر داده‌ی بازاری
        pass
```

---

### 2. 📂 `agents/` - عاملین هوش مصنوعی

**چرا وجود دارد:**
- تقسیم‌کار بین عاملین
- تصمیم‌گیری مستقل
- یادگیری مستمر

**چه می‌کند:**
```
agents/
├── decision_agent.py      # تصمیم‌گیر
├── risk_agent.py          # کنترل‌کننده ریسک
├── opportunity_agent.py   # شناسایی‌کننده فرصت
├── learning_agent.py      # یادگیری‌کننده
└── monitor_agent.py       # نظارت‌کننده
```

**هر عامل چه می‌کند:**

#### 2.1 `decision_agent.py` - تصمیم‌گیر
```
مسئولیت: تصمیم نهایی برای معامله

ورودی:
  - سیگنال‌های تحلیل
  - وضعیت بازار
  - تاریخ معاملات

خروجی:
  - تصمیم: BUY، SELL، یا HOLD
  - اطمینان تصمیم: 0-100%

منطق:
  IF تمام سیگنال‌ها تأیید شوند
     AND اطمینان > 70%
  THEN تصمیم "BUY" را بگیر
```

#### 2.2 `risk_agent.py` - کنترل‌کننده ریسک
```
مسئولیت: اطمینان از ایمنی معامله

بررسی‌های امنیتی:
  1. آیا حساب تعادل کافی است؟
  2. آیا ریسک هر معامله > 3% است؟ (بدی است)
  3. آیا تعداد معاملات باز > حد مجاز؟
  4. آیا زیان روزانه به حد نهایی رسید؟

مثال:
  حساب: 10,000 دلار
  معامله جدید می‌خواهد: 500 دلار ریسک
  بررسی: 500/10000 = 5% > 3%
  نتیجه: REJECT (رد کردن)
```

#### 2.3 `opportunity_agent.py` - شناسایی‌کننده فرصت
```
مسئولیت: پیدا کردن فرصت‌های تجاری

جستجو برای:
  1. Order Blocks (نقاط خریدی بزرگ‌تاجران)
  2. Fair Value Gaps (شکاف‌های قیمتی)
  3. Break of Structure (شکست سازمان)
  4. Price Action Patterns (الگوهای قیمتی)

خروجی:
  - فهرستی از فرصت‌ها
  - میزان قوت هر فرصت
  - جایگاه ورود و خروج پیشنهادی
```

#### 2.4 `learning_agent.py` - یادگیری‌کننده
```
مسئولیت: یادگیری از معاملات

پس از هر معامله:
  1. نتیجه را ثبت کنید (برد/باخت)
  2. تحلیل کنید چرا برد یا باخت؟
  3. الگوهای موفق را یادبگیرید
  4. پارامترها را ببهتر کنید

مثال:
  معامله 1: BUY → سود 50 دلار
    ✓ استفاده کردیم Order Block
    ✓ قوت ارز مثبت بود
    ✓ نوسان‌پذیری مناسب بود
  → این شرایط را بیشتر جستجو کنید
```

#### 2.5 `monitor_agent.py` - نظارت‌کننده
```
مسئولیت: نظارت بر معاملات باز

وظایف:
  1. بررسی Stop Loss و Take Profit
  2. اطلاع‌رسانی تغییرات قیمت
  3. شناسایی خطاها
  4. اطمینان از اتصال MT5

مثال:
  معاملهٔ باز: EUR/USD BUY
  Entry: 1.0850
  Stop Loss: 1.0820 (-30 نقطه)
  Take Profit: 1.0900 (+50 نقطه)
  
  وظیفه: پیگیری دایمی تا بسته‌شدن
```

---

### 3. 📂 `strategies/` - استراتژی‌های تجاری

**چرا وجود دارد:**
- جداسازی استراتژی‌ها از هسته
- آسان‌تر تغییر و بهبود
- تست فرضیات مختلف

**چه می‌کند:**
```
strategies/
├── base_strategy.py       # کلاس پایه‌ای
├── smc_strategy.py        # استراتژی Smart Money
├── price_action_strategy.py # استراتژی Price Action
├── ict_strategy.py        # استراتژی ICT
├── hybrid_strategy.py     # ترکیب استراتژی‌ها
└── __init__.py
```

**هر استراتژی:**
```python
class BaseStrategy:
    def __init__(self):
        self.name = "استراتژی"
        self.parameters = {}
    
    def generate_signal(self, market_data):
        # تحلیل داده‌های بازار
        # بازگردان سیگنال BUY/SELL
        pass
    
    def validate_signal(self, signal):
        # بررسی قوت سیگنال
        # بازگردان میزان اطمینان
        pass
```

---

### 4. 📂 `analysis/` - موتورهای تحلیل

**چرا وجود دارد:**
- تحلیل جنبه‌های مختلف بازار
- محاسبه شاخص‌ها و معیارها
- فراهم‌کردن اطلاعات برای تصمیم‌گیری

**چه می‌کند:**
```
analysis/
├── price_action/
│   ├── pattern_detector.py      # شناسایی الگو
│   ├── support_resistance.py    # نقاط حساس
│   └── trend_analyzer.py        # تحلیل روند
│
├── smc/
│   ├── order_block_finder.py    # پیدا کردن Order Blocks
│   ├── liquidity_analyzer.py    # تحلیل نقدینگی
│   └── structure_analyzer.py    # تحلیل سازمان
│
├── ict/
│   ├── kill_zones.py            # شناسایی ساعات سودآوری
│   ├── fvg_detector.py          # شناسایی شکاف‌های قیمتی
│   └── bos_analyzer.py          # تحلیل شکست سازمان
│
├── technical/
│   ├── indicators.py            # شاخص‌های فنی
│   ├── volatility.py            # تحلیل نوسان‌پذیری
│   └── correlation.py           # تحلیل همبستگی
│
└── sentiment/
    ├── news_sentiment.py        # احساس اخبار
    └── market_sentiment.py      # احساس بازار
```

**مثال هر ماژول:**

#### `pattern_detector.py`
```
وظیفه: شناسایی الگوهای شمعی

الگوهای قابل‌شناسایی:
  1. Bullish Engulfing: نشان‌دهندۀ خرید
  2. Bearish Engulfing: نشان‌دهندۀ فروش
  3. Pin Bar: نشان‌دهندۀ رد شدن
  4. Inside Bar: نشان‌دهندۀ تردید

ورودی: آخرین ۱۰ شمع
خروجی: الگو (اگر پیدا شود) + قوت
```

#### `support_resistance.py`
```
وظیفه: پیدا کردن نقاط حساس

روش شناسایی:
  1. جایی که قیمت چند بار توقف کرد
  2. نقاطی که Round Numbers باشند (1.0800)
  3. نقاطی که بزرگ‌تاجران معامله کردند

خروجی:
  - Support Level: 1.0800
  - Resistance Level: 1.0900
  - Strength: قوت (0-100%)
```

#### `indicators.py`
```
موارد شاخص فنی:
  1. Moving Averages (MA)
  2. RSI (نشان‌دهندۀ سرعت)
  3. MACD (نشان‌دهندۀ رفتار)
  4. Stochastic (نشان‌دهندۀ حداکثر)
  5. Bollinger Bands (نشان‌دهندۀ نوسان)

هر شاخص:
  - نحوۀ محاسبه
  - چه می‌گوید
  - مقادیر بهینه
```

---

### 5. 📂 `infrastructure/` - زیرساخت

**چرا وجود دارد:**
- مدیریت سخت‌افزار و نرم‌افزار
- اتصالات خارجی
- ذخیره‌سازی و بازیابی داده‌ها

**چه می‌کند:**
```
infrastructure/
├── database/
│   ├── models.py              # تعریف جداول
│   ├── migrations/            # تغییرات پایگاه داده
│   └── schemas.py             # ساختار داده‌ها
│
├── cache/
│   ├── redis_cache.py         # کش ریدیس
│   └── memory_cache.py        # کش حافظه
│
├── messaging/
│   ├── rabbitmq_queue.py      # صف RabbitMQ
│   └── event_bus.py           # اتوبوس رویدادها
│
├── mt5/
│   ├── connection.py          # اتصال MT5
│   ├── order_manager.py       # مدیریت سفارشات
│   └── market_data.py         # داده‌های بازار
│
└── storage/
    ├── file_storage.py        # ذخیره فایل
    └── backup.py              # پشتیبان‌گیری
```

**توضیح هر بخش:**

#### `database/models.py`
```
جداول اصلی:

1. Users
   - user_id, email, password, api_key
   
2. Trades
   - trade_id, symbol, entry_price, exit_price
   - entry_time, exit_time, profit_loss
   
3. Accounts
   - account_id, user_id, balance, equity
   - leverage, account_type (Demo/Live/FTMO)
   
4. Signals
   - signal_id, symbol, type (BUY/SELL)
   - strength, timestamp
   
5. Logs
   - log_id, timestamp, level, message
```

#### `mt5/connection.py`
```
وظیفه: اتصال و ارتباط با MetaTrader 5

کارکردها:
  1. احراز هویت
  2. دریافت داده‌های بازار
  3. ارسال سفارشات
  4. بررسی وضعیت حساب
  5. دریافت معاملات باز/بسته

مثال:
  connection = MT5Connection(
      login=123456,
      password="xxxx",
      server="MetaQuotes-Demo"
  )
  connection.send_order(
      symbol="EURUSD",
      volume=0.1,
      type=ORDER_TYPE.BUY,
      price=1.0850
  )
```

#### `messaging/event_bus.py`
```
وظیفه: رویدادها را انتشار دهد

رویدادهای اصلی:
  1. MARKET_DATA_RECEIVED
  2. SIGNAL_GENERATED
  3. TRADE_OPENED
  4. TRADE_CLOSED
  5. ERROR_OCCURRED

مثال:
  event_bus.emit("SIGNAL_GENERATED", {
      symbol: "EURUSD",
      type: "BUY",
      strength: 85
  })
```

---

### 6. 📂 `services/` - خدمات

**چرا وجود دارد:**
- فراهم‌کردن عملکردهای تخصصی
- جداسازی نگرانی‌ها
- قابل‌استفاده‌شدن در قسمت‌های مختلف

**چه می‌کند:**
```
services/
├── licensing/
│   ├── license_manager.py     # مدیریت گواهی‌نامه
│   ├── activation.py          # فعال‌سازی
│   └── validation.py          # تأیید
│
├── subscription/
│   ├── subscription_manager.py # مدیریت اشتراک
│   ├── billing.py             # صورت‌حساب
│   └── payment_handler.py      # مدیریت پرداخت
│
├── account_management/
│   ├── user_manager.py        # مدیریت کاربران
│   ├── account_manager.py     # مدیریت حساب‌های تجاری
│   └── profile_manager.py     # مدیریت پروفایل
│
├── notifications/
│   ├── telegram_bot.py        # ربات تلگرام
│   ├── email_service.py       # سرویس ایمیل
│   └── sms_service.py         # سرویس پیامک
│
├── reporting/
│   ├── report_generator.py    # تولید گزارش
│   ├── statistics.py          # محاسبات آماری
│   └── export.py              # صادر کردن داده‌ها
│
└── backtesting/
    ├── backtest_engine.py     # موتور بک‌تست
    ├── optimizer.py           # بهینه‌سازی
    └── results_analyzer.py    # تحلیل نتایج
```

**هر سرویس:**

#### `licensing/license_manager.py`
```
وظیفه: مدیریت گواهی‌نامه نرم‌افزار

عملکردها:
  1. تولید گواهی‌نامه
  2. فعال‌سازی گواهی‌نامه
  3. تأیید صحت گواهی‌نامه
  4. جلوگیری از کپی برداری

مثال:
  license = LicenseManager()
  is_valid = license.validate("LICENSE_KEY_12345")
  if is_valid:
      print("گواهی‌نامه معتبر است")
```

#### `telegram_bot.py`
```
وظیفه: ارسال پیام‌های تلگرام

پیام‌هایی که ارسال می‌کند:
  1. "معامله باز شد: EUR/USD BUY @ 1.0850"
  2. "معامله بسته شد: سود 50 دلار"
  3. "اخطار: زیان روزانه به 80% رسید"
  4. "سفارش منتظر: EUR/USD 1.0800"

مثال:
  telegram = TelegramBot(token="123456:ABC")
  telegram.send_message(
      chat_id=789,
      message="معامله جدید: EUR/USD BUY"
  )
```

#### `backtesting/backtest_engine.py`
```
وظیفه: آزمایش استراتژی در تاریخ

فرآیند:
  1. داده‌های تاریخی را بارگذاری کنید
  2. برای هر شمع:
     a. سیگنال تولید کنید
     b. معامله را شبیه‌سازی کنید
     c. نتیجه را ثبت کنید
  3. گزارش نتایج را ایجاد کنید

خروجی:
  - تعداد معاملات
  - درصد برد
  - حداکثر سود/ضرر
  - Sharpe Ratio
```

---

### 7. 📂 `integrations/` - یکپارچه‌سازی‌ها

**چرا وجود دارد:**
- اتصال به خدمات خارجی
- دریافت داده‌های خارجی
- استفاده از API‌های شخص ثالث

**چه می‌کند:**
```
integrations/
├── news_api/
│   ├── news_fetcher.py        # دریافت اخبار
│   ├── sentiment_analyzer.py  # تحلیل احساس
│   └── economic_calendar.py   # تقویم اقتصادی
│
├── exchange_api/
│   ├── binance_connector.py   # اتصال Binance
│   ├── forex_data.py          # داده‌های فارکس
│   └── crypto_data.py         # داده‌های رمزارز
│
├── payment/
│   ├── stripe_handler.py      # اتصال Stripe
│   └── paypal_handler.py      # اتصال PayPal
│
└── external/
    ├── openai_connector.py    # اتصال OpenAI
    └── external_signals.py    # سیگنال‌های خارجی
```

---

### 8. 📂 `dashboard/` - داشبورد

**چرا وجود دارد:**
- ارائۀ اطلاعات به کاربران
- نظارت بر وضعیت سیستم
- کنترل تنظیمات

**چه می‌کند:**
```
dashboard/
├── frontend/
│   ├── components/            # اجزای React
│   ├── pages/                 # صفحات
│   ├── styles/                # سبک‌ها
│   └── utils/                 # ابزارها
│
├── backend/
│   ├── api.py                 # API FastAPI
│   ├── websocket.py           # WebSocket
│   └── auth.py                # احراز هویت
│
└── public/
    ├── index.html             # صفحهٔ اصلی
    └── assets/                # منابع
```

**نمای داشبورد:**
```
┌────────────────────────────────────────┐
│ Galaxy Vast Trading Bot™               │
├────────────────────────────────────────┤
│                                        │
│ حساب:                                  │
│  • موجودی: $10,000                    │
│  • سود روز: +$250 (2.5%)              │
│  • معاملات باز: 2                      │
│                                        │
│ معاملات:                               │
│  EUR/USD BUY @ 1.0850 ◄───────────     │
│  GBP/USD SELL @ 1.2700                 │
│                                        │
│ سیگنال‌های منتظر:                      │
│  ✓ USD/JPY (قوت: 78%)                 │
│  ⚠ AUD/USD (قوت: 65%)                 │
│                                        │
└────────────────────────────────────────┘
```

---

### 9. 📂 `tests/` - آزمون‌ها

**چرا وجود دارد:**
- اطمینان از صحت کد
- شناسایی خطاها قبل از استقرار
- اطمینان از ایمنی تغییرات

**چه می‌کند:**
```
tests/
├── unit/
│   ├── test_engine.py
│   ├── test_agents.py
│   ├── test_strategies.py
│   └── test_analysis.py
│
├── integration/
│   ├── test_mt5_connection.py
│   ├── test_database.py
│   └── test_services.py
│
└── fixtures/
    ├── sample_data.py
    └── mock_mt5.py
```

---

### 10. 📂 `config/` - تنظیمات

**چرا وجود دارد:**
- پذیری‌ تنظیم برای محیط‌های مختلف
- امنیت کلیدهای API
- تسهیل توزیع‌شدگی

**چه می‌کند:**
```
config/
├── default.yaml              # تنظیمات پیش‌فرض
├── development.yaml          # تنظیمات توسعه
├── production.yaml           # تنظیمات تولید
├── ftmo.yaml                 # تنظیمات FTMO
├── prop_firm.yaml            # تنظیمات Prop Firm
└── .env.example              # نمونهٔ متغیرهای محیط
```

**محتوای `default.yaml`:**
```yaml
# MT5 Connection
mt5:
  login: 123456
  password: xxxx
  server: "MetaQuotes-Demo"

# Risk Management
risk:
  max_risk_per_trade: 2.0      # %
  max_daily_loss: 5.0          # %
  max_open_trades: 5
  max_correlation: 0.7

# Strategy
strategy:
  use_price_action: true
  use_smc: true
  use_ict: true
  confidence_threshold: 70

# Backtesting
backtest:
  start_date: "2023-01-01"
  end_date: "2024-01-01"
  initial_balance: 10000
```

---

### 11. 📂 `docs/` - مستندات

**چرا وجود دارد:**
- آموزش کاربران
- راهنمایی توسعه‌دهندگان
- ثبت تصمیمات

**چه می‌کند:**
```
docs/
├── user_guide/                # راهنمای کاربر
├── developer_guide/           # راهنمای توسعه‌دهنده
├── architecture/              # مستندات معماری
├── api_reference/             # مرجع API
└── tutorials/                 # آموزش‌ها
```

---

### 12. 📂 `scripts/` - اسکریپت‌ها

**چرا وجود دارد:**
- اتمتیشن کارهای تکراری
- تسهیل مدیریت سیستم
- بکاپ و تصفیه

**چه می‌کند:**
```
scripts/
├── setup.sh                   # راه‌اندازی سیستم
├── start.sh                   # شروع ربات
├── backup.py                  # پشتیبان‌گیری
├── cleanup.py                 # تصفیه
└── monitor.py                 # نظارت
```

---

### 13. 📂 `deploy/` - استقرار

**چرا وجود دارد:**
- مدیریت استقرار
- تنظیم محیط‌های مختلف
- خودکارسازی نصب

**چه می‌کند:**
```
deploy/
├── docker/
│   ├── Dockerfile             # تعریف تصویر
│   ├── docker-compose.yml     # ترکیب سرویس‌ها
│   └── .dockerignore          # فایل‌های نادیده
│
├── kubernetes/
│   ├── deployment.yaml        # استقرار
│   └── service.yaml           # سرویس
│
└── ansible/
    ├── playbook.yml           # Playbook
    └── roles/                 # نقش‌ها
```

---

### 14. 📂 `logs/` - گزارش‌ها

**چرا وجود دارد:**
- ثبت تمام رویدادها
- تشخیص خطاها
- تحلیل عملکرد

**چه می‌کند:**
```
logs/
├── 2024-01/
│   ├── trading.log            # معاملات
│   ├── system.log             # سیستم
│   ├── errors.log             # خطاها
│   └── performance.log        # عملکرد
```

---

## 📊 نمودار وابستگی‌های فولدرها

```
┌────────────────┐
│   config/      │  ← تمام فولدرها از اینجا تنظیمات می‌خوانند
└────────┬───────┘
         │
┌────────┴────────────────────────────┐
│         core/                        │  ← هسته اصلی
│  (engine, logger, config_manager)   │
└────────┬───────────┬────────────────┘
         │           │
    ┌────┴────┐  ┌───┴────┐
    │agents/  │  │services│  ← تصمیم و خدمات
    └────┬────┘  └───┬────┘
         │            │
    ┌────┴────────────┴─────┐
    │   analysis/            │  ← تحلیل داده‌ها
    │ (price_action, smc)    │
    └────┬───────────────────┘
         │
    ┌────┴─────────────────┐
    │ infrastructure/       │  ← ارتباط و ذخیره‌سازی
    │ (mt5, database)       │
    └───────────────────────┘
```

---

## 🚀 خلاصۀ ساختار

| فولدر | اهمیت | وابستگی |
|-------|------|---------|
| `core/` | ⭐⭐⭐⭐⭐ | کسی - اساس |
| `infrastructure/` | ⭐⭐⭐⭐⭐ | کسی - ارتباط |
| `agents/` | ⭐⭐⭐⭐ | core, analysis |
| `analysis/` | ⭐⭐⭐⭐ | infrastructure |
| `strategies/` | ⭐⭐⭐ | agents, analysis |
| `services/` | ⭐⭐⭐ | infrastructure |
| `integrations/` | ⭐⭐ | services |
| `dashboard/` | ⭐⭐ | services |
| `tests/` | ⭐⭐⭐ | تمام فولدرها |
| `config/` | ⭐⭐ | تمام فولدرها |

---

**🎯 مرحلۀ بعدی:** ایجاد ماژول‌ها به‌ترتیب اولویت
