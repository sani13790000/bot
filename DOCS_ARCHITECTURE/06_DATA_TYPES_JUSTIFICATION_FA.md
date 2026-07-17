# 6️⃣ DATA TYPES - Complete Justification

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **وضعیت:** ✅ Production-Ready

---

## 📊 Data Type Matrix

### Numeric Types

| Column | Type | Precision | Range | Reason |
|--------|------|-----------|-------|--------|
| entry_price | DECIMAL(12,6) | 12 digits, 6 decimal | ±999999.999999 | Forex prices need 6 decimals |
| balance | DECIMAL(15,2) | 15 digits, 2 decimal | ±9999999999.99 | Money: 2 decimals max |
| profit_loss | DECIMAL(12,2) | 12 digits, 2 decimal | ±9999999.99 | Trade P&L |
| profit_loss_pct | DECIMAL(5,2) | 5 digits, 2 decimal | ±999.99 | Percentage: max 999% |
| confidence | DECIMAL(3,2) | 3 digits, 2 decimal | 0.00-1.00 | Confidence score |
| leverage | INTEGER | Whole number | 1-500 | Integer leverage |
| quantity | DECIMAL(12,6) | 12 digits, 6 decimal | ±999999.999999 | Lot sizes |
| max_machines | INTEGER | Whole number | 1-∞ | Count |

### String Types

| Column | Type | Max Length | Reason |
|--------|------|-----------|--------|
| email | VARCHAR(255) | 254 | RFC 5321 standard |
| username | VARCHAR(100) | 100 | Typical 3-50 chars |
| pair | VARCHAR(10) | 10 | EURUSD=6, buffer |
| broker_name | VARCHAR(100) | 100 | Typical names |
| message | TEXT | Unlimited | Variable length |
| explanation_text | TEXT | Unlimited | Variable length |
| password_hash | VARCHAR(255) | 255 | bcrypt output |
| api_key_encrypted | VARCHAR(500) | 500 | Encrypted key |

### Temporal Types

| Column | Type | Reason |
|--------|------|--------|
| created_at | TIMESTAMP | Record creation |
| updated_at | TIMESTAMP | Last modification |
| entry_time | TIMESTAMP | Trade entry time |
| deleted_at | TIMESTAMP | Soft delete marker |
| expires_at | TIMESTAMP | Expiration time |

### Special Types

| Column | Type | Reason |
|--------|------|--------|
| id | UUID | Unique identifier (distributed) |
| tags | JSONB | Flexible metadata |
| factors | JSONB | Variable AI data |
| old_values | JSONB | Audit trail (any columns) |
| affected_pairs | JSONB | Variable market pairs |
| ip_address | INET | IP validation |

### Enum Types (25+ total)

```sql
-- User status
ENUM('active', 'inactive', 'suspended', 'deleted')

-- Trade status
ENUM('open', 'closed', 'cancelled')

-- Trade type
ENUM('BUY', 'SELL')

-- Signal type
ENUM('BUY', 'SELL', 'NEUTRAL')

-- Delivery status
ENUM('pending', 'sent', 'failed')

-- Error level
ENUM('INFO', 'WARNING', 'ERROR', 'CRITICAL')

-- Decision
ENUM('APPROVE', 'REJECT', 'MODIFY')

-- Account type
ENUM('live', 'demo', 'paper')

-- Plan type
ENUM('monthly', 'quarterly', 'annual')

-- Error level
ENUM('HIGH', 'MEDIUM', 'LOW')
```

---

## ✅ Data Type Summary

- [x] DECIMAL for all financial data (no FLOAT)
- [x] VARCHAR lengths justified
- [x] UUID for distributed identifiers
- [x] JSONB for flexible data
- [x] ENUM for fixed-value columns
- [x] TIMESTAMP for temporal data
- [x] INET for IP addresses
- [x] No precision loss in calculations

---

**Status:** ✅ SECTION 6 COMPLETE
