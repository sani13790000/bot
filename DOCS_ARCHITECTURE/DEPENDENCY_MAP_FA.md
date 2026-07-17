# 🔗 نقشهٔ وابستگی‌های سیستم

**نسخه:** 1.0  
**تاریخ:** 1403/04/27  
**وضعیت:** ✅ نهایی

---

## 📋 فهرست محتوا

1. [نمودارهای وابستگی](#نمودارهای-وابستگی)
2. [وابستگی‌های هر ماژول](#وابستگی‌های-هر-ماژول)
3. [جریان داده](#جریان-داده)
4. [نقاط بحرانی](#نقاط-بحرانی)
5. [ماتریس وابستگی](#ماتریس-وابستگی)

---

## نمودارهای وابستگی

### 📊 نمودار ۱: وابستگی‌های سطح بالا

```
┌─────────────────────────────────────────────────────────┐
│              Dashboard (رابط کاربری)                    │
│           React + Vue Components                        │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────┴────────────────────────────────────┐
│                 API Gateway (درگاه API)                 │
│              FastAPI WebSocket                          │
└────────────────────┬────────────────────────────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───┴────┐      ┌───┴────┐      ┌───┴────┐
│ Core   │      │Services│      │ Agents │
│Engine  │      │Manager │      │System  │
└───┬────┘      └───┬────┘      └───┬────┘
    │                │                │
    ├────────────────┼────────────────┤
    │                │                │
┌───┴────────────────┴────────────────┴───┐
│    Infrastructure & Analysis Modules    │
│                                          │
│  ├─ MT5 Connection                      │
│  ├─ Database (PostgreSQL)               │
│  ├─ Cache (Redis)                       │
│  ├─ Message Queue (RabbitMQ)            │
│  ├─ Price Action Engine                 │
│  ├─ SMC Analysis                        │
│  ├─ ICT Concepts                        │
│  └─ News Intelligence                   │
└──────────────────────────────────────────┘
```

### 📊 نمودار ۲: جریان داده (Data Flow)

```
MetaTrader 5 (بازار)
    ↓ (داده‌های لحظه‌ای)
    │
┌─────────────────────────────────────┐
│   Market Data Receiver              │
│   (دریافت‌کننده داده‌های بازار)     │
└─────────┬───────────────────────────┘
          │
          ↓
    ┌──────────────┐
    │ Data Cache   │ (ریدیس)
    │ (کش داده‌ها)  │
    └─────┬────────┘
          │
    ┌─────┴─────────────────────────────────────────────┐
    │                                                   │
    ↓                                                   ↓
┌─────────────────────┐                    ┌─────────────────────┐
│ Analysis Engines    │                    │ Event Dispatcher    │
│                     │                    │                     │
│ • Price Action      │                    │ توزیع رویدادها     │
│ • SMC               │                    │                     │
│ • ICT               │                    └────────┬────────────┘
│ • Technical         │                             │
│                     │                             ↓
└──────────┬──────────┘                    ┌──────────────────┐
           │                               │  Event Listeners │
           │                               │                  │
           ├──────────────┬────────────────┤ • Agents        │
           │              │                │ • Dashboard     │
           │              │                │ • Services      │
           ↓              ↓                └──────────────────┘
      ┌─────────────────────────┐
      │   Signal Generator      │
      │   (تولید‌کننده سیگنال)   │
      └──────────┬──────────────┘
                 │
                 ↓
      ┌─────────────────────────┐
      │   Decision Agent        │
      │   (تصمیم‌گیر)           │
      └──────────┬──────────────┘
                 │
      ┌──────────┴──────────────┐
      │ Risk Agent Validation   │
      └──────────┬──────────────┘
                 │
    ┌────────────┴────────────┐
    │ YES          │       NO │
    ↓              │         ↓
┌─────────┐        │    ┌──────────────┐
│ Order   │        │    │ Reject Trade │
│ Manager │        │    │ (رد معامله)  │
└────┬────┘        │    └──────────────┘
     │             │
     └─────────────┴─────────────┐
                                 │
                   ┌─────────────┴──────────────┐
                   │                            │
                   ↓                            ↓
            ┌────────────────┐         ┌─────────────────┐
            │ MT5 Execution  │         │ Database Logger │
            │ (اجرای معامله) │         │ (ثبت‌کننده DB)  │
            └────────┬───────┘         └─────────────────┘
                     │
                     ↓
            ┌─────────────────┐
            │ Trade Monitor   │
            │ (نظارت‌کننده)   │
            └────────┬────────┘
                     │
         ┌───────────┴──────────┐
         │                      │
         ↓                      ↓
    ┌──────────┐        ┌──────────────┐
    │ Close    │        │ Notifications│
    │ Signal   │        │ (اطلاعات)   │
    └─┬────────┘        └──────────────┘
      │
      ↓
  Learning Agent (یادگیری)
```

---

## وابستگی‌های هر ماژول

### 🔴 Core Engine (هسته)

```
core/engine.py
├── ✓ مستقل‌تر (اساس است)
├─→ requires: config_manager, logger, event_dispatcher
├─→ provides: lifecycle management
└─→ used_by: تمام ماژول‌ها

core/config_manager.py
├── ✓ مستقل‌تر
├─→ requires: yaml, environment
├─→ provides: configuration
└─→ used_by: تمام ماژول‌ها

core/event_dispatcher.py
├─→ requires: logging
├─→ provides: event system
└─→ used_by: تمام ماژول‌ها

core/logger.py
├── ✓ مستقل‌تر
├─→ requires: logging module
├─→ provides: logging system
└─→ used_by: تمام ماژول‌ها
```

### 🟡 Infrastructure (زیرساخت)

```
infrastructure/mt5/connection.py
├─→ requires: mt5 library, config
├─→ provides: market connection, order execution
└─→ used_by: agents, services, analysis

infrastructure/database/models.py
├─→ requires: sqlalchemy, config
├─→ provides: data persistence
└─→ used_by: services, logging, reporting

infrastructure/cache/redis_cache.py
├─→ requires: redis library, config
├─→ provides: fast data access
└─→ used_by: analysis engines, agents

infrastructure/messaging/event_bus.py
├─→ requires: rabbitmq library
├─→ provides: event distribution
└─→ used_by: agents, services, dashboard
```

### 🟢 Analysis Engines (موتورهای تحلیل)

```
analysis/price_action/pattern_detector.py
├─→ requires: market_data, historical_prices
├─→ provides: price patterns
└─→ used_by: agents, strategies

analysis/smc/order_block_finder.py
├─→ requires: market_data, volatility
├─→ provides: smart money signals
└─→ used_by: agents, strategies

analysis/ict/kill_zones.py
├─→ requires: market_data, timezone_info
├─→ provides: time-based signals
└─→ used_by: agents

analysis/technical/indicators.py
├─→ requires: market_data
├─→ provides: technical indicators
└─→ used_by: analysis engines, agents
```

### 🔵 Agents (عاملین)

```
agents/decision_agent.py
├─→ requires: signal_data, risk_parameters
├─→ requires: price_action, smc, ict engines
├─→ provides: buy/sell decisions
└─→ used_by: order_manager

agents/risk_agent.py
├─→ requires: account_data, decision_data
├─→ requires: risk_parameters
├─→ provides: trade validation
└─→ used_by: order_manager

agents/opportunity_agent.py
├─→ requires: analysis engines
├─→ provides: trade opportunities
└─→ used_by: decision_agent

agents/learning_agent.py
├─→ requires: trade_history, results
├─→ provides: performance insights
└─→ used_by: parameter optimizer

agents/monitor_agent.py
├─→ requires: open_trades, market_data
├─→ provides: trade monitoring
└─→ used_by: dashboard, notifications
```

### 🟣 Services (خدمات)

```
services/licensing/license_manager.py
├─→ requires: database, encryption
├─→ provides: license validation
└─→ used_by: core engine, dashboard

services/subscription/subscription_manager.py
├─→ requires: database, payment services
├─→ provides: subscription management
└─→ used_by: user manager, dashboard

services/account_management/user_manager.py
├─→ requires: database, authentication
├─→ provides: user management
└─→ used_by: dashboard, core engine

services/notifications/telegram_bot.py
├─→ requires: telegram library, config
├─→ provides: notifications
└─→ used_by: agents, services

services/reporting/report_generator.py
├─→ requires: database, statistics
├─→ provides: trade reports
└─→ used_by: dashboard, export

services/backtesting/backtest_engine.py
├─→ requires: strategies, historical_data
├─→ requires: analysis engines
├─→ provides: backtesting results
└─→ used_by: dashboard, optimizer
```

### 🟠 Integrations (یکپارچه‌سازی‌ها)

```
integrations/news_api/news_fetcher.py
├─→ requires: news_api library, config
├─→ provides: economic news
└─→ used_by: analysis engines

integrations/exchange_api/binance_connector.py
├─→ requires: binance library, config
├─→ provides: exchange data
└─→ used_by: analysis engines (optional)

integrations/payment/stripe_handler.py
├─→ requires: stripe library, config
├─→ provides: payment processing
└─→ used_by: subscription manager
```

---

## جریان داده

### 📍 مسیر ۱: تحلیل بازار (Market Analysis)

```
1. Market Data (from MT5)
   └→ Size: ~1KB per tick
   └→ Frequency: Every 1 second
   
2. Data Cache (Redis)
   └→ Keep last 10000 candles
   └→ TTL: 24 hours
   
3. Analysis Engines (Parallel)
   ├→ Price Action (Processing: 100ms)
   ├→ SMC Analysis (Processing: 150ms)
   └→ ICT Analysis (Processing: 100ms)
   
4. Signal Generation
   └→ Aggregate signals
   └→ Calculate confidence
   
5. Event Broadcasting
   └→ Emit to all listeners
```

### 📍 مسیر ۲: اجرای معامله (Trade Execution)

```
1. Signal Generated (Confidence > 70%)
   
2. Opportunity Agent
   └→ Validate opportunity
   └→ Suggest entry/exit
   
3. Decision Agent
   └→ Aggregate signals
   └→ Make decision
   
4. Risk Agent
   ├→ Check balance
   ├→ Check daily loss limit
   ├→ Check position size
   └→ Approve/Reject
   
5. Order Manager
   └→ Send to MT5
   └→ Receive confirmation
   
6. Trade Monitor
   └→ Track P&L
   └→ Watch for exit signals
   
7. Learning Agent
   └→ Record outcome
   └→ Update parameters
```

### 📍 مسیر ۳: رویدادها و اطلاعات (Events & Notifications)

```
Event Generated
  │
  └→ Event Bus (RabbitMQ)
     │
     ├→ Database Logger
     │   └→ PostgreSQL
     │
     ├→ Telegram Bot
     │   └→ Send message
     │
     ├→ Dashboard WebSocket
     │   └→ Update UI
     │
     └→ Email Service
         └→ Send email (if configured)
```

---

## نقاط بحرانی

### ⚠️ نقطهٔ حساس ۱: MT5 Connection

```
اگر اتصال MT5 قطع شود:
  1. Monitor Agent متوجه می‌شود
  2. Exponential backoff retry
  3. اگر ۳ دقیقه ادامه داشت:
     a. تمام معاملات باز بسته شود
     b. Alert ارسال شود
     c. Engine متوقف شود

وابستگی:
  - infrastructure/mt5/connection.py (اصلی)
  - agents/monitor_agent.py
  - services/notifications/telegram_bot.py
```

### ⚠️ نقطهٔ حساس ۲: Database Connection

```
اگر پایگاه داده در دسترس نباشد:
  1. Cache (Redis) به عنوان backup
  2. In-memory storage موقتی
  3. اگر ۱ ساعت ادامه داشت:
     a. Engine متوقف شود
     b. Sync قفل شود

وابستگی:
  - infrastructure/database/models.py (اصلی)
  - infrastructure/cache/redis_cache.py (backup)
```

### ⚠️ نقطهٔ حساس ۳: Order Execution

```
اگر سفارش اجرا نشود:
  1. Retry ۳ بار
  2. اگر ناموفق:
     a. Manual override مورد نیاز است
     b. Alert به تلگرام فرستاده شود
     c. Log تفصیلی ثبت شود

وابستگی:
  - infrastructure/mt5/order_manager.py
  - services/notifications/telegram_bot.py
  - core/logger.py
```

---

## ماتریس وابستگی

### جدول ۱: وابستگی‌های ماژول

| ماژول | Core | DB | Cache | MT5 | Analysis | Agents |
|-------|------|-----|-------|-----|----------|--------|
| **Core** | - | ✓ | - | - | - | - |
| **Infrastructure** | ✓ | - | - | - | - | - |
| **Analysis** | ✓ | - | ✓ | - | - | - |
| **Agents** | ✓ | ✓ | ✓ | ✓ | ✓ | - |
| **Services** | ✓ | ✓ | ✓ | ✓ | - | ✓ |
| **Dashboard** | ✓ | ✓ | ✓ | - | - | ✓ |

### جدول ۲: درجات وابستگی

```
وابستگی قوی (Strong):
  • agents ← analysis engines
  • agents ← risk parameters
  • services ← database
  
وابستگی متوسط (Medium):
  • analysis ← market data cache
  • services ← MT5 connection
  
وابستگی ضعیف (Weak):
  • dashboard ← news api
  • integrations ← services
```

---

## 🔄 چرخه‌های وابستگی (Circular Dependencies)

### ✅ موارد تجنب شده:

```
❌ AVOID: Analysis ← Agent ← Analysis
   (موجب بلاکی می‌شود)

✅ BETTER: 
   Agents subscribe to Analysis events
   (استفاده از Event Bus)

❌ AVOID: MT5 ← Service ← Analysis ← MT5
   (چرخهٔ وابستگی)

✅ BETTER:
   Dependency Injection Pattern
   (تزریق وابستگی)
```

---

## 📦 نمایش وابستگی‌های Pip

### requirements.txt مثال

```
# Core
pydantic==2.0.0              # Data validation
python-dotenv==1.0.0        # Environment variables

# MT5 Integration
MetaTrader5==5.0.45

# Database
sqlalchemy==2.0.0
psycopg2-binary==2.9.0       # PostgreSQL driver
alembic==1.12.0              # Migrations

# Caching
redis==5.0.0

# Message Queue
pika==1.3.0                  # RabbitMQ

# API
fastapi==0.104.0
uvicorn==0.24.0
websockets==11.0.0

# Data Analysis
pandas==2.0.0
numpy==1.24.0
scipy==1.11.0

# Machine Learning
tensorflow==2.14.0           # Or PyTorch
scikit-learn==1.3.0

# News & Intelligence
newsapi==0.1.1
textblob==0.17.1            # Sentiment analysis

# Notifications
python-telegram-bot==20.0

# Email
python-multipart==0.0.6
aiosmtplib==2.1.0

# Testing
pytest==7.4.0
pytest-asyncio==0.21.0

# Monitoring
prometheus-client==0.18.0

# Dashboard Frontend (Node.js)
# npm install react@18 vue@3 axios tailwindcss
```

---

## 🚀 ترتیب بوت‌استرپ (Startup Sequence)

```
1️⃣ Load Configuration
   └→ core/config_manager.py
   
2️⃣ Initialize Logger
   └→ core/logger.py
   
3️⃣ Connect to Database
   └→ infrastructure/database/
   
4️⃣ Connect to Cache
   └→ infrastructure/cache/redis_cache.py
   
5️⃣ Connect to Message Queue
   └→ infrastructure/messaging/event_bus.py
   
6️⃣ Connect to MT5
   └→ infrastructure/mt5/connection.py
   
7️⃣ Initialize Event Dispatcher
   └→ core/event_dispatcher.py
   
8️⃣ Start Analysis Engines
   └→ analysis/*/
   
9️⃣ Start Agents
   └→ agents/*/
   
🔟 Start Services
   └→ services/*/
   
1️⃣1️⃣ Start Dashboard
   └→ dashboard/backend/api.py
   
1️⃣2️⃣ Start Monitoring
   └→ agents/monitor_agent.py
```

---

## 🛡️ مدیریت خطاها (Error Handling)

### سطح ۱: Local Error Handling

```python
# در هر ماژول
try:
    result = analysis_engine.analyze(data)
except AnalysisException as e:
    logger.error(f"Analysis failed: {e}")
    event_dispatcher.emit("ANALYSIS_ERROR", e)
    return None
```

### سطح ۲: Service-Level Error Handling

```python
# در services
try:
    execute_trade(signal)
except OrderExecutionException as e:
    risk_agent.validate_fallback()
    telegram_bot.send_alert(f"Order failed: {e}")
```

### سطح ۳: Global Error Handling

```python
# در core engine
try:
    engine.run()
except Exception as e:
    logger.critical(f"Engine crashed: {e}")
    notification_service.alert_admin()
    shutdown_gracefully()
```

---

## 📊 خلاصهٔ وابستگی‌ها

### ماژول‌های مستقل:
- ✅ core/logger.py
- ✅ core/config_manager.py
- ✅ analysis engines (تا حدی)

### ماژول‌های وابسته:
- 🔴 agents (وابسته به analysis + infrastructure)
- 🔴 services (وابسته به database + MT5)
- 🔴 dashboard (وابسته به همه‌ چیز)

### ماژول‌های میانی:
- 🟡 infrastructure (کم‌وبیش مستقل)
- 🟡 integrations (اختیاری)

---

**🎯 اصول طراحی:**
1. **Loose Coupling** - ماژول‌ها از هم مستقل باشند
2. **High Cohesion** - هر ماژول یک کار خاص انجام دهد
3. **Event-Driven** - رویدادها برای ارتباط استفاده شوند
4. **Dependency Injection** - وابستگی‌ها تزریق شوند
5. **Graceful Degradation** - سیستم بتواند با خرابی‌ها کار کند

---

**مرحلۀ بعدی:** ایجاد فایل ARCHITECTURE_FA.md با توضیح معماری فنی
