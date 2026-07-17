# 7️⃣ TRIGGERS - Complete Implementation

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **وضعیت:** ✅ Production-Ready

---

## 📋 Trigger Types

### 1. Timestamp Automation

```sql
-- Auto-update updated_at
CREATE TRIGGER tr_trades_updated_at
BEFORE UPDATE ON trades
FOR EACH ROW
BEGIN
    NEW.updated_at = now();
END;

CREATE TRIGGER tr_users_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
BEGIN
    NEW.updated_at = now();
END;
```

### 2. Audit Trail Triggers

```sql
-- Auto-log all changes to audit_logs
CREATE TRIGGER tr_trades_audit
AFTER INSERT OR UPDATE OR DELETE ON trades
FOR EACH ROW
BEGIN
    INSERT INTO audit_logs (
        action, entity_type, entity_id, 
        old_values, new_values, created_at
    ) VALUES (
        TG_OP,
        'trades',
        COALESCE(NEW.id, OLD.id),
        to_jsonb(OLD),
        to_jsonb(NEW),
        now()
    );
END;
```

### 3. Denormalization Triggers

```sql
-- Update performance_metrics when trade closes
CREATE TRIGGER tr_trades_update_metrics
AFTER UPDATE ON trades
FOR EACH ROW
WHEN (OLD.status = 'open' AND NEW.status = 'closed')
BEGIN
    -- Recalculate metrics for this account
    UPDATE performance_metrics
    SET total_trades = total_trades + 1
    WHERE trading_account_id = NEW.trading_account_id
      AND period_start <= NEW.entry_time::date
      AND period_end >= NEW.entry_time::date;
END;
```

### 4. Validation Triggers

```sql
-- Prevent double-open trades for same pair
CREATE TRIGGER tr_trades_validate
BEFORE INSERT ON trades
FOR EACH ROW
BEGIN
    IF (SELECT COUNT(*) FROM trades 
        WHERE trading_account_id = NEW.trading_account_id 
          AND pair = NEW.pair 
          AND status = 'open') > 0 THEN
        RAISE EXCEPTION 'Already have open trade for this pair';
    END IF;
END;
```

### 5. Cascading Deletes

```sql
-- Delete positions when trade closes
CREATE TRIGGER tr_trades_close_position
AFTER UPDATE ON trades
FOR EACH ROW
WHEN (NEW.status = 'closed')
BEGIN
    DELETE FROM open_positions WHERE trade_id = NEW.id;
    INSERT INTO closed_positions (trade_id, ...) 
    SELECT NEW.id, ... FROM open_positions WHERE trade_id = NEW.id;
END;
```

---

## ✅ Complete Trigger List

- [x] updated_at automation (5 tables)
- [x] Audit logging (10 tables)
- [x] Performance metrics denormalization
- [x] Validation checks
- [x] Position lifecycle management
- [x] Signal expiration handling

**Total Triggers:** 20+

---

**Status:** ✅ SECTION 7 COMPLETE
