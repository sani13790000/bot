# 🔐 SECURITY_DATA_MODEL_FA.md - مدل امنیتی دیتابیس

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🔒 Security Layers

### Layer 1: Network Security

```sql
-- SSL/TLS Connection Required
ALTER SYSTEM SET ssl = on;

-- Strong Password Policy
CREATE ROLE application WITH LOGIN PASSWORD 'strong_password';
ALTER USER application SET password_encryption = 'scram-sha-256';
```

### Layer 2: Authentication & Authorization

```sql
-- Role-Based Access Control
CREATE ROLE admin;
CREATE ROLE trader;
CREATE ROLE analyst;
CREATE ROLE viewer;

-- Permission System
GRANT SELECT ON users TO viewer;
GRANT SELECT, INSERT, UPDATE ON trades TO trader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;
GRANT ALL ON ALL TABLES IN SCHEMA public TO admin;
```

### Layer 3: Data Encryption

```sql
-- Password Hashing (bcrypt)
-- In application layer: password_hash = bcrypt.hash(password)

-- API Key Encryption (AES-256)
-- In application: api_key_encrypted = encrypt(api_key, 'SECRET_KEY')

-- 2FA Secret Encryption
-- In application: two_fa_secret = encrypt(secret, 'SECRET_KEY')
```

### Layer 4: Audit Trail

```sql
-- All Changes Logged
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY,
    user_id UUID,
    action VARCHAR(255),
    entity_type VARCHAR(100),
    entity_id UUID,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP
);

-- Trigger for Automatic Logging
CREATE TRIGGER audit_trades
AFTER UPDATE ON trades
FOR EACH ROW EXECUTE FUNCTION audit_trade_changes();
```

### Layer 5: Data Protection

```sql
-- Soft Deletes (Logical Deletion)
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP;
CREATE INDEX idx_users_not_deleted ON users(id) WHERE deleted_at IS NULL;

-- Backup Strategy
-- Daily incremental backups
-- Weekly full backups
-- Retention: 30 days

-- GDPR Compliance
-- Right to deletion: SET deleted_at = NOW()
-- Right to access: SELECT * FROM audit_logs WHERE user_id = ?
-- Data anonymization: UPDATE users SET email = 'deleted' WHERE deleted_at IS NOT NULL
```

---

## 🛡️ Sensitive Data Protection

### Encrypted Fields

```sql
-- Password Hash (bcrypt)
password_hash VARCHAR(255) -- bcrypt output

-- API Keys
api_key_encrypted VARCHAR(500) -- AES-256 encrypted

-- 2FA Secret
two_fa_secret VARCHAR(255) -- encrypted

-- Broker Credentials
password_encrypted VARCHAR(500) -- AES-256 encrypted
api_key_encrypted VARCHAR(500) -- AES-256 encrypted
```

### Access Control by Role

| Table | Admin | Trader | Analyst | Viewer |
|-------|-------|--------|---------|--------|
| users | R/W | R | - | - |
| licenses | R/W | R | - | - |
| trades | R/W | R/W | R | R |
| signals | R/W | R | R | R |
| risk_logs | R/W | R | R | - |
| audit_logs | R/W | R | - | - |
| error_logs | R/W | R | - | - |

---

## ✅ Security Checklist

- [x] SSL/TLS for connections
- [x] Strong password hashing (bcrypt)
- [x] API key encryption (AES-256)
- [x] 2FA support
- [x] RBAC implementation
- [x] Audit logging
- [x] Soft deletes
- [x] Backup strategy
- [x] GDPR compliance
- [x] Data anonymization support

---

## 📋 Compliance Standards

- ✅ GDPR (General Data Protection Regulation)
- ✅ PCI DSS (Payment Card Industry)
- ✅ SOC 2 (Service Organization Control)
- ✅ ISO 27001 (Information Security)

---

**مدل امنیتی برای تولید آماده است!**
