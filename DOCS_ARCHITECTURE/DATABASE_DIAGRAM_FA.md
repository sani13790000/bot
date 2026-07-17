# 📊 DATABASE_DIAGRAM_FA.md - نقشهٔ رابطه‌های دیتابیس

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🔗 ERD - Entity Relationship Diagram

### Layer 1: Authentication

```
┌─────────────────────────┐
│       users             │
│  (id, email, username)  │
└────────────┬────────────┘
             │
        ┌────┼────┐
        │         │
     (1:N)    (1:N)
        │         │
    ┌───▼───┐  ┌──▼──────┐
    │roles  │  │licenses  │
    └───────┘  └──┬───────┘
         ▲        │
         │    (1:N)
      (N:N)       │
         │    ┌───▼──────────────────┐
    ┌────┴────────────────────┐     │
    │ role_permissions        │     │
    │ (role_id, perm_id)      │     │
    └─────────────────────────┘  ┌──▼────────┐
                                  │subscriptions│
                                  │(plan, price)│
    ┌─────────────────────────┐   └──────────┘
    │ permissions             │
    │ (name, module, action)  │
    └─────────────────────────┘
```

### Layer 2-3: Accounts

```
┌──────────────────────────┐
│ broker_accounts          │
│ (MT5/Binance connection) │
└────────────┬─────────────┘
             │
         (1:N)
             │
        ┌────▼──────────────┐
        │ trading_accounts   │
        │ (balance, equity)  │
        └────────┬───────────┘
                 │
        ┌────────┼────────────────┐
        │        │                │
    (1:N)   (1:N)            (1:N)
        │        │                │
   ┌────▼──┐ ┌──▼────┐  ┌────────▼────┐
   │trades │ │signals│  │risk_logs    │
   └───────┘ └──────┘   └─────────────┘
```

### Layer 4: Trading Operations

```
┌─────────────────────────────────────┐
│ trades (معاملات)                    │
│ (pair, entry/exit, P&L)             │
└────┬──────────────────────┬─────────┘
     │                      │
 (1:1)│                  (1:1)
     │                      │
┌────▼───────────────┐  ┌──▼─────────────┐
│ open_positions     │  │ closed_positions│
│ (current price)    │  │ (realized P&L)  │
└────────────────────┘  └─────────────────┘

     │
 (1:1)
     │
┌────▼──────────────────┐
│ ai_explanations       │
│ (explanation text)    │
└───────────────────────┘
```

### Layer 5: Signals & Analysis

```
┌──────────────────────────┐
│ signals                  │
│ (pair, type, strength)   │
└──────┬───────────────────┘
       │
   (1:1)
       │
┌──────▼────────────────────┐
│ ai_decisions              │
│ (APPROVE/REJECT/MODIFY)   │
└───────────────────────────┘

       │
   (1:N)
       │
  ┌────┴──────────────────────────────────┐
  │                                       │
┌─▼──────────────────┐  ┌────────────────▼──┐
│smart_money_analysis│  │price_action_      │
│(Order Blocks)      │  │analysis (patterns)│
└────────────────────┘  └───────────────────┘
```

### Full Database Relationship Map

```
users ──┬── roles ──┬── permissions
        │           │
        ├── licenses ├── subscriptions
        │           │
        ├── broker_accounts ── trading_accounts ──┬── trades
        │                                         │
        ├── signals ──── ai_decisions             ├── open_positions
        │                │                        │
        │                └── ai_explanations      └── closed_positions
        │
        ├── backtests ── optimization_results
        │
        ├── strategies ── strategy_configurations
        │
        ├── notifications
        ├── telegram_messages
        ├── email_messages
        ├── screenshots
        │
        ├── audit_logs
        ├── error_logs
        │
        └── system_settings

Additional Connections:
news_events → affected_pairs (JSONB)
economic_calendar → (no direct FK)
risk_logs → trading_accounts
trading_journal → trades + trading_accounts
performance_metrics → trading_accounts
```

---

## 🔑 Key Relationships

### 1:1 Relationships
- trade ↔ open_position
- trade ↔ closed_position
- signal ↔ ai_decision

### 1:N Relationships
- users → licenses
- users → trading_accounts
- users → signals
- users → backtests
- licenses → subscriptions
- broker_accounts → trading_accounts
- trading_accounts → trades
- trading_accounts → open_positions
- trading_accounts → closed_positions
- trading_accounts → risk_logs
- trading_accounts → performance_metrics
- signals → trades
- signals → ai_explanations
- backtests → optimization_results
- strategies → backtests
- strategies → strategy_configurations

### N:N Relationships
- users ↔ roles (via user_roles)
- roles ↔ permissions (via role_permissions)

---

**نقشهٔ رابطه‌های دیتابیس برای تولید آماده است!**
