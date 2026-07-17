# 🔗 نقشهٔ رابطه‌های دیتابیس - DATABASE_DIAGRAM_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 📊 ERD (Entity-Relationship Diagram)

```
┌─────────────────────────────────────────────────────────────────┐
│                     LAYER 1: Authentication                     │
├─────────────────────────────────────────────────────────────────┤

        ┌──────────┐
        │  users   │ (کاربران)
        │ (PK: id) │
        └────┬─────┘
             │
        ┌────┼────────────────────┬─────────────────────┐
        │    │                    │                     │
        │    └──→ licenses     →  subscriptions   →  hardware_bindings
        │    │   (1:N)             (1:N)              (1:N)
        │    │
        │    └──→ trading_accounts (1:N)
        │    │   │
        │    │   ├──→ trades (1:N)
        │    │   │   │
        │    │   │   ├──→ open_positions (1:1)
        │    │   │   └──→ closed_positions (1:1)
        │    │   │
        │    │   ├──→ risk_logs (1:N)
        │    │   └──→ performance_metrics (1:N)
        │    │
        │    ├──→ signals (1:N)
        │    │   │
        │    │   ├──→ ai_decisions (1:1)
        │    │   ├──→ smart_money_analysis (1:1)
        │    │   └──→ price_action_analysis (1:1)
        │    │
        │    ├──→ notifications (1:N)
        │    ├──→ telegram_messages (1:N)
        │    ├──→ email_messages (1:N)
        │    │
        │    ├──→ backtests (1:N)
        │    │   └──→ optimization_results (1:N)
        │    │
        │    ├──→ trading_journal (1:N)
        │    ├──→ screenshots (1:N)
        │    │
        │    └──→ audit_logs (1:N)
        │    └──→ error_logs (1:N)
        │
        └──→ broker_accounts (1:N)
            └──→ trading_accounts (1:N)

┌─────────────────────────────────────────────────────────────────┐
│                 LAYER 2: Analysis & Signals                     │
├─────────────────────────────────────────────────────────────────┤

    signals ─────→ trades (1:N)
       │              │
       ├──→ ai_decisions
       ├──→ ai_explanations
       ├──→ smart_money_analysis
       └──→ price_action_analysis

┌─────────────────────────────────────────────────────────────────┐
│              LAYER 3: Market Data & Strategies                  │
├─────────────────────────────────────────────────────────────────┤

    news_events ─────→ (affects trading)
    economic_calendar ─→ (affects trading)
    
    strategies ────→ backtests (1:N)
                   └──→ optimization_results (1:N)
```

---

## 🔑 نقاط اتصال کلیدی

### 1. Users → Licenses (1:N)
```
users.id → licenses.user_id
→ subscriptions
→ hardware_bindings
```

### 2. Users → Trading Accounts (1:N)
```
users.id → trading_accounts.user_id
→ broker_accounts.id
```

### 3. Trading Accounts → Trades (1:N)
```
trading_accounts.id → trades.trading_account_id
→ open_positions / closed_positions
→ signals → ai_decisions
```

### 4. Signals → AI Decisions (1:1)
```
signals.id → ai_decisions.signal_id (UNIQUE)
→ ai_explanations
```

### 5. Strategies → Backtests (1:N)
```
strategies.id → backtests.strategy_id
→ optimization_results
```

---

## 📈 جریان داده

### Flow 1: معامله جدید
```
1. MarketData → signals (سیگنال تولید می‌شود)
2. signals → ai_decisions (تصمیم گرفته می‌شود)
3. ai_decisions → trades (معامله اجرا می‌شود)
4. trades → open_positions (موضعیت باز می‌شود)
5. open_positions → closed_positions (موضعیت بسته می‌شود)
6. closed_positions → performance_metrics (معیارها محاسبه می‌شود)
```

### Flow 2: یادگیری AI
```
1. trades (تاریخچهٔ معاملات)
2. ai_explanations (توضیح چرا شده)
3. trading_journal (یادداشت‌های کاربر)
4. backtests (بک‌تست روی داده‌های نو)
5. optimization_results (پارامترها بهینه می‌شوند)
```

---

## 🎯 نقاط بحرانی

### High Cardinality (بسیار پر تراکنش)
- **trades**: 10-100 معامله روزانه
- **signals**: 100-1000 سیگنال روزانه
- **audit_logs**: هر عمل ثبت می‌شود
- **error_logs**: هر خطا ثبت می‌شود

### Real-time Updates
- **open_positions**: قیمت جاری بروزرسانی می‌شود
- **performance_metrics**: هر نتیجهٔ معامله
- **risk_logs**: هر رویداد ریسک

### Archive Strategy
- **closed_positions**: آرشیو برای تحلیل تاریخی
- **error_logs**: حداقل 1 سال نگهداری
- **audit_logs**: حداقل 2-3 سال (GDPR)

---

## 🔐 نقاط امنیتی

### Encryption
```
passwords ────→ bcrypt
api_keys ─────→ AES-256
two_fa_secret ─→ encrypted
```

### Access Control
```
users ──→ roles ──→ permissions
  ↓
audit_logs (تمام تغییرات ثبت می‌شود)
```

### Data Isolation
```
trading_account_id (هر کاربر فقط خود را می‌بیند)
user_id (هر رکورد به کاربر متصل است)
```

---

## 📊 نمای کلی شاخص‌ها

```
INDEXES (45+)
├── PRIMARY KEY (38)
├── FOREIGN KEY (50+)
├── UNIQUE (20+)
├── COMPOSITE (10+)
└── JSONB GIN (8)
```

---

**نقشه‌ی کامل برای توسعه آماده است!**
