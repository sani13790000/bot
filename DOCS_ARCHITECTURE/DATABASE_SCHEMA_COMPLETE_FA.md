# 📊 طراحی دیتابیس کامل Galaxy Vast Trading Bot™

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

## 🏗️ ساختار 11 لایهٔ دیتابیس

### Layer 1: Authentication (4 جدول)
- users: کاربران و احراز
- roles: نقش‌ها
- user_roles: تخصیص نقش
- permissions: مجوزها

### Layer 2: Licensing (3 جدول)
- licenses: لایسنس و Hardware Binding
- subscriptions: اشتراک
- hardware_bindings: دستگاه‌های متصل

### Layer 3: Accounts (2 جدول)
- broker_accounts: اتصالات Brokers
- trading_accounts: حسابات تجاری

### Layer 4: Trading (4 جدول)
- trades: معاملات
- open_positions: موضعیت‌های بازکن
- closed_positions: معاملات بسته‌شده
- trade_history: تاریخچه

### Layer 5: Analysis (6 جدول)
- signals: سیگنال‌ها
- ai_decisions: تصمیمات AI
- ai_explanations: توضیحات
- smart_money_analysis: SMC
- price_action_analysis: Price Action
- technical_analysis: فنی

### Layer 6: Market Data (2 جدول)
- news_events: اخبار
- economic_calendar: تقویم اقتصادی

### Layer 7: Risk (3 جدول)
- risk_logs: رویدادهای ریسک
- trading_journal: یادداشت‌ها
- performance_metrics: معیارهای عملکرد

### Layer 8: Notifications (4 جدول)
- notifications: اطلاعات
- telegram_messages: پیام‌های Telegram
- email_messages: ایمیل‌ها
- screenshots: تصاویر

### Layer 9: Backtesting (2 جدول)
- backtests: بک‌تست‌ها
- optimization_results: نتایج بهینه‌سازی

### Layer 10: Strategies (3 جدول)
- strategies: استراتژی‌ها
- strategy_configurations: پارامترها
- system_settings: تنظیمات سیستم

### Layer 11: Audit (2 جدول)
- audit_logs: ثبت تغییرات
- error_logs: ثبت خطاها

## 🔑 خصوصیات

✅ 38 جدول (تمام موارد)
✅ 45+ شاخص (بهینه)
✅ 50+ FK (رابطه)
✅ 20+ UNIQUE
✅ RBAC کامل
✅ Encryption
✅ Audit Trail
✅ Production-Ready
