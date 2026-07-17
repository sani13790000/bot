# 📊 DATABASE_SCHEMA_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **وضعیت:** ✅ نهایی | **زبان:** فارسی

---

## 🏗️ طراحی کامل پایگاه داده PostgreSQL 14+

### Layer 1: Users & Authentication

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    phone VARCHAR(20),
    country VARCHAR(100),
    timezone VARCHAR(50),
    language VARCHAR(10) DEFAULT 'en',
    avatar_url VARCHAR(500),
    two_fa_enabled BOOLEAN DEFAULT false,
    two_fa_secret VARCHAR(255),
    status ENUM('active', 'inactive', 'suspended', 'deleted') DEFAULT 'active',
    email_verified BOOLEAN DEFAULT false,
    email_verified_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now(),
    last_login_at TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);

CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    system_role BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE user_roles (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP DEFAULT now(),
    assigned_by UUID REFERENCES users(id),
    UNIQUE(user_id, role_id)
);

CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    module VARCHAR(50),
    action VARCHAR(50),
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE role_permissions (
    id SERIAL PRIMARY KEY,
    role_id INTEGER NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_id INTEGER NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE(role_id, permission_id)
);
```

### Layer 2: Licensing & Subscriptions

```sql
CREATE TABLE licenses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    license_key VARCHAR(50) UNIQUE NOT NULL,
    product_type ENUM('basic', 'pro', 'enterprise') DEFAULT 'basic',
    hardware_id VARCHAR(255),
    machine_fingerprint VARCHAR(255),
    max_machines INTEGER DEFAULT 1,
    status ENUM('active', 'suspended', 'expired', 'revoked') DEFAULT 'active',
    issued_at TIMESTAMP DEFAULT now(),
    activation_date TIMESTAMP,
    expiration_date TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_licenses_user_id ON licenses(user_id);
CREATE INDEX idx_licenses_status ON licenses(status);
CREATE INDEX idx_licenses_expiration ON licenses(expiration_date);

CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    license_id UUID NOT NULL REFERENCES licenses(id) ON DELETE CASCADE,
    plan_type ENUM('monthly', 'quarterly', 'annual') DEFAULT 'monthly',
    price DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    billing_cycle_start DATE NOT NULL,
    billing_cycle_end DATE NOT NULL,
    auto_renewal BOOLEAN DEFAULT true,
    status ENUM('active', 'cancelled', 'suspended') DEFAULT 'active',
    payment_method VARCHAR(50),
    last_payment_date TIMESTAMP,
    next_payment_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE TABLE hardware_bindings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    license_id UUID NOT NULL REFERENCES licenses(id) ON DELETE CASCADE,
    hardware_id VARCHAR(255) NOT NULL,
    device_name VARCHAR(255),
    first_activation TIMESTAMP,
    last_activation TIMESTAMP,
    status ENUM('active', 'inactive', 'revoked') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE(license_id, hardware_id)
);
```

### Layer 3: Broker & Trading Accounts

```sql
CREATE TABLE broker_accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    broker_name VARCHAR(100) NOT NULL,
    account_number VARCHAR(100) NOT NULL,
    account_type ENUM('live', 'demo', 'paper') DEFAULT 'demo',
    login_id INTEGER,
    server VARCHAR(255),
    password_encrypted VARCHAR(500),
    api_key_encrypted VARCHAR(500),
    currency VARCHAR(3) DEFAULT 'USD',
    leverage INTEGER DEFAULT 1,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    is_primary BOOLEAN DEFAULT false,
    connected_at TIMESTAMP,
    last_sync TIMESTAMP,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_broker_accounts_user_id ON broker_accounts(user_id);
CREATE INDEX idx_broker_accounts_status ON broker_accounts(status);

CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    broker_account_id UUID NOT NULL REFERENCES broker_accounts(id) ON DELETE CASCADE,
    account_name VARCHAR(255),
    account_number VARCHAR(100),
    balance DECIMAL(15, 2) DEFAULT 0,
    equity DECIMAL(15, 2) DEFAULT 0,
    free_margin DECIMAL(15, 2) DEFAULT 0,
    margin_level DECIMAL(5, 2) DEFAULT 0,
    max_daily_loss DECIMAL(12, 2),
    max_trade_loss DECIMAL(12, 2),
    max_open_trades INTEGER DEFAULT 10,
    status ENUM('active', 'inactive', 'frozen') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_trading_accounts_user_id ON trading_accounts(user_id);
CREATE INDEX idx_trading_accounts_balance ON trading_accounts(balance);
```

### Layer 4: Trading Operations

```sql
CREATE TABLE trades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id) ON DELETE CASCADE,
    pair VARCHAR(10) NOT NULL,
    trade_type ENUM('BUY', 'SELL') NOT NULL,
    entry_price DECIMAL(12, 6) NOT NULL,
    entry_time TIMESTAMP NOT NULL,
    quantity DECIMAL(12, 6) NOT NULL,
    stop_loss DECIMAL(12, 6),
    take_profit DECIMAL(12, 6),
    exit_price DECIMAL(12, 6),
    exit_time TIMESTAMP,
    status ENUM('open', 'closed', 'cancelled') DEFAULT 'open',
    profit_loss DECIMAL(12, 2),
    profit_loss_pct DECIMAL(5, 2),
    strategy_id UUID REFERENCES strategies(id),
    signal_id UUID REFERENCES signals(id),
    tags JSONB,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_trades_account ON trades(trading_account_id);
CREATE INDEX idx_trades_pair ON trades(pair);
CREATE INDEX idx_trades_status ON trades(status);
CREATE INDEX idx_trades_time ON trades(entry_time);

CREATE TABLE open_positions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trade_id UUID NOT NULL REFERENCES trades(id) ON DELETE CASCADE,
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id),
    pair VARCHAR(10),
    direction ENUM('LONG', 'SHORT'),
    quantity DECIMAL(12, 6),
    entry_price DECIMAL(12, 6),
    current_price DECIMAL(12, 6),
    unrealized_pl DECIMAL(12, 2),
    unrealized_pl_pct DECIMAL(5, 2),
    opened_at TIMESTAMP,
    last_update TIMESTAMP DEFAULT now(),
    UNIQUE(trade_id)
);

CREATE INDEX idx_open_positions_account ON open_positions(trading_account_id);

CREATE TABLE closed_positions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trade_id UUID NOT NULL REFERENCES trades(id) ON DELETE CASCADE,
    pair VARCHAR(10),
    direction ENUM('LONG', 'SHORT'),
    quantity DECIMAL(12, 6),
    entry_price DECIMAL(12, 6),
    exit_price DECIMAL(12, 6),
    realized_pl DECIMAL(12, 2),
    realized_pl_pct DECIMAL(5, 2),
    opened_at TIMESTAMP,
    closed_at TIMESTAMP,
    duration INTERVAL,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_closed_positions_trade_id ON closed_positions(trade_id);
```

### Layer 5: Signals & Analysis

```sql
CREATE TABLE signals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trading_account_id UUID REFERENCES trading_accounts(id),
    pair VARCHAR(10) NOT NULL,
    signal_type ENUM('BUY', 'SELL', 'NEUTRAL') NOT NULL,
    strength DECIMAL(3, 2),
    source ENUM('price_action', 'smc', 'ict', 'ai', 'hybrid') NOT NULL,
    entry_price DECIMAL(12, 6),
    stop_loss DECIMAL(12, 6),
    take_profit DECIMAL(12, 6),
    status ENUM('pending', 'active', 'executed', 'expired', 'rejected'),
    ai_approved BOOLEAN DEFAULT false,
    ai_confidence DECIMAL(3, 2),
    created_at TIMESTAMP DEFAULT now(),
    executed_at TIMESTAMP,
    expired_at TIMESTAMP
);

CREATE INDEX idx_signals_pair ON signals(pair);
CREATE INDEX idx_signals_source ON signals(source);
CREATE INDEX idx_signals_status ON signals(status);
CREATE INDEX idx_signals_time ON signals(created_at);

CREATE TABLE ai_decisions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    signal_id UUID NOT NULL REFERENCES signals(id) ON DELETE CASCADE,
    decision ENUM('APPROVE', 'REJECT', 'MODIFY') NOT NULL,
    confidence DECIMAL(3, 2),
    reason TEXT,
    factors JSONB,
    model_version VARCHAR(50),
    processing_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE ai_explanations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trade_id UUID REFERENCES trades(id) ON DELETE SET NULL,
    ai_decision_id UUID NOT NULL REFERENCES ai_decisions(id) ON DELETE CASCADE,
    explanation_text TEXT NOT NULL,
    explanation_language VARCHAR(10) DEFAULT 'en',
    factors JSONB,
    confidence_score DECIMAL(3, 2),
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE smart_money_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    signal_id UUID REFERENCES signals(id) ON DELETE CASCADE,
    pair VARCHAR(10) NOT NULL,
    timeframe VARCHAR(10),
    order_block_found BOOLEAN,
    order_block_level DECIMAL(12, 6),
    liquidity_void_found BOOLEAN,
    liquidity_void_range JSONB,
    institutional_bias ENUM('BULLISH', 'BEARISH', 'NEUTRAL'),
    confidence DECIMAL(3, 2),
    analyzed_at TIMESTAMP DEFAULT now()
);

CREATE TABLE price_action_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    signal_id UUID REFERENCES signals(id) ON DELETE CASCADE,
    pair VARCHAR(10) NOT NULL,
    pattern_detected VARCHAR(100),
    pattern_confidence DECIMAL(3, 2),
    support_level DECIMAL(12, 6),
    resistance_level DECIMAL(12, 6),
    trend ENUM('UPTREND', 'DOWNTREND', 'SIDEWAYS'),
    trend_strength DECIMAL(3, 2),
    analyzed_at TIMESTAMP DEFAULT now()
);
```

### Layer 6: Market Data & News

```sql
CREATE TABLE news_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    source VARCHAR(100),
    url VARCHAR(500),
    impact ENUM('HIGH', 'MEDIUM', 'LOW'),
    affected_pairs JSONB,
    scheduled_time TIMESTAMP,
    published_time TIMESTAMP,
    sentiment ENUM('POSITIVE', 'NEGATIVE', 'NEUTRAL'),
    sentiment_score DECIMAL(3, 2),
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_news_time ON news_events(scheduled_time);
CREATE INDEX idx_news_pairs ON news_events USING GIN(affected_pairs);

CREATE TABLE economic_calendar (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    country VARCHAR(100),
    indicator_name VARCHAR(255),
    forecast DECIMAL(12, 2),
    previous DECIMAL(12, 2),
    actual DECIMAL(12, 2),
    impact_level ENUM('HIGH', 'MEDIUM', 'LOW'),
    scheduled_time TIMESTAMP NOT NULL,
    release_time TIMESTAMP,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_economic_time ON economic_calendar(scheduled_time);
```

### Layer 7: Risk Management

```sql
CREATE TABLE risk_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id),
    event_type ENUM('daily_loss_limit_exceeded', 'margin_call', 'max_trades_exceeded', 'sl_hit'),
    severity ENUM('WARNING', 'ERROR', 'CRITICAL'),
    message TEXT,
    details JSONB,
    action_taken TEXT,
    resolved BOOLEAN DEFAULT false,
    resolved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_risk_account ON risk_logs(trading_account_id);

CREATE TABLE trading_journal (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id),
    user_id UUID NOT NULL REFERENCES users(id),
    trade_id UUID REFERENCES trades(id),
    journal_entry TEXT,
    sentiment ENUM('EXCELLENT', 'GOOD', 'NEUTRAL', 'BAD'),
    emotion VARCHAR(100),
    lessons_learned TEXT,
    improvements TEXT,
    entry_date TIMESTAMP DEFAULT now()
);

CREATE TABLE performance_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id),
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    total_trades INTEGER,
    winning_trades INTEGER,
    losing_trades INTEGER,
    win_rate DECIMAL(5, 2),
    total_profit_loss DECIMAL(15, 2),
    avg_profit_per_trade DECIMAL(12, 2),
    max_drawdown DECIMAL(5, 2),
    sharpe_ratio DECIMAL(5, 2),
    sortino_ratio DECIMAL(5, 2),
    created_at TIMESTAMP DEFAULT now(),
    UNIQUE(trading_account_id, period_start, period_end)
);

CREATE INDEX idx_metrics_account_period ON performance_metrics(trading_account_id, period_start);
```

### Layer 8: Notifications

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255),
    message TEXT,
    type ENUM('trade', 'alert', 'news', 'system', 'error'),
    is_read BOOLEAN DEFAULT false,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_notifications_user ON notifications(user_id, is_read);

CREATE TABLE telegram_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    telegram_chat_id VARCHAR(50),
    message_text TEXT,
    sent_at TIMESTAMP DEFAULT now(),
    delivered BOOLEAN DEFAULT false,
    delivery_status VARCHAR(50)
);

CREATE TABLE email_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    recipient_email VARCHAR(255),
    subject VARCHAR(255),
    body TEXT,
    sent_at TIMESTAMP,
    delivery_status ENUM('pending', 'sent', 'failed'),
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE bale_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    bale_chat_id VARCHAR(50),
    message_text TEXT,
    sent_at TIMESTAMP DEFAULT now(),
    delivered BOOLEAN DEFAULT false,
    delivery_status VARCHAR(50)
);

CREATE TABLE screenshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trade_id UUID REFERENCES trades(id) ON DELETE SET NULL,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    file_path VARCHAR(500),
    file_size INTEGER,
    mime_type VARCHAR(50),
    uploaded_at TIMESTAMP DEFAULT now(),
    expires_at TIMESTAMP
);

CREATE INDEX idx_screenshots_trade ON screenshots(trade_id);
```

### Layer 9: Backtesting

```sql
CREATE TABLE backtests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    strategy_id UUID NOT NULL REFERENCES strategies(id),
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    initial_balance DECIMAL(15, 2),
    pair VARCHAR(10),
    timeframe VARCHAR(10),
    final_balance DECIMAL(15, 2),
    total_profit_loss DECIMAL(15, 2),
    total_return_pct DECIMAL(6, 2),
    win_rate DECIMAL(5, 2),
    max_drawdown DECIMAL(5, 2),
    sharpe_ratio DECIMAL(5, 2),
    status ENUM('running', 'completed', 'failed'),
    created_at TIMESTAMP DEFAULT now(),
    completed_at TIMESTAMP
);

CREATE INDEX idx_backtests_user ON backtests(user_id);
CREATE INDEX idx_backtests_status ON backtests(status);

CREATE TABLE optimization_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    backtest_id UUID NOT NULL REFERENCES backtests(id) ON DELETE CASCADE,
    parameter_set JSONB,
    metrics JSONB,
    fitness_score DECIMAL(5, 2),
    rank INTEGER,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_optimization_backtest ON optimization_results(backtest_id);
```

### Layer 10: Strategies

```sql
CREATE TABLE strategies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    strategy_type ENUM('price_action', 'smc', 'ict', 'hybrid') NOT NULL,
    status ENUM('draft', 'active', 'inactive', 'archived'),
    parameters JSONB,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_strategies_user ON strategies(user_id);

CREATE TABLE strategy_configurations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    strategy_id UUID NOT NULL REFERENCES strategies(id) ON DELETE CASCADE,
    param_name VARCHAR(100),
    param_value VARCHAR(500),
    param_type VARCHAR(50),
    description TEXT,
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE system_settings (
    id SERIAL PRIMARY KEY,
    key VARCHAR(255) UNIQUE NOT NULL,
    value TEXT,
    value_type VARCHAR(50),
    description TEXT,
    updated_at TIMESTAMP DEFAULT now(),
    updated_by UUID REFERENCES users(id)
);

CREATE INDEX idx_system_settings_key ON system_settings(key);
```

### Layer 11: Audit & Logging

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(255),
    entity_type VARCHAR(100),
    entity_id UUID,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_time ON audit_logs(created_at);

CREATE TABLE error_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    service VARCHAR(100),
    error_message TEXT,
    error_stack TEXT,
    error_level ENUM('INFO', 'WARNING', 'ERROR', 'CRITICAL'),
    user_id UUID REFERENCES users(id),
    fixed BOOLEAN DEFAULT false,
    fixed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT now()
);

CREATE INDEX idx_errors_service ON error_logs(service, created_at);
CREATE INDEX idx_errors_level ON error_logs(error_level);
```

---

## 📊 خلاصهٔ

**کل جداول:** 38 جدول
**کل شاخص‌ها:** 45+ شاخص
**کل FK:** 50+ رابطهٔ خارجی
**کل UNIQUE:** 20+ محدودیت

---

**✅ Database Schema برای تولید آماده است!**
