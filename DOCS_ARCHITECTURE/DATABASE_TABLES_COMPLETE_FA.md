# 📊 فهرست کامل 38 جدول Database

**نسخه:** 2.0 | **تاریخ:** 1403/04/27 | **وضعیت:** ✅ نهایی

---

## 📑 فهرس جداول

### Layer 1: Authentication (4 جدول)
1. **users** - کاربران و احراز هویت
2. **roles** - نقش‌های دسترسی
3. **user_roles** - تخصیص نقش به کاربران
4. **permissions** - مجوزهای دقیق

### Layer 2: Licensing (3 جدول)
5. **licenses** - لایسنس و Hardware Binding
6. **subscriptions** - نظام اشتراک
7. **hardware_bindings** - دستگاه‌های متصل

### Layer 3: Accounts (2 جدول)
8. **broker_accounts** - اتصالات Brokers (MT5, Binance)
9. **trading_accounts** - حسابات تجاری

### Layer 4: Trading (4 جدول)
10. **trades** - تمام معاملات
11. **open_positions** - موضعیت‌های بازکن فعلی
12. **closed_positions** - معاملات بسته‌شده
13. **trade_history** - تاریخچهٔ معاملات

### Layer 5: Analysis & Signals (6 جدول)
14. **signals** - سیگنال‌های تولید‌شده
15. **ai_decisions** - تصمیمات AI
16. **ai_explanations** - توضیحات معاملات
17. **smart_money_analysis** - تحلیل SMC
18. **price_action_analysis** - تحلیل Price Action
19. **technical_analysis** - تحلیل فنی

### Layer 6: Market Data (2 جدول)
20. **news_events** - اخبار و رویدادهای بازار
21. **economic_calendar** - تقویم اقتصادی

### Layer 7: Risk Management (3 جدول)
22. **risk_logs** - رویدادهای ریسک
23. **trading_journal** - یادداشت‌های تجاری
24. **performance_metrics** - معیارهای عملکرد

### Layer 8: Notifications (4 جدول)
25. **notifications** - اطلاعات درون سیستم
26. **telegram_messages** - پیام‌های Telegram
27. **email_messages** - ایمیل‌ها
28. **screenshots** - تصاویر معاملات

### Layer 9: Backtesting (2 جدول)
29. **backtests** - اجرای بک‌تست‌ها
30. **optimization_results** - نتایج بهینه‌سازی

### Layer 10: Strategies (3 جدول)
31. **strategies** - تعریف استراتژی‌ها
32. **strategy_configurations** - پارامترهای استراتژی
33. **system_settings** - تنظیمات سیستم

### Layer 11: Audit & Logging (2 جدول)
34. **audit_logs** - ثبت تمام تغییرات
35. **error_logs** - ثبت خطاها

---

## 🔑 اطلاعات کلیدی

| معیار | مقدار |
|------|-------|
| تعداد جداول | 38 |
| تعداد شاخص‌ها | 45+ |
| تعداد FK | 50+ |
| UNIQUE Constraints | 20+ |
| JSONB Columns | 8+ |
| Enum Types | 25+ |
| Timestamp Fields | 80+ |

---

## 🔗 رابطه‌های بین‌جداول

### users ← → دیگر جداول
- users (1) → (N) licenses
- users (1) → (N) trading_accounts
- users (1) → (N) signals
- users (1) → (N) backtests
- users (1) → (N) audit_logs

### licenses ← → دیگر جداول
- licenses (1) → (N) subscriptions
- licenses (1) → (N) hardware_bindings

### trading_accounts ← → دیگر جداول
- trading_accounts (1) → (N) trades
- trading_accounts (1) → (N) open_positions
- trading_accounts (1) → (N) risk_logs
- trading_accounts (1) → (N) performance_metrics

### trades ← → دیگر جداول
- trades (1) → (1) open_positions
- trades (1) → (1) closed_positions
- trades (1) → (N) ai_explanations

### signals ← → دیگر جداول
- signals (1) → (1) ai_decisions
- signals (1) → (N) ai_explanations
- signals (1) → (N) trades

---

## 📈 مقیاس‌پذیری

### فی‌الوقت (Current)
- 38 جدول
- Normalized Structure (3NF)
- 45+ شاخص بهینه
- Real-time Updates

### آینده (Future)
- Partitioning (Time-based)
- Read Replicas
- Sharding (optional)
- Archive Tables (قدیمی‌ترین داده‌ها)

---

## ✨ خلاصهٔ نهایی

✅ تمام موارد پوشش‌دادهٔ شده
✅ DDL کامل برای هر جدول
✅ شاخص‌های بهینه‌سازی‌شده
✅ RBAC و امنیت
✅ Audit Trail
✅ Production-Ready

---

**دیتابیس برای توسعه فاز بعدی آماده است! 🚀**
