# 🔐 مدل امنیتی داده - SECURITY_DATA_MODEL_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🔐 پنج لایهٔ امنیت

### 1️⃣ Network Security
```sql
-- SSL/TLS برای اتصالات
sslmode = require
```

### 2️⃣ Authentication
```sql
-- JWT Tokens
authorization HEADER 'Bearer eyJhbGc...'

-- Two-Factor Authentication
two_fa_enabled BOOLEAN
two_fa_secret VARCHAR(255) ENCRYPTED
```

### 3️⃣ Authorization (RBAC)
```sql
users → roles → permissions

-- مثال:
user.role = 'trader'
role.permissions = ['trading.buy', 'trading.sell', 'analysis.view']
```

### 4️⃣ Encryption
```sql
-- At Rest
password_hash VARCHAR(255)        -- bcrypt (irreversible)
api_key_encrypted VARCHAR(500)    -- AES-256
two_fa_secret VARCHAR(255)        -- encrypted

-- In Transit
SSL/TLS ┴ HTTPS مجبور
```

### 5️⃣ Audit & Compliance
```sql
audit_logs (تمام تغییرات)
error_logs (تمام خطاها)
deleted_at (soft delete برای GDPR)
```

---

## 🔑 استراتژی رمزگذاری

### Password Hash (bcrypt)
```python
import bcrypt

# ذخیره
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(12))

# تایید
bcrypt.checkpw(password.encode(), password_hash)
```

### API Keys (AES-256)
```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()  # مفتاح اصلی
cipher = Fernet(key)

# رمزگذاری
encrypted = cipher.encrypt(api_key.encode())

# رمزگشایی
decrypted = cipher.decrypt(encrypted)
```

### Two-Factor Secret
```python
import pyotp

secret = pyotp.random_base32()
totp = pyotp.TOTP(secret)

# تایید کد
is_valid = totp.verify(user_code)
```

---

## 👤 مدل RBAC

### Roles
```sql
admin       -- دسترسی کامل
trader      -- معاملات و تحلیل
analyst     -- تحلیل فقط
viewer      -- مشاهدهٔ فقط
```

### Permissions
```sql
-- Trading
trading.buy
trading.sell
trading.cancel

-- Analysis
analysis.view
analysis.export
analysis.backtest

-- Admin
admin.users
admin.settings
admin.audit
```

### Assignment
```sql
users 1-to-N user_roles N-to-1 roles 1-to-N role_permissions N-to-1 permissions
```

---

## 📝 Audit Trail

### audit_logs Schema
```sql
id             UUID              -- شناسایی یکتا
user_id        UUID              -- کاربری که اقدام کرد
action         VARCHAR(255)      -- عملی (CREATE, UPDATE, DELETE)
entity_type    VARCHAR(100)      -- جدول (trades, users, etc)
entity_id      UUID              -- رکورد مورد نظر
old_values     JSONB             -- مقدار قبل
new_values     JSONB             -- مقدار بعد
ip_address     INET              -- IP کاربر
user_agent     TEXT              -- مرورگر
created_at     TIMESTAMP         -- زمان
```

### Query مثال
```sql
SELECT *
FROM audit_logs
WHERE entity_type = 'users' AND entity_id = 'user-123'
ORDER BY created_at DESC;
```

---

## 🔒 Data Isolation

### Principle
```sql
-- هر کاربر فقط داده‌های خود را ببیند
WHERE user_id = current_user_id
```

### Implementation
```sql
-- View برای کاربران عادی
CREATE VIEW user_trades AS
SELECT *
FROM trades
WHERE trading_account_id IN (
    SELECT id FROM trading_accounts 
    WHERE user_id = current_user_id
);
```

---

## 🛡️ SQL Injection Prevention

### ✅ صحیح (Parameterized Queries)
```python
cursor.execute(
    "SELECT * FROM users WHERE email = %s AND status = %s",
    (email, 'active')
)
```

### ❌ نادرست (String Concatenation)
```python
cursor.execute(f"SELECT * FROM users WHERE email = '{email}'")
# Vulnerable!
```

---

## 🔄 Soft Deletes

### Implementation
```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

-- حذف نرم
UPDATE users SET deleted_at = now() WHERE id = 'user-123';

-- نمایش فقط فعال
SELECT * FROM users WHERE deleted_at IS NULL;

-- بازیابی
UPDATE users SET deleted_at = NULL WHERE id = 'user-123';
```

### مزایا
- بازیابی ممکن
- Audit Trail حفظ می‌شود
- Foreign Keys نشکسته می‌شود

---

## 📋 GDPR Compliance

### Data Subject Rights

#### 1. Right to Access
```sql
SELECT *
FROM users, trades, signals, audit_logs
WHERE user_id = 'user-123'
ORDER BY created_at;
```

#### 2. Right to Deletion
```sql
-- حذف نرم
UPDATE users SET deleted_at = now() 
WHERE id = 'user-123';

-- حذف تمام داده‌های متصل
UPDATE trades SET deleted_at = now()
WHERE trading_account_id IN (...);
```

#### 3. Right to Rectification
```sql
UPDATE users 
SET email = 'newemail@example.com'
WHERE id = 'user-123';
-- Logged in audit_logs
```

#### 4. Data Portability
```sql
-- Export به JSON
SELECT row_to_json(u) FROM users u WHERE id = 'user-123';
```

---

## 🔑 مدیریت کلید

### Master Key
```
هرگز در کد نباشد!
Environment Variable → سرور محفوظ
HSM (Hardware Security Module) → Production
```

### Encryption Keys
```
حداقل ۲۵۶-bit AES
خودکار rotation هر ۳ ماه
Backup شده در مکان محفوظ
```

---

## ✅ Security Checklist

- [x] SSL/TLS (Network)
- [x] Password Hashing (bcrypt)
- [x] API Key Encryption (AES-256)
- [x] Two-Factor Authentication
- [x] RBAC (Role-Based Access Control)
- [x] Audit Logging
- [x] Data Isolation (per user)
- [x] SQL Injection Prevention
- [x] Soft Deletes (GDPR)
- [x] Data Encryption at Rest
- [x] IP Logging & User Agent
- [x] Error Logging (secure)

---

**مدل امنیتی برای تولید آماده است!**
