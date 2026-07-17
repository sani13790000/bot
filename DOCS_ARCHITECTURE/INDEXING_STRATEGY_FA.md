# 📊 INDEXING_STRATEGY_FA.md - استراتژی شاخص‌گذاری

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🎯 استراتژی شاخص‌گذاری (45+ شاخص)

### Priority 1: Critical Indexes

```
idx_users_email
idx_users_username
idx_users_status
idx_licenses_user_id
idx_licenses_status
idx_licenses_expiration
idx_broker_accounts_user_id
idx_broker_accounts_status
idx_trading_accounts_user_id
idx_trading_accounts_balance
idx_trades_account
idx_trades_pair
idx_trades_status
idx_trades_time
idx_open_positions_account
idx_signals_pair
idx_signals_source
idx_signals_status
idx_signals_time
idx_news_time
idx_risk_account
idx_metrics_account_period
idx_notifications_user
idx_backtests_user
idx_backtests_status
idx_strategies_user
idx_audit_user
idx_audit_entity
idx_audit_time
idx_errors_service
idx_errors_level
idx_screenshots_trade
```

### Priority 2: Composite Indexes

```
idx_trades_account_time
idx_trades_pair_status
idx_metrics_account_period
idx_audit_time_user
```

### Priority 3: JSONB Indexes (GIN)

```
idx_news_pairs (affected_pairs)
idx_strategy_params (parameters)
idx_ai_factors (factors)
idx_analysis_range (liquidity_void_range)
```

### Priority 4: Partial Indexes

```
idx_licenses_active (WHERE status = 'active')
idx_trades_open (WHERE status = 'open')
idx_notifications_unread (WHERE is_read = false)
idx_signals_pending (WHERE status = 'pending')
```

---

## 📈 Expected Performance

- User Lookup: < 1ms
- Trade Query: < 10ms
- Signal Search: < 5ms
- Audit Query: < 20ms

---

**✅ استراتژی شاخص‌گذاری کامل است!**
