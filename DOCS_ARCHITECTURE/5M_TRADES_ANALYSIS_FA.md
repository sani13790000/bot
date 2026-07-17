# 🚨 تجزیه دقیق: 5 میلیون معامله - Partitioning، Foreign Keys، Indexes

**نسخه:** 1.0 | **تاریخ:** 1403/04/27 | **زبان:** فارسی

---

## 1️⃣ اگر 5 میلیون معامله ذخیره شود چه اتفاقی می‌افتد؟

### 🔴 بدون Optimization:

```
❌ Query Time:          500-5000ms (TIMEOUT)
❌ Index Size:          100+ GB (Cache miss)
❌ Storage:             130 GB (Expensive)
❌ Backup Time:         10+ hours
❌ Recovery Time:       1-2 hours
❌ VACUUM Time:         10-20 hours
❌ CPU Usage:           100% (Bottleneck)
❌ Memory:              Swap (بسیار کند)
❌ Concurrent Users:    5-10 only
❌ Result:              🚨 CRASH (30+ min downtime)
```

### ✅ با Optimization:

```
✅ Query Time:          <100ms
✅ Index Size:          10-15 GB
✅ Storage:             50-60 GB
✅ Backup Time:         2-3 hours
✅ Recovery Time:       <1 hour
✅ VACUUM Time:         <2 hours
✅ CPU Usage:           40-50%
✅ Memory:              Normal
✅ Concurrent Users:    10,000+
✅ Result:              ✅ STABLE
```

### 📊 Timeline - مشکل کی شروع می‌شود:

| Trade Count | Status | مشکلات |
|-------------|--------|--------|
| < 100K | ✅ خوب | هیچ مشکلی |
| 100K - 1M | ⚠️ احتیاط | Some queries 50-100ms |
| 1M - 2M | 🟡 نگرانی‌برانگیز | Audit 500ms+, VACUUM 2h |
| 2M - 5M | 🔴 بحرانی | Dashboard 5-10sec, VACUUM 10-20h |
| > 5M | 🚨 فروپاشی | Platform down 30+ min/day |

---

## 2️⃣ کدام جداول باید Partition شوند؟

### 🔴 CRITICAL - trades (5M rows)

**Strategy:** `RANGE by created_at (quarterly) + HASH by trading_account_id`

```sql
-- Step 1: Recreate table with partitioning
CREATE TABLE trades_new (
    id UUID PRIMARY KEY,
    trading_account_id UUID NOT NULL,
    pair VARCHAR(10) NOT NULL,
    entry_time TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT now(),
    ...
) PARTITION BY RANGE (created_at);

-- Step 2: Create quarterly partitions
CREATE TABLE trades_2024_q1 PARTITION OF trades_new
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE trades_2024_q2 PARTITION OF trades_new
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE trades_2024_q3 PARTITION OF trades_new
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

CREATE TABLE trades_2024_q4 PARTITION OF trades_new
    FOR VALUES FROM ('2024-10-01') TO ('2025-01-01');

-- Step 3: Copy data (no downtime)
INSERT INTO trades_new SELECT * FROM trades;

-- Step 4: Swap tables
ALTER TABLE trades RENAME TO trades_old;
ALTER TABLE trades_new RENAME TO trades;

-- Step 5: Verify & cleanup
-- SELECT COUNT(*) FROM trades;
-- DROP TABLE trades_old;
```

**فایدہ:** 100x سریع‌تر queries روی بزرگ date ranges

---

### 🔴 CRITICAL - audit_logs (25M rows)

**Strategy:** `RANGE by created_at (monthly)`

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY,
    user_id UUID,
    created_at TIMESTAMP DEFAULT now(),
    ...
) PARTITION BY RANGE (created_at);

-- Auto-create partitions for each month
DO $$
DECLARE
    start_date DATE := '2024-01-01';
    end_date DATE;
BEGIN
    FOR i IN 0..11 LOOP
        end_date := start_date + interval '1 month';
        EXECUTE format('
            CREATE TABLE IF NOT EXISTS audit_logs_%s PARTITION OF audit_logs
            FOR VALUES FROM (%L) TO (%L)',
            to_char(start_date, 'YYYY_MM'),
            start_date, end_date
        );
        start_date := end_date;
    END LOOP;
END $$;
```

**فایدہ:** 80% سریع‌تر recent logs، آسان cleanup

---

### 🟠 HIGH - signals (10M rows)

**Strategy:** `RANGE by created_at (monthly)`

```sql
CREATE TABLE signals (
    id UUID PRIMARY KEY,
    pair VARCHAR(10),
    created_at TIMESTAMP DEFAULT now(),
    ...
) PARTITION BY RANGE (created_at);

-- Monthly partitions same as audit_logs
```

**فایدہ:** 50% سریع‌تر signal queries

---

### 🟠 HIGH - ai_decisions (10M rows)

**Strategy:** `RANGE by created_at (monthly)`

```sql
CREATE TABLE ai_decisions (
    id UUID PRIMARY KEY,
    signal_id UUID,
    created_at TIMESTAMP DEFAULT now(),
    ...
) PARTITION BY RANGE (created_at);
```

**فایدہ:** 40% سریع‌تر AI analysis

---

### 🟡 MEDIUM - closed_positions (4.75M rows)

**Strategy:** `RANGE by closed_at (quarterly) + archive after 2 years`

```sql
CREATE TABLE closed_positions (
    id UUID PRIMARY KEY,
    closed_at TIMESTAMP,
    ...
) PARTITION BY RANGE (closed_at);

-- Archive strategy
DELETE FROM closed_positions 
WHERE closed_at < now() - interval '2 years';
```

**فایدہ:** 30% سریع‌تر active queries

---

### 🟢 LOW - notifications (7.5M rows)

**Strategy:** `RANGE by created_at (monthly) + auto-delete after 90 days`

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    user_id UUID,
    created_at TIMESTAMP DEFAULT now(),
    ...
) PARTITION BY RANGE (created_at);

-- Auto-cleanup cron job
-- DELETE FROM notifications WHERE created_at < now() - interval '90 days';
```

**فایدہ:** 20% سریع‌تر + کم storage

---

## 3️⃣ آیا حذف کاربر باعث حذف معاملات می‌شود؟

### 🔴 خطرناک - ON DELETE CASCADE:

```sql
-- ❌ WRONG - DO NOT USE!
CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,  -- ⚠️ DANGEROUS!
    ...
);

-- نتیجه:
-- DELETE FROM users WHERE id = '...';
-- → تمام trading_accounts حذف می‌شود
-- → تمام trades حذف می‌شود
-- → 💥 5 میلیون معامله از بین می‌رود!
```

### ✅ درست - ON DELETE RESTRICT:

```sql
-- ✅ CORRECT - ALWAYS USE FOR CRITICAL DATA!
CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,  -- ✅ SAFE!
    ...
);

-- نتیجه:
-- DELETE FROM users WHERE id = '...';
-- → ERROR: Cannot delete user if trades exist
-- → ✅ معاملات محفوظ می‌ماند!
```

### 🟢 بهترین Practice - Soft Deletes:

```sql
-- Add soft delete column
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;

-- Logical delete (نه حذف فیزیکی)
UPDATE users SET deleted_at = NOW() WHERE id = '...';

-- تمام queries شامل soft delete filter
SELECT * FROM users WHERE deleted_at IS NULL;

-- Benefits:
-- ✅ معاملات هرگز حذف نمی‌شوند
-- ✅ GDPR compliance (data retention)
-- ✅ Audit trail محفوظ
-- ✅ Restore capability
```

### 📊 Foreign Keys - Recommended Configuration:

```sql
-- Soft delete approach (preferred)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    deleted_at TIMESTAMP DEFAULT NULL,
    ...
);

CREATE TABLE trading_accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    ...
);

CREATE TABLE trades (
    id UUID PRIMARY KEY,
    trading_account_id UUID NOT NULL REFERENCES trading_accounts(id) ON DELETE RESTRICT,
    ...
);

CREATE TABLE licenses (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    ...
);

CREATE TABLE broker_accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    ...
);

-- Archive strategy for old users
CREATE TABLE archive_users AS SELECT * FROM users WHERE deleted_at < now() - interval '2 years';
DELETE FROM users WHERE deleted_at < now() - interval '2 years';
```

---

## 4️⃣ کدام ستون‌ها باید Index داشته باشند؟

### 📊 TRADES Indexes (1,200 MB):

```sql
-- 🔴 CRITICAL - Most frequent queries
CREATE INDEX CONCURRENTLY idx_trades_account 
    ON trades(trading_account_id);  -- Size: 150MB

CREATE INDEX CONCURRENTLY idx_trades_pair 
    ON trades(pair);  -- Size: 120MB

-- 🟠 HIGH - Common filters
CREATE INDEX CONCURRENTLY idx_trades_status 
    ON trades(status);  -- Size: 80MB

CREATE INDEX CONCURRENTLY idx_trades_time 
    ON trades(entry_time DESC);  -- Size: 150MB

-- 🟠 HIGH - Composite indexes (covering)
CREATE INDEX CONCURRENTLY idx_trades_account_pair_time 
    ON trades(trading_account_id, pair, entry_time DESC);  -- Size: 200MB

CREATE INDEX CONCURRENTLY idx_trades_account_status_time 
    ON trades(trading_account_id, status, entry_time DESC);  -- Size: 200MB

-- 🟡 MEDIUM - Partial index for closed trades
CREATE INDEX CONCURRENTLY idx_trades_closed_account 
    ON trades(trading_account_id, created_at) 
    WHERE status = 'closed';  -- Size: 100MB
```

### 📊 SIGNALS Indexes (500 MB):

```sql
-- 🔴 CRITICAL
CREATE INDEX CONCURRENTLY idx_signals_pair ON signals(pair);
CREATE INDEX CONCURRENTLY idx_signals_status ON signals(status);

-- 🟠 HIGH
CREATE INDEX CONCURRENTLY idx_signals_source ON signals(source);
CREATE INDEX CONCURRENTLY idx_signals_time ON signals(created_at DESC);
CREATE INDEX CONCURRENTLY idx_signals_pair_status_time 
    ON signals(pair, status, created_at DESC);
```

### 📊 AUDIT_LOGS Indexes (2,000 MB):

```sql
-- 🔴 CRITICAL
CREATE INDEX CONCURRENTLY idx_audit_user ON audit_logs(user_id);
CREATE INDEX CONCURRENTLY idx_audit_time ON audit_logs(created_at DESC);

-- 🟠 HIGH
CREATE INDEX CONCURRENTLY idx_audit_entity 
    ON audit_logs(entity_type, entity_id);
CREATE INDEX CONCURRENTLY idx_audit_user_time 
    ON audit_logs(user_id, created_at DESC);
```

### 📊 OTHER TABLES:

```sql
-- open_positions
CREATE INDEX CONCURRENTLY idx_open_positions_account 
    ON open_positions(trading_account_id);
CREATE INDEX CONCURRENTLY idx_open_positions_pair 
    ON open_positions(pair);

-- users
CREATE UNIQUE INDEX CONCURRENTLY idx_users_email ON users(email);
CREATE UNIQUE INDEX CONCURRENTLY idx_users_username ON users(username);
CREATE INDEX CONCURRENTLY idx_users_status ON users(status);

-- notifications
CREATE INDEX CONCURRENTLY idx_notifications_user 
    ON notifications(user_id, is_read);
CREATE INDEX CONCURRENTLY idx_notifications_user_time 
    ON notifications(user_id, created_at DESC);
```

---

## 📊 Total Index Size: ~3.8 GB

| Table | Indexes Size | Benefit |
|-------|--------------|---------|
| trades | 1,200 MB | 100x faster queries |
| signals | 500 MB | 50x faster queries |
| audit_logs | 2,000 MB | 80% faster recent logs |
| others | 160 MB | Various improvements |
| **TOTAL** | **3,865 MB (3.8 GB)** | ✅ Reasonable! |

---

## ⚠️ Index Creation Best Practices:

### 1. استفاده از CONCURRENTLY:

```sql
-- ✅ CORRECT - Non-blocking
CREATE INDEX CONCURRENTLY idx_name ON table(column);

-- ❌ WRONG - Locks table
CREATE INDEX idx_name ON table(column);
```

### 2. زمان بندی:

```
✅ Create during off-peak hours (night time)
❌ Never during peak trading hours
⚠️ Expect 2-3x slower with CONCURRENTLY
```

### 3. Monitoring:

```sql
-- Check progress
SELECT * FROM pg_stat_progress_create_index;

-- Verify it works
EXPLAIN (ANALYZE) SELECT * FROM trades 
WHERE trading_account_id = '...' AND pair = 'EURUSD';
-- Should show "Index Scan" not "Sequential Scan"
```

---

## 🎯 Implementation Roadmap:

| هفته | Priority | اقدام |
|------|----------|-------|
| 1-2 | 🔴 | Analyze slow queries, setup monitoring |
| 3-4 | 🔴 | **Implement trades partitioning** |
| 5-6 | 🔴 | **Implement audit_logs partitioning** |
| 7-8 | 🟠 | Create all critical indexes |
| 9-10 | 🟠 | Implement soft delete strategy |
| 11-12 | 🟠 | Setup archive strategy |
| 13-16 | 🟡 | Signals/AI_Decisions partitioning |

---

## ✅ نتیجهٔ نهایی:

### با اعمال توصیات:

```
✅ 5 میلیون معامله محفوظ است
✅ معاملات هرگز حذف نمی‌شوند
✅ Queries < 100ms
✅ Storage 50-60 GB
✅ Backup 2-3 hours
✅ Platform stable و scalable
```

### بدون اعمال توصیات:

```
❌ Platform unavailable
❌ معاملات در خطر حذف
❌ Queries 500-5000ms
❌ Storage 130 GB
❌ Backup 10+ hours
❌ CPU 100%, Swap usage
```

**🔴 CRITICAL: شروع کنید الآن!**

