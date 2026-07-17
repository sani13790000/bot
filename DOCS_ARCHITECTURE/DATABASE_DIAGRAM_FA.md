# 📊 DATABASE_DIAGRAM_FA.md - نقشهٔ رابطه‌ها

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🔗 ERD - نمودار رابطه‌های موجودیت

### رابطه‌های اصلی

```
users (مرکز)
├── roles (N-N via user_roles)
├── licenses (1-N)
│   ├── subscriptions (1-N)
│   └── hardware_bindings (1-N)
├── broker_accounts (1-N)
│   └── trading_accounts (1-N)
│       ├── trades (1-N)
│       │   ├── open_positions (1-1)
│       │   ├── closed_positions (1-1)
│       │   ├── ai_explanations (1-N)
│       │   └── screenshots (1-N)
│       ├── signals (1-N)
│       │   ├── ai_decisions (1-1)
│       │   ├── smart_money_analysis (1-1)
│       │   └── price_action_analysis (1-1)
│       ├── risk_logs (1-N)
│       └── trading_journal (N-1)
├── signals (N-N)
├── backtests (1-N)
│   ├── strategies (N-1)
│   └── optimization_results (1-N)
├── notifications (1-N)
├── telegram_messages (1-N)
├── email_messages (1-N)
├── bale_messages (1-N)
├── audit_logs (1-N)
└── error_logs (1-N)
```

### جریان داده اساسی

```
Market Data (MT5)
    ↓
Trading Accounts
    ├─→ Trades (ایجاد)
    │   ├─→ Open Positions
    │   └─→ Closed Positions
    │
    ├─→ Signals (تولید)
    │   ├─→ AI Decisions
    │   ├─→ Smart Money Analysis
    │   └─→ Price Action Analysis
    │
    └─→ Risk Logs (نظارت)
        └─→ Notifications (اطلاع)
```

---

**✅ نقشهٔ رابطه‌ها کامل است!**
