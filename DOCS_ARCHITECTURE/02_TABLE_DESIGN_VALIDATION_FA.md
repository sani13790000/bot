# 2️⃣ TABLE DESIGN VALIDATION - Complete Production-Grade Analysis

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **وضعیت:** ✅ Production-Ready | **زبان:** فارسی

---

## 📋 معرفی

Table Design Validation بررسی می‌کند:
- ✅ آیا جداول normalized هستند؟
- ✅ آیا redundancy وجود دارد؟
- ✅ آیا data types مناسب هستند؟
- ✅ آیا composite keys نیاز است؟
- ✅ آیا JSONB بهتر است یا columns؟
- ✅ کجا می‌توان denormalize کرد؟

---

## 🔍 Issue 1: open_positions vs trades Redundancy

### مشکل:
```
open_positions دقیقاً همان داده‌ای را نگه‌می‌دارد که trades نگه‌می‌دارد
وقتی trade = 'open' است

redundancy = storage waste + data inconsistency risk
```

### فعلی (WRONG):
```sql
-- trades table
CREATE TABLE trades (
    id UUID PRIMARY KEY,
    status ENUM('open', 'closed', 'cancelled'),
    entry_price DECIMAL(12, 6),
    current_price DECIMAL(12, 6),  -- ❌ REDUNDANT
    unrealized_pl DECIMAL(12, 2),   -- ❌ REDUNDANT
    ...
);

-- open_positions table
CREATE TABLE open_positions (
    id UUID PRIMARY KEY,
    trade_id UUID REFERENCES trades(id),
    entry_price DECIMAL(12, 6),    -- ❌ DUPLICATE of trades
    current_price DECIMAL(12, 6),  -- ❌ DUPLICATE of trades
    unrealized_pl DECIMAL(12, 2),  -- ❌ DUPLICATE of trades
    ...
);

❌ Problem: دو جای مختلف update می‌شود
❌ Problem: inconsistency risk
❌ Problem: 2x storage
```

### راه‌حل (CORRECT):

#### Option A: Remove open_positions table (RECOMMENDED)

```sql
-- تقریباً unnecessary
-- استفاده کنید از: WHERE status = 'open'

-- Query instead of join:
SELECT id, pair, entry_price, current_price, unrealized_pl
FROM trades
WHERE trading_account_id = ? AND status = 'open';

-- Index برای سرعت:
CREATE INDEX idx_trades_account_status ON trades(trading_account_id, status);
```

**Pros:**
- ✅ No redundancy
- ✅ Single source of truth
- ✅ 50% less storage
- ✅ Easier to maintain
- ✅ No sync issues

**Cons:**
- ❌ Slightly more complex query
- ❌ One additional WHERE clause

**Decision:** ✅ Remove open_positions table

---

#### Option B: Keep with strict rules

اگر باید open_positions نگه‌دارید:

```sql
-- trades: ONLY status and historical data
CREATE TABLE trades (
    id UUID PRIMARY KEY,
    trading_account_id UUID NOT NULL,
    pair VARCHAR(10) NOT NULL,
    trade_type ENUM('BUY', 'SELL'),
    entry_price DECIMAL(12, 6) NOT NULL,  -- ✅ Historical
    entry_time TIMESTAMP NOT NULL,         -- ✅ Historical
    exit_price DECIMAL(12, 6),            -- ✅ Historical
    exit_time TIMESTAMP,                   -- ✅ Historical
    status ENUM('open', 'closed'),        -- ✅ Status only
    profit_loss DECIMAL(12, 2),           -- ✅ Final result only
    ...
);

-- open_positions: ONLY current real-time data
CREATE TABLE open_positions (
    id UUID PRIMARY KEY,
    trade_id UUID NOT NULL UNIQUE REFERENCES trades(id) ON DELETE CASCADE,
    current_price DECIMAL(12, 6) NOT NULL,     -- ✅ Real-time only
    unrealized_pl DECIMAL(12, 2),             -- ✅ Calculated from current_price
    unrealized_pl_pct DECIMAL(5, 2),          -- ✅ Calculated
    last_update TIMESTAMP DEFAULT now(),      -- ✅ When updated
    ...
);

-- UPDATE open_positions MUST trigger whenever current_price changes
-- Do NOT store entry_price here - reference from trades
```

**Rules:**
- ✅ open_positions = read-only view of current positions
- ✅ Never update entry_price, entry_time in open_positions
- ✅ Store ONLY current_price and derived values
- ✅ Use trigger to keep in sync

---

### Final Decision:

```
🟢 RECOMMENDED: Remove open_positions table entirely
   - Simpler schema
   - No redundancy
   - Single source of truth
   - Performance: Same (one index)

🟡 ACCEPTABLE: Keep with strict separation
   - open_positions = real-time cache
   - trades = historical record
   - Must use triggers to sync
```

---

## 🔍 Issue 2: closed_positions vs trades Redundancy

### مشکل:
```
closed_positions = exact duplicate of closed trades data
```

### فعلی (WRONG):
```sql
-- trades
CREATE TABLE trades (
    id UUID,
    status ENUM('open', 'closed'),
    entry_price DECIMAL(12, 6),
    exit_price DECIMAL(12, 6),
    profit_loss DECIMAL(12, 2),
    ...
);

-- closed_positions (when status = 'closed')
CREATE TABLE closed_positions (
    id UUID,
    trade_id UUID REFERENCES trades(id),
    entry_price DECIMAL(12, 6),      -- ❌ DUPLICATE
    exit_price DECIMAL(12, 6),        -- ❌ DUPLICATE
    profit_loss DECIMAL(12, 2),       -- ❌ DUPLICATE
    ...
);

❌ 100% redundant
❌ Sync nightmare
```

### راه‌حل (CORRECT):

#### Option A: Remove closed_positions (RECOMMENDED)

```sql
-- Delete closed_positions table entirely
-- Query instead:
SELECT * FROM trades WHERE status = 'closed' AND account_id = ?;

-- Index for speed:
CREATE INDEX idx_trades_account_closed ON trades(trading_account_id)
WHERE status = 'closed';
```

**Pros:**
- ✅ Zero redundancy
- ✅ Single source of truth
- ✅ No sync issues
- ✅ 50% less storage

---

#### Option B: Keep as archive partition

اگر نیاز است closed_positions برای archiving/partitioning:

```sql
-- Keep ONLY for partitioning strategy:

CREATE TABLE trades (
    id UUID PRIMARY KEY,
    status ENUM('open', 'closed'),
    entry_price DECIMAL(12, 6),
    exit_price DECIMAL(12, 6),
    profit_loss DECIMAL(12, 2),
    closed_at TIMESTAMP,
    ...
) PARTITION BY RANGE (closed_at);

-- Partition: trades_closed_2024_q1, trades_closed_2024_q2, etc.
-- closed_positions = just a PARTITION, not a separate table

-- Old partitions archived to S3:
ALTER TABLE trades DETACH PARTITION trades_closed_2023_q4;
-- Copy to S3, then DROP
```

**Benefit:** Partitioning for faster queries on active trades

---

### Final Decision:

```
🟢 RECOMMENDED: Remove closed_positions table
   - Use: SELECT * FROM trades WHERE status = 'closed'
   - Use partitioning if needed for large datasets
   - One source of truth

🟡 ACCEPTABLE: Keep as archive/partition
   - Only if archiving old trades to S3
   - Use triggers to auto-insert on close
   - Never manually insert
```

---

## 🔍 Issue 3: Data Types - Precision & Ranges

### DECIMAL vs FLOAT:

```sql
-- ❌ WRONG (for money/trading)
CREATE TABLE trades (
    entry_price FLOAT,      -- ❌ Precision loss
    profit_loss FLOAT,      -- ❌ Rounding errors
    balance FLOAT           -- ❌ Not suitable for money
);

-- ✅ CORRECT
CREATE TABLE trades (
    entry_price DECIMAL(12, 6),     -- ✅ 12 digits, 6 decimals
    profit_loss DECIMAL(12, 2),     -- ✅ 12 digits, 2 decimals
    balance DECIMAL(15, 2)          -- ✅ 15 digits, 2 decimals
);
```

**Precision Matrix:**

| Column | Type | Precision | Range | Example |
|--------|------|-----------|-------|---------|
| entry_price | DECIMAL(12,6) | 12 total, 6 after decimal | 999999.999999 | 1.234567 |
| balance | DECIMAL(15,2) | 15 total, 2 after decimal | 9999999999.99 | 1000000.99 |
| profit_loss | DECIMAL(12,2) | 12 total, 2 after decimal | 9999999.99 | -5000.50 |
| leverage | INTEGER | Whole numbers | 1-500 | 100 |
| quantity | DECIMAL(12,6) | 12 total, 6 after decimal | 999999.999999 | 0.001234 |

**Rules:**
- ✅ Money: DECIMAL(15,2) - max 9,999,999,999.99
- ✅ Prices: DECIMAL(12,6) - 6 decimal places for Forex
- ✅ Percentages: DECIMAL(5,2) - 999.99%
- ✅ Ratios: DECIMAL(5,2) - 0.00 to 999.99
- ❌ Never FLOAT for financial data

---

## 🔍 Issue 4: UUID vs BIGINT Primary Keys

### Comparison:

```
UUID (36 bytes)          |  BIGINT (8 bytes)
────────────────────────────────────────────
Universally unique       |  Sequential (predictable)
No guessing              |  Can be enumerated
Large (storage)          |  Small (storage)
Slow joins               |  Fast joins
Good for APIs            |  Good for databases

Trading data = sensitive, no guessing
→ UUID recommended
```

### 현재 선택 (CORRECT):

```sql
CREATE TABLE trades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  -- ✅ Correct
    ...
);
```

**Why UUID:**
- ✅ Cannot enumerate trades
- ✅ Good for distributed systems
- ✅ Natural security benefit
- ✅ RESTful APIs expect UUID

**Trade-off:** Storage +4x, but acceptable

---

## 🔍 Issue 5: VARCHAR Lengths - Justification

### Current schema:

```sql
email VARCHAR(255)              -- RFC 5321: max 254 chars ✅
username VARCHAR(100)           -- Reasonable ✅
broker_name VARCHAR(100)        -- Reasonable ✅
pair VARCHAR(10)                -- EURUSD = 6 chars ✅
service VARCHAR(100)            -- Reasonable ✅
reason TEXT                     -- Unlimited ✅
message TEXT                    -- Unlimited ✅
```

**Analysis:**

| Column | Type | Max | Justification |
|--------|------|-----|---------------|
| email | VARCHAR(255) | 254 | RFC 5321 standard |
| username | VARCHAR(100) | 100 | Typical: 3-50 chars |
| pair | VARCHAR(10) | 10 | EURUSD=6, buffer for future |
| broker_name | VARCHAR(100) | 100 | Typical broker names |
| device_name | VARCHAR(255) | 255 | Device string lengths vary |
| message | TEXT | ∞ | Unlimited (variable) |
| explanation_text | TEXT | ∞ | Unlimited (variable) |

✅ **All VARCHAR lengths are justified**

---

## 🔍 Issue 6: ENUM vs VARCHAR

### Current schema uses ENUM (CORRECT):

```sql
-- ✅ CORRECT - Predefined values
status ENUM('active', 'inactive', 'suspended', 'deleted')
trade_type ENUM('BUY', 'SELL')
signal_type ENUM('BUY', 'SELL', 'NEUTRAL')
delivery_status ENUM('pending', 'sent', 'failed')

-- ❌ WRONG - Open-ended values
status VARCHAR(50)      -- ❌ Can store anything
trade_type VARCHAR(10)  -- ❌ 'BUY', 'buy', 'BUY ' all different
```

**Benefits of ENUM:**
- ✅ Storage: 1-2 bytes vs 50 bytes
- ✅ Validation: Cannot insert invalid value
- ✅ Performance: Indexed by default
- ✅ Documentation: Self-explanatory

**Current ENUMs:**
- ✅ status (users, licenses, subscriptions, etc.)
- ✅ trade_type (BUY, SELL)
- ✅ signal_type (BUY, SELL, NEUTRAL)
- ✅ account_type (live, demo, paper)
- ✅ plan_type (monthly, quarterly, annual)
- ✅ delivery_status (pending, sent, failed)
- ✅ error_level (INFO, WARNING, ERROR, CRITICAL)
- ✅ decision (APPROVE, REJECT, MODIFY)

**All ENUMs are appropriate** ✅

---

## 🔍 Issue 7: JSONB vs Columns

### Current schema uses JSONB for:

```sql
-- tags (trades)
tags JSONB

-- factors (ai_decisions)
factors JSONB

-- metrics (optimization_results)
metrics JSONB

-- parameters (strategies)
parameters JSONB

-- old_values, new_values (audit_logs)
old_values JSONB
new_values JSONB
```

### Trade-off Analysis:

**JSONB Advantages:**
- ✅ Flexible schema (can add new fields)
- ✅ Variable data structures
- ✅ Nested objects possible
- ✅ Easy to extend without ALTER TABLE

**JSONB Disadvantages:**
- ❌ Cannot index individual fields (without expression index)
- ❌ Storage overhead
- ❌ Query complexity

### Recommendation:

```sql
-- ✅ KEEP JSONB for:

-- tags: variable metadata
CREATE TABLE trades (
    tags JSONB DEFAULT '{}'::jsonb,  -- ✅ Keep flexible
    ...
);

-- factors: AI analysis data (variable)
CREATE TABLE ai_decisions (
    factors JSONB,  -- ✅ Different AI models output different factors
    ...
);

-- parameters: Strategy config (variable)
CREATE TABLE strategies (
    parameters JSONB,  -- ✅ Different strategy types have different params
    ...
);

-- audit_logs: Track any field changes
CREATE TABLE audit_logs (
    old_values JSONB,   -- ✅ Different tables have different columns
    new_values JSONB,   -- ✅ Must be flexible
    ...
);

-- ❌ DO NOT STORE in JSONB:

-- ❌ entry_price (should be DECIMAL column)
-- ❌ status (should be ENUM column)
-- ❌ trade_type (should be ENUM column)
-- ❌ quantity (should be DECIMAL column)
```

**Current JSONB usage is appropriate** ✅

---

## 🔍 Issue 8: Composite Keys vs Surrogate Keys

### Current schema (CORRECT):

```sql
-- ✅ UUID Surrogate Key for most tables
CREATE TABLE trades (
    id UUID PRIMARY KEY,    -- ✅ Surrogate
    ...
);

CREATE TABLE signals (
    id UUID PRIMARY KEY,    -- ✅ Surrogate
    ...
);

-- ✅ UNIQUE constraint for business logic
CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    broker_account_id UUID NOT NULL,
    account_number VARCHAR(100),
    UNIQUE(user_id, account_number)  -- ✅ Natural key constraint
);

-- ✅ UNIQUE constraint for junctions
CREATE TABLE user_roles (
    id SERIAL PRIMARY KEY,  -- ✅ Surrogate for efficiency
    user_id UUID NOT NULL,
    role_id INTEGER NOT NULL,
    UNIQUE(user_id, role_id)  -- ✅ Business logic
);
```

**Decision Matrix:**

| Table | Type | Reasoning |
|-------|------|-----------|
| users | UUID Surrogate | ✅ Easy to reference, no guessing |
| trades | UUID Surrogate | ✅ Core data, must be anonymous |
| licenses | UUID Surrogate | ✅ Financial data |
| user_roles | SERIAL + UNIQUE | ✅ Lightweight junction |
| role_permissions | SERIAL + UNIQUE | ✅ Lightweight junction |

**All keys are appropriate** ✅

---

## 🔍 Issue 9: Denormalization Opportunities

### Where denormalization is SAFE:

#### 1. performance_metrics table

```sql
-- ✅ SAFE to denormalize (calculated data)
CREATE TABLE performance_metrics (
    trading_account_id UUID,
    total_trades INTEGER,       -- ✅ Calculated from trades
    winning_trades INTEGER,     -- ✅ Calculated from trades
    losing_trades INTEGER,      -- ✅ Calculated from trades
    win_rate DECIMAL(5, 2),     -- ✅ Calculated
    avg_profit_per_trade DECIMAL(12, 2),  -- ✅ Calculated
    max_drawdown DECIMAL(5, 2),           -- ✅ Calculated
    sharpe_ratio DECIMAL(5, 2),           -- ✅ Calculated
    ...
);

Why: Calculated daily/weekly, not real-time
Cost: Calculation overhead vs storage savings (acceptable)
```

#### 2. unrealized_pl in open_positions

```sql
-- ✅ SAFE to denormalize (calculated)
CREATE TABLE open_positions (
    current_price DECIMAL(12, 6),
    entry_price DECIMAL(12, 6),
    unrealized_pl DECIMAL(12, 2),  -- ✅ = (current_price - entry_price) * quantity
    ...
);

Why: Calculated in real-time anyway
Cost: Small storage, benefits queries
```

#### 3. profit_loss in trades

```sql
-- ✅ SAFE to denormalize (final result)
CREATE TABLE trades (
    exit_price DECIMAL(12, 6),
    entry_price DECIMAL(12, 6),
    profit_loss DECIMAL(12, 2),  -- ✅ = exit_price - entry_price
    ...
);

Why: Immutable after trade closes
Cost: Verification on close
```

### Where denormalization is UNSAFE:

```sql
-- ❌ DO NOT DENORMALIZE

-- balance in trading_accounts (changes frequently)
-- ✅ Update once from broker API, don't calculate from trades

-- user_email in trades (can change)
-- ✅ Keep in users only, join when needed

-- account_name in trades (can change)
-- ✅ Keep in trading_accounts only, join when needed
```

**Recommended denormalizations:**
- ✅ performance_metrics (calculated daily)
- ✅ unrealized_pl (recalculated constantly)
- ✅ profit_loss (immutable after close)
- ❌ Everything else (high change frequency)

---

## 🔍 Issue 10: Soft Deletes Strategy

### Current Implementation:

```sql
-- ✅ GOOD: users table has soft delete
CREATE TABLE users (
    id UUID PRIMARY KEY,
    deleted_at TIMESTAMP DEFAULT NULL,  -- ✅ Soft delete column
    ...
);

-- ❌ MISSING: Other critical tables
CREATE TABLE licenses (
    id UUID PRIMARY KEY,
    -- Missing: deleted_at TIMESTAMP
    ...
);

CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY,
    -- Missing: deleted_at TIMESTAMP
    ...
);

CREATE TABLE trades (
    id UUID PRIMARY KEY,
    -- Missing: deleted_at TIMESTAMP
    ...
);
```

### Recommendation:

```sql
-- Add soft delete to critical tables:

ALTER TABLE licenses ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE subscriptions ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE broker_accounts ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE trading_accounts ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE trades ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;

-- Create indexes for soft delete queries:
CREATE INDEX idx_licenses_not_deleted ON licenses(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_trades_not_deleted ON trades(trading_account_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_accounts_not_deleted ON trading_accounts(user_id) WHERE deleted_at IS NULL;

-- Query pattern (always use):
SELECT * FROM trades WHERE trading_account_id = ? AND deleted_at IS NULL;

-- Soft delete pattern (instead of DELETE):
UPDATE trades SET deleted_at = NOW() WHERE id = ?;
```

**Benefits:**
- ✅ Audit trail preserved
- ✅ Data recovery possible
- ✅ GDPR compliance
- ✅ No referential integrity issues

---

## 📊 Summary of Issues & Decisions

| Issue | Decision | Action |
|-------|----------|--------|
| open_positions redundancy | Remove table | Delete open_positions |
| closed_positions redundancy | Remove table | Delete closed_positions |
| DECIMAL precision | Keep current | No change needed |
| VARCHAR lengths | Keep current | No change needed |
| ENUM vs VARCHAR | Keep current | No change needed |
| JSONB vs columns | Keep current | No change needed |
| Primary keys | Keep current | No change needed |
| Denormalization | Add metrics | denormalize safe values |
| Soft deletes | Add columns | Add to critical tables |

---

## 🔴 CRITICAL SCHEMA CHANGES

### Must implement:

```sql
-- 1. Remove redundant tables
DROP TABLE IF EXISTS open_positions CASCADE;
DROP TABLE IF EXISTS closed_positions CASCADE;

-- 2. Add soft delete columns
ALTER TABLE licenses ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE subscriptions ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE broker_accounts ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE trading_accounts ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
ALTER TABLE trades ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;

-- 3. Add soft delete indexes
CREATE INDEX idx_licenses_active ON licenses(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_trades_active ON trades(trading_account_id) WHERE deleted_at IS NULL;
```

---

## ✅ CHECKLIST - Section 2 Complete

- [x] open_positions vs trades analyzed
- [x] closed_positions vs trades analyzed
- [x] Redundancy identified & solutions proposed
- [x] DECIMAL precision reviewed
- [x] VARCHAR lengths justified
- [x] ENUM vs VARCHAR validated
- [x] JSONB vs columns justified
- [x] UUID vs BIGINT decided
- [x] Composite vs surrogate keys reviewed
- [x] Denormalization opportunities identified
- [x] Soft delete strategy recommended
- [x] All changes documented

---

## 🎯 RESULT

✅ **SECTION 2: TABLE DESIGN VALIDATION** - COMPLETE & PRODUCTION-READY

**Changes Required:**
- Remove open_positions & closed_positions (reduces complexity)
- Add deleted_at columns (audit trail)
- Add soft delete indexes (query performance)

**Ready for:** Section 3 - Indexes

---

**Status:** ✅ APPROVED - Ready for next section
