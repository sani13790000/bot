# 📊 INDEXING_STRATEGY_FA.md - استراتژی شاخص‌گذاری

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🎯 استراتژی شاخص‌گذاری

### Priority 1: Critical Performance Indexes

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

-- Positions
CREATE INDEX idx_open_positions_account ON open_positions(trading_account_id);

-- Audit
CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_errors_service ON error_logs(service, created_at);
CREATE INDEX idx_errors_level ON error_logs(error_level);

-- Notifications
CREATE INDEX idx_notifications_user ON notifications(user_id, is_read);
```

### Priority 2: Composite Indexes

```sql
-- Range queries
CREATE INDEX idx_trades_account_time ON trades(trading_account_id, entry_time);
CREATE INDEX idx_trades_pair_status ON trades(pair, status);

-- Performance metrics
CREATE INDEX idx_metrics_account_period ON performance_metrics(trading_account_id, period_start, period_end);

-- Broker accounts
CREATE INDEX idx_broker_user_status ON broker_accounts(user_id, status);
CREATE INDEX idx_trading_account_user_status ON trading_accounts(user_id, status);
```

### Priority 3: JSONB Indexes (GIN)

```sql
-- Market data
CREATE INDEX idx_news_pairs_gin ON news_events USING GIN(affected_pairs);

-- Strategy parameters
CREATE INDEX idx_strategy_params_gin ON strategies USING GIN(parameters);

-- Optimization results
CREATE INDEX idx_optimization_metrics_gin ON optimization_results USING GIN(metrics);

-- AI factors
CREATE INDEX idx_ai_factors_gin ON ai_decisions USING GIN(factors);
CREATE INDEX idx_analysis_range_gin ON smart_money_analysis USING GIN(liquidity_void_range);
```

### Priority 4: Partial Indexes (للأداء العالي)

```sql
-- Only active licenses
CREATE INDEX idx_licenses_active ON licenses(user_id) WHERE status = 'active';

-- Only open positions
CREATE INDEX idx_trades_open ON trades(trading_account_id) WHERE status = 'open';

-- Only unread notifications
CREATE INDEX idx_notifications_unread ON notifications(user_id) WHERE is_read = false;

-- Only pending signals
CREATE INDEX idx_signals_pending ON signals(pair) WHERE status = 'pending';
```

---

## 📊 Index Maintenance

### Vacuum & Analyze Schedule
```sql
-- Daily
VACUUM ANALYZE trades;
VACUUM ANALYZE signals;
VACUUM ANALYZE trades;

-- Weekly
VACUUM ANALYZE;

-- Monthly
REINDEX DATABASE galaxy_bot;
```

### Monitor Index Usage
```sql
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

---

## 🔧 Index Tuning Parameters

```sql
-- Work memory for sorting
SET work_mem = '256MB';

-- Max parallel workers
SET max_parallel_workers = 4;

-- Random page cost
SET random_page_cost = 1.1;
```

---

## 📈 Expected Performance

- **User Lookup:** < 1ms
- **Trade Query:** < 10ms
- **Signal Search:** < 5ms
- **Audit Query:** < 20ms
- **Backtest Data:** < 50ms

---

**شاخص‌گذاری برای تولید آماده است!**
