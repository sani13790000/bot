# 🔗 نقشهٔ رابطه‌های دیتابیس (ERD)

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 📊 نمودار رابطه‌های کلیدی

```
┌─────────────────────────────────────────────────────────────────┐
│                         USERS LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│                          users                                  │
│  (id, email, username, password_hash, 2fa, status)             │
│  PK: id (UUID)                                                 │
│  UK: email, username                                           │
└────────────────┬────────────────────────────────────────────────┘
                 │
        ┌────────┼────────┬──────────┬──────────┐
        │        │        │          │          │
    (1:N)   (1:N) │    (1:N)  (1:N)
        │        │        │          │          │
        ▼        ▼        ▼          ▼          ▼
┌──────────┐ ┌────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│licenses  │ │trading_    │ │signals   │ │backtests │ │audit_    │
│(1:N subs)│ │accounts    │ │          │ │          │ │logs      │
│hardware_ │ └────┬───────┘ │          │ │          │ │          │
│bindings  │ (1:N)│         │          │ │          │ │          │
└──────────┘      │      (1:N)      (1:N) │          │          │
                  │         │             │          │          │
            (1:N) │    (1:1)│             │          │          │
                  ▼        ▼              ▼          ▼          ▼
            ┌──────────┐ ┌──────────┐  ┌──────────┐ ┌──────────┐
            │trades    │ │ai_       │  │open_     │ │error_    │
            │(signals) │ │decisions │  │positions │ │logs      │
            └────┬─────┘ │(factors) │  │(current) │ │          │
                 │       └──────────┘  └──────────┘ └──────────┘
            (1:1)│
                 │     ┌──────────────────────┐
                 └────▶│closed_positions      │
                       │(realized P&L)        │
                       └──────────────────────┘
```

---

## 🔄 جریان داده‌های اصلی

### Trade Workflow
```
Signal Created
    ↓ (1:1)
AI Decision
    ↓ (APPROVE)
Trade Created
    ↓ (1:1)
Open Position
    ↓ (trade runs)
Risk Logs (if needed)
    ↓ (trade closes)
Closed Position
    ↓ (1:1)
Performance Metrics
    ↓
Audit Logs (all changes)
```

### Analysis Workflow
```
Market Data (from MT5)
    ↓
News Events
    ↓
Economic Calendar
    ↓
Signal Generated (multiple sources)
    ├─ Price Action Analysis
    ├─ SMC Analysis
    ├─ Technical Analysis
    │
    ▼
AI Decision (APPROVE/REJECT/MODIFY)
    ↓
AI Explanations (for user)
    ↓
Trade Creation (if approved)
```

---

## 🎯 رابطه‌های حساس

### Critical 1:1 Relations
- trade → open_position (تنها یکی)
- trade → closed_position (تنها یکی)
- signal → ai_decision (تنها یکی)

### Critical N:1 Relations
- trades → trading_accounts (نسبت بالا)
- signals → users (نسبت بالا)
- audit_logs → users (نسبت بالا)

---

## 📈 Index Strategy

### High Priority (سرعت بحرانی)
```sql
-- Users
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);

-- Licenses
CREATE INDEX idx_licenses_user_id ON licenses(user_id);
CREATE INDEX idx_licenses_status ON licenses(status);
CREATE INDEX idx_licenses_expiration ON licenses(expiration_date);

-- Trading
CREATE INDEX idx_trades_account ON trades(trading_account_id);
CREATE INDEX idx_trades_pair ON trades(pair);
CREATE INDEX idx_trades_status ON trades(status);
CREATE INDEX idx_trades_time ON trades(entry_time);

-- Signals
CREATE INDEX idx_signals_pair ON signals(pair);
CREATE INDEX idx_signals_source ON signals(source);
CREATE INDEX idx_signals_status ON signals(status);

-- Audit
CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
```

---

## 🔐 Data Integrity

### Foreign Key Constraints
- تمام FKs دارای ON DELETE CASCADE یا SET NULL
- تمام تصحیح خودکار

### Unique Constraints
- licenses.license_key
- licenses.hardware_id + license_id
- performance_metrics (account + period)
- user_roles (user_id + role_id)
- role_permissions (role_id + permission_id)

---

## ✨ نکات مهم

✅ تمام جداول دارای UUID PK (بجای SERIAL)
✅ تمام جداول دارای created_at و updated_at
✅ Soft Deletes برای sensitive data
✅ JSONB برای flexibility
✅ Enums برای enum types
✅ تمام FKs protected

---

**رابطه‌های دیتابیس برای تولید آماده است! 🚀**
