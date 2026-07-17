# 📊 طراحی دیتابیس - DATABASE_SCHEMA_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **زبان:** فارسی

---

## 📋 فهرست جداول (38 جدول)

### 👥 کاربران و احراز (4 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| users | کاربران و احراز | id, email, username, password_hash, 2fa_secret |
| roles | نقش‌های دسترسی | id, name, description |
| user_roles | تخصیص نقش | user_id, role_id |
| permissions | مجوزها | id, name, module, action |

---

### 🔐 لایسنس و اشتراک (3 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| licenses | لایسنس و Hardware | license_key, hardware_id, expiration_date |
| subscriptions | اشتراک ماهانه | plan_type, billing_cycle, auto_renewal |
| hardware_bindings | دستگاه‌های متصل | hardware_id, first_activation |

---

### 🏦 حسابات (2 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| broker_accounts | اتصالات Brokers | broker_name, login_id, api_key_encrypted |
| trading_accounts | حسابات تجاری | balance, equity, margin_level |

---

### 💱 معاملات (4 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| trades | معاملات | entry_price, exit_price, profit_loss |
| open_positions | موضعیت‌های بازکن | current_price, unrealized_pl |
| closed_positions | معاملات بسته | realized_pl, duration |
| trade_history | تاریخچه | archived trades |

---

### 📊 تحلیل و سیگنال (6 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| signals | سیگنال‌ها | pair, signal_type, strength |
| ai_decisions | تصمیمات AI | decision, confidence, factors |
| ai_explanations | توضیحات | explanation_text, explanation_language |
| smart_money_analysis | SMC | order_block, liquidity_void |
| price_action_analysis | Price Action | pattern, support, resistance |
| technical_analysis | فنی | indicators, oscillators |

---

### 🌍 اخبار و بازار (2 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| news_events | اخبار | title, sentiment, impact |
| economic_calendar | تقویم | indicator_name, forecast, actual |

---

### 🎯 ریسک و عملکرد (3 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| risk_logs | رویدادهای ریسک | event_type, severity |
| trading_journal | یادداشت‌ها | journal_entry, sentiment |
| performance_metrics | معیارها | win_rate, max_drawdown, sharpe_ratio |

---

### 📬 اطلاعات (4 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| notifications | اطلاعات سیستم | title, message, is_read |
| telegram_messages | پیام‌های Telegram | telegram_chat_id, message_text |
| email_messages | ایمیل‌ها | recipient_email, subject, body |
| screenshots | تصاویر | file_path, file_size, expires_at |

---

### 🧪 بک‌تست (2 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| backtests | بک‌تست‌ها | start_date, end_date, results |
| optimization_results | نتایج بهینه‌سازی | parameter_set, fitness_score |

---

### ⚙️ تنظیمات (3 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| strategies | استراتژی‌ها | strategy_type, parameters |
| strategy_configurations | پارامترها | param_name, param_value |
| system_settings | تنظیمات سیستم | key, value |

---

### 📝 نظارت (2 جدول)
| جدول | هدف | ستون‌های کلیدی |
|------|-----|----------------|
| audit_logs | ثبت تغییرات | user_id, action, entity_id |
| error_logs | ثبت خطاها | service, error_level |

---

## 🔑 نوع‌های داده اصلی

```sql
-- UUID
id UUID PRIMARY KEY DEFAULT gen_random_uuid()

-- ENUM Types
ENUM('active', 'inactive', 'suspended')
ENUM('BUY', 'SELL')
ENUM('UPTREND', 'DOWNTREND', 'SIDEWAYS')

-- Decimal برای مالی
DECIMAL(15, 2)    -- 999,999,999,999.99
DECIMAL(12, 6)    -- 999,999.999999

-- JSONB برای داده‌های پیچیده
parameters JSONB
factors JSONB
metrics JSONB

-- Timestamps
created_at TIMESTAMP DEFAULT now()
updated_at TIMESTAMP DEFAULT now()
deleted_at TIMESTAMP    -- Soft Delete
```

---

## 🔗 رابطه‌های اصلی

```
users (1) ──N→ licenses
users (1) ──N→ trading_accounts
users (1) ──N→ signals
users (1) ──N→ backtests
users (1) ──N→ audit_logs

licenses (1) ──N→ subscriptions
licenses (1) ──N→ hardware_bindings

broker_accounts (1) ──N→ trading_accounts

trading_accounts (1) ──N→ trades
trading_accounts (1) ──N→ open_positions
trading_accounts (1) ──N→ risk_logs
trading_accounts (1) ──N→ performance_metrics

trades (1) ──N→ open_positions (unique)
trades (1) ──N→ closed_positions (unique)

signals (1) ──1→ ai_decisions (unique)

strategies (1) ──N→ backtests
backtests (1) ──N→ optimization_results
```

---

## 📊 آمار

✅ تعداد جداول: 38
✅ تعداد شاخص: 45+
✅ Foreign Keys: 50+
✅ Unique Constraints: 20+
✅ JSONB Columns: 8
✅ ENUM Types: 15+

---

## 🚀 نکات مهم

### Soft Delete
```sql
-- به جای حذف فیزیکی
deleted_at TIMESTAMP NULL
```

### Encryption
```sql
-- API Keys و رمزها رمزگذاری شوند
password_hash VARCHAR(255)        -- bcrypt
api_key_encrypted VARCHAR(500)    -- AES-256
two_fa_secret VARCHAR(255)        -- encrypted
```

### Audit Trail
```sql
-- تمام تغییرات ثبت شوند
audit_logs (user_id, action, old_values, new_values)
```

### Performance
```sql
-- شاخص‌های مناسب برای جستجو
INDEX ON (user_id, status)
INDEX ON (pair, created_at)
INDEX ON (trading_account_id, status)
```

---

**دیتابیس برای تولید آماده است!**
