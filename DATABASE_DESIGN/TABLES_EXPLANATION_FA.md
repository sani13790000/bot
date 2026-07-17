# 📋 توضیح دقیق جداول - TABLES_EXPLANATION_FA.md

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 👥 لایهٔ اول: کاربران و احراز (4 جدول)

### users
**هدف:** ذخیره کاربران و احراز هویت
**داده‌ها:** ایمیل، رمز (hash)، 2FA، تنظیمات
**رابطه:** 1-به-N با licenses, trading_accounts, signals
**مقیاس‌پذیری:** شاخص‌های سریع برای email/username

```sql
SELECT * FROM users 
WHERE email = 'user@example.com' AND status = 'active';
```

### roles
**هدف:** تعریف نقش‌های دسترسی
**نقش‌های پیش‌فرض:** admin, trader, analyst, viewer
**رابطه:** N-به-N با permissions

### user_roles
**هدف:** تخصیص نقش‌ها به کاربران
**داده‌ها:** user_id, role_id
**رابطه:** پیوند N-به-N

### permissions
**هدف:** تعریف مجوزهای دقیق
**مثال:** trading.buy, trading.sell, analysis.view

---

## 🔐 لایهٔ دوم: لایسنس و اشتراک (3 جدول)

### licenses
**هدف:** مدیریت لایسنس و Hardware Binding
**داده‌ها:** license_key, hardware_id, expiration_date
**رابطه:** 1-به-۱ با users
**مقیاس‌پذیری:** شاخص برای وضعیت و تاریخ انقضا

```sql
SELECT * FROM licenses 
WHERE expiration_date < now() AND status = 'active';
```

### subscriptions
**هدف:** مدیریت اشتراک و دورهٔ شامل
**داده‌ها:** plan_type (monthly/quarterly/annual), price, billing_cycle
**رابطه:** N-به-۱ با licenses

### hardware_bindings
**هدف:** ذخیره دستگاه‌های متصل
**داده‌ها:** hardware_id, device_name, activation_date
**رابطه:** N-به-۱ با licenses

---

## 🏦 لایهٔ سوم: حسابات (2 جدول)

### broker_accounts
**هدف:** اتصال MT5 و Brokers
**داده‌ها:** broker_name, login_id, api_key_encrypted
**رابطه:** N-به-۱ با users
**امنیت:** API Key و رمز رمزگذاری شده

### trading_accounts
**هدف:** حساب‌های تجاری و موجودی
**داده‌ها:** balance, equity, margin_level, max_daily_loss
**رابطه:** N-به-۱ با users و broker_accounts
**بروزرسانی:** هر تغییر موجودی

---

## 💱 لایهٔ چهارم: معاملات (4 جدول)

### trades
**هدف:** ذخیره تمام معاملات
**داده‌ها:** entry_price, exit_price, profit_loss, status
**رابطه:** N-به-۱ با trading_accounts, signals
**شاخص:** account, pair, status, time

### open_positions
**هدف:** معاملات بازکن فعلی
**داده‌ها:** current_price, unrealized_pl
**رابطه:** ۱-به-۱ با trades (unique)
**بروزرسانی:** مستمر برای قیمت جاری

### closed_positions
**هدف:** معاملات بسته‌شده
**داده‌ها:** realized_pl, duration
**رابطه:** ۱-به-۱ با trades
**آرشیو:** برای تجزیه‌وتحلیل تاریخی

---

## 📊 لایهٔ پنجم: تحلیل و سیگنال (6 جدول)

### signals
**هدف:** سیگنال‌های تولیدشده
**داده‌ها:** pair, signal_type (BUY/SELL/NEUTRAL), strength (0-1)
**رابطه:** N-به-۱ با trading_accounts
**شاخص:** pair, source, status, time

### ai_decisions
**هدف:** تصمیمات AI بر روی سیگنال‌ها
**داده‌ها:** decision (APPROVE/REJECT/MODIFY), confidence, factors
**رابطه:** ۱-به-۱ با signals

### ai_explanations
**هدف:** توضیح معاملات AI برای کاربر
**داده‌ها:** explanation_text (فارسی), factors, confidence_score
**رابطه:** N-به-۱ با trades

### smart_money_analysis
**هدف:** تحلیل Smart Money Concepts
**داده‌ها:** order_block, liquidity_void, institutional_bias
**رابطه:** ۱-به-۱ با signals

### price_action_analysis
**هدف:** تحلیل Price Action
**داده‌ها:** pattern, support_level, resistance_level, trend
**رابطه:** ۱-به-۱ با signals

---

## 🌍 لایهٔ ششم: اخبار و بازار (2 جدول)

### news_events
**هدف:** اخبار و تأثیرات بازار
**داده‌ها:** title, sentiment, impact (HIGH/MEDIUM/LOW)
**شاخص:** scheduled_time, affected_pairs
**فیلتر:** برای خروج معاملات

### economic_calendar
**هدف:** تقویم اقتصادی
**داده‌ها:** country, indicator_name, forecast, actual
**شاخص:** scheduled_time

---

## 🎯 لایهٔ هفتم: ریسک و عملکرد (3 جدول)

### risk_logs
**هدف:** رویدادهای ریسک (حد روزانه، Margin Call)
**داده‌ها:** event_type, severity (WARNING/ERROR/CRITICAL)
**رابطه:** N-به-۱ با trading_accounts

### trading_journal
**هدف:** یادداشت‌های تجاری برای یادگیری
**داده‌ها:** journal_entry, sentiment (EXCELLENT/GOOD/NEUTRAL/BAD)
**رابطه:** N-به-۱ با trades و users

### performance_metrics
**هدف:** معیارهای عملکرد (Win Rate, Drawdown, Sharpe)
**داده‌ها:** win_rate, max_drawdown, sharpe_ratio
**رابطه:** ۱-به-N با trading_accounts
**محاسبه:** خودکار از trades

---

## 📬 لایهٔ هشتم: اطلاعات (4 جدول)

### notifications
**هدف:** اطلاعات درون سیستم
**داده‌ها:** title, message, type (trade/alert/news)
**رابطه:** N-به-۱ با users
**شاخص:** user_id, is_read

### telegram_messages
**هدف:** پیام‌های Telegram برای کاربران
**داده‌ها:** telegram_chat_id, message_text
**رابطه:** N-به-۱ با users

### email_messages
**هدف:** ایمیل‌های ارسالی
**داده‌ها:** recipient_email, subject, body
**رابطه:** N-به-۱ با users

### screenshots
**هدف:** تصاویر معاملات برای گزارش
**داده‌ها:** file_path, file_size, mime_type
**رابطه:** N-به-۱ با trades و users
**انقضا:** خودکار

---

## 🧪 لایهٔ نهم: بک‌تست (2 جدول)

### backtests
**هدف:** اجرای بک‌تست استراتژی‌ها
**داده‌ها:** start_date, end_date, results (Sharpe, Drawdown)
**رابطه:** N-به-۱ با strategies و users

### optimization_results
**هدف:** نتایج بهینه‌سازی پارامترها
**داده‌ها:** parameter_set, metrics, fitness_score
**رابطه:** N-به-۱ با backtests

---

## ⚙️ لایهٔ دهم: تنظیمات (3 جدول)

### strategies
**هدف:** تعریف استراتژی‌های تجاری
**داده‌ها:** name, strategy_type (price_action/smc/ict)
**رابطه:** N-به-۱ با users

### strategy_configurations
**هدف:** پارامترهای استراتژی
**داده‌ها:** param_name, param_value, param_type
**رابطه:** N-به-۱ با strategies

### system_settings
**هدف:** تنظیمات سیستم
**داده‌ها:** key, value, value_type
**رابطه:** بروزرسانی‌شده توسط users

---

## 📝 لایهٔ یازدهم: نظارت (2 جدول)

### audit_logs
**هدف:** ثبت تمام تغییرات (برای GDPR)
**داده‌ها:** user_id, action, entity_id, old_values, new_values
**شاخص:** user, entity
**اهمیت:** قانونی و امنیتی

### error_logs
**هدف:** ثبت خطاهای سیستم
**داده‌ها:** service, error_message, error_stack
**شاخص:** service, error_level
**تجزیه:** برای مراقبت و خطایابی

---

**تمام جداول برای تولید آماده هستند!**
