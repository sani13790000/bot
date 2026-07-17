# 📑 استراتژی شاخص‌گذاری - INDEXING_STRATEGY_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🎯 استراتژی شاخص‌گذاری

### اصول
1. **Selectivity بالا** - شاخص فقط برای ستون‌های انتخابی
2. **Query Pattern** - شاخص بر اساس جستجوهای عملی
3. **Write Performance** - تعادل بین Read و Write
4. **Storage** - هر شاخص اضافی فضا مصرف می‌کند

---

## 📋 شاخص‌های Priority 1 (الویت بالا)

### users
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created ON users(created_at DESC);
```

### licenses
```sql
CREATE INDEX idx_licenses_user_id ON licenses(user_id);
CREATE INDEX idx_licenses_status ON licenses(status);
CREATE INDEX idx_licenses_expiration ON licenses(expiration_date);
CREATE INDEX idx_licenses_key ON licenses(license_key);
```

### trading_accounts
```sql
CREATE INDEX idx_trading_accounts_user_id ON trading_accounts(user_id);
CREATE INDEX idx_trading_accounts_status ON trading_accounts(status);
CREATE INDEX idx_trading_accounts_balance ON trading_accounts(balance);
```

### trades
```sql
CREATE INDEX idx_trades_account ON trades(trading_account_id);
CREATE INDEX idx_trades_pair ON trades(pair);
CREATE INDEX idx_trades_status ON trades(status);
CREATE INDEX idx_trades_time ON trades(entry_time DESC);
CREATE INDEX idx_trades_pair_time ON trades(pair, entry_time DESC);
```

### signals
```sql
CREATE INDEX idx_signals_pair ON signals(pair);
CREATE INDEX idx_signals_source ON signals(source);
CREATE INDEX idx_signals_status ON signals(status);
CREATE INDEX idx_signals_time ON signals(created_at DESC);
CREATE INDEX idx_signals_pair_time ON signals(pair, created_at DESC);
```

### open_positions
```sql
CREATE INDEX idx_open_positions_account ON open_positions(trading_account_id);
CREATE INDEX idx_open_positions_pair ON open_positions(pair);
```

### notifications
```sql
CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_read ON notifications(user_id, is_read);
CREATE INDEX idx_notifications_type ON notifications(type);
```

---

## 📑 شاخص‌های Priority 2 (الویت متوسط)

### performance_metrics
```sql
CREATE UNIQUE INDEX idx_perf_metric_unique 
ON performance_metrics(trading_account_id, period_start, period_end);
CREATE INDEX idx_perf_metric_account ON performance_metrics(trading_account_id);
```

### news_events
```sql
CREATE INDEX idx_news_time ON news_events(scheduled_time DESC);
CREATE INDEX idx_news_impact ON news_events(impact);
CREATE INDEX idx_news_pairs ON news_events USING GIN(affected_pairs);
```

### economic_calendar
```sql
CREATE INDEX idx_econ_time ON economic_calendar(scheduled_time DESC);
CREATE INDEX idx_econ_country ON economic_calendar(country);
```

### risk_logs
```sql
CREATE INDEX idx_risk_account ON risk_logs(trading_account_id);
CREATE INDEX idx_risk_severity ON risk_logs(severity);
CREATE INDEX idx_risk_time ON risk_logs(created_at DESC);
```

### audit_logs
```sql
CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at DESC);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
```

### error_logs
```sql
CREATE INDEX idx_error_service ON error_logs(service, created_at DESC);
CREATE INDEX idx_error_level ON error_logs(error_level);
CREATE INDEX idx_error_user ON error_logs(user_id);
```

---

## 🔍 شاخص‌های JSONB (GIN)

### strategies
```sql
CREATE INDEX idx_strategies_params ON strategies USING GIN(parameters);
```

### ai_decisions
```sql
CREATE INDEX idx_ai_factors ON ai_decisions USING GIN(factors);
```

### optimization_results
```sql
CREATE INDEX idx_opt_params ON optimization_results USING GIN(parameter_set);
CREATE INDEX idx_opt_metrics ON optimization_results USING GIN(metrics);
```

### signals
```sql
CREATE INDEX idx_signals_pairs ON signals USING GIN(affected_pairs);
```

---

## 🔄 Composite Indexes (شاخص‌های ترکیبی)

### جستجوی متکرر
```sql
-- برای سرعت جستجوی ترکیبی
CREATE INDEX idx_trades_filter ON trades(trading_account_id, pair, status, entry_time DESC);
CREATE INDEX idx_signals_filter ON signals(pair, source, status, created_at DESC);
CREATE INDEX idx_positions_filter ON open_positions(trading_account_id, pair, direction);
```

---

## 📊 Partial Indexes (شاخص‌های محدود)

### فقط برای رکورد‌های فعال
```sql
-- صرف‌فه کاهی برای جستجوهای شایع
CREATE INDEX idx_active_licenses 
ON licenses(user_id) WHERE status = 'active';

CREATE INDEX idx_active_trades 
ON trades(trading_account_id) WHERE status = 'open';

CREATE INDEX idx_unread_notifications 
ON notifications(user_id) WHERE is_read = false;
```

---

## ⚡ Covering Indexes (شاخص‌های پوشش‌دهنده)

### شاخص شامل ستون‌های اضافی
```sql
-- PostgreSQL 11+ INCLUDE Clause
CREATE INDEX idx_trades_cover ON trades(trading_account_id) 
INCLUDE (entry_price, exit_price, status);

CREATE INDEX idx_signals_cover ON signals(pair, source) 
INCLUDE (signal_type, strength, status);
```

---

## 🎯 استراتژی بهینه‌سازی

### 1. Index Bloat Prevention
```sql
-- بروزرسانی آمار شاخص‌ها
REINDEX INDEX idx_trades_time;
ANALYZE trades;
```

### 2. Unused Index Detection
```sql
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE 'pk_%';
```

### 3. Index Monitoring
```sql
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

---

## 📈 مقیاس‌پذیری

### Index Partitioning
```sql
-- برای جداول بزرگ (trades, signals)
CREATE INDEX idx_trades_2024_01 
ON trades(trading_account_id) 
WHERE entry_time >= '2024-01-01' AND entry_time < '2024-02-01';
```

### Parallel Index Build
```sql
-- PostgreSQL 11+ - ساخت شاخص در موازی
SET max_parallel_maintenance_workers = 4;
CREATE INDEX CONCURRENTLY idx_trades_new ON trades(trading_account_id);
```

---

## ✅ Checklist

- [x] Primary Keys (38 جدول)
- [x] Foreign Keys (50+ رابطه)
- [x] Unique Constraints (20+)
- [x] Composite Indexes (10+)
- [x] JSONB Indexes (8)
- [x] Partial Indexes (3+)
- [x] Covering Indexes (2+)

**کل شاخص‌ها: 45+**

---

**استراتژی شاخص‌گذاری برای تولید آماده است!**
