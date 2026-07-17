# 📋 TABLES_EXPLANATION_FA.md - توضیح تفصیلی جداول

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## جدول به جدول توضیح

### users
**هدف:** ذخیره کاربران سیستم
**داده:** ایمیل، رمز (bcrypt)، 2FA، تنظیمات شخصی
**رابطه:** 1-N با licenses, trading_accounts, signals, backtests
**مقیاس‌پذیری:** شاخص‌های سریع برای جستجو

### roles
**هدف:** تعریف نقش‌های دسترسی (admin, trader, analyst)
**داده:** نام، توضیح
**رابطه:** N-N با permissions

### user_roles
**هدف:** تخصیص نقش‌ها به کاربران
**رابطه:** پیوند N-N
**مقیاس‌پذیری:** UNIQUE برای جلوگیری از تکرار

### licenses
**هدف:** مدیریت لایسنس و Hardware Binding
**داده:** کلید لایسنس، نوع محصول، Hardware ID، تاریخ انقضا
**رابطه:** 1-N با subscriptions و hardware_bindings
**مقیاس‌پذیری:** شاخص برای تاریخ انقضا

### subscriptions
**هدف:** مدیریت اشتراک
**داده:** پلن، قیمت، دورهٔ شامل
**رابطه:** N-1 با licenses

### hardware_bindings
**هدف:** ذخیره دستگاه‌های متصل
**داده:** hardware_id، نام دستگاه
**مقیاس‌پذیری:** UNIQUE برای هر دستگاه

### broker_accounts
**هدف:** اتصال Brokers (MT5، Binance)
**داده:** شماره حساب، login_id، API keys (رمزگذاری‌شده)
**رابطه:** 1-N با trading_accounts

### trading_accounts
**هدف:** حساب‌های تجاری
**داده:** موجودی، equity، margin level
**رابطه:** N-1 با users و broker_accounts
**بروزرسانی:** هر تغییر موجودی

### trades
**هدف:** ثبت تمام معاملات
**داده:** جفت، نوع، قیمت ورود/خروج، P&L
**رابطه:** N-1 با trading_accounts
**شاخص:** account, pair, status, time

### open_positions
**هدف:** موضعیت‌های بازکن فعلی
**داده:** قیمت جاری، P&L بی‌تحقق
**رابطه:** 1-1 با trades
**بروزرسانی:** مستمر

### closed_positions
**هدف:** معاملات بسته‌شده
**داده:** P&L تحقق‌یافته، مدت‌زمان
**رابطه:** 1-1 با trades

### signals
**هدف:** سیگنال‌های تولیدشده
**داده:** جفت، نوع، قوت، منبع
**رابطه:** 1-N با trades

### ai_decisions
**هدف:** تصمیمات AI
**داده:** APPROVE/REJECT/MODIFY، اعتماد
**رابطه:** 1-1 با signals

### ai_explanations
**هدف:** توضیح معاملات AI
**داده:** متن توضیح، عوامل
**رابطه:** N-1 با trades

### smart_money_analysis
**هدف:** تحلیل SMC
**داده:** Order Blocks، Liquidity Voids
**رابطه:** 1-1 با signals

### price_action_analysis
**هدف:** تحلیل Price Action
**داده:** الگو، Support/Resistance
**رابطه:** 1-1 با signals

### news_events
**هدف:** اخبار بازار
**داده:** عنوان، تأثیر، احساس
**شاخص:** scheduled_time, affected_pairs

### economic_calendar
**هدف:** تقویم اقتصادی
**داده:** شاخص، پیش‌بینی
**شاخص:** scheduled_time

### risk_logs
**هدف:** رویدادهای ریسک
**داده:** نوع رویداد، شدت
**رابطه:** N-1 با trading_accounts

### trading_journal
**هدف:** یادداشت‌های تجاری
**داده:** احساسات، درس‌ها
**رابطه:** N-1 با trades

### performance_metrics
**هدف:** معیارهای عملکرد
**داده:** Win Rate، Drawdown، Sharpe
**رابطه:** 1-N با trading_accounts
**محاسبه:** خودکار از trades

### notifications
**هدف:** اطلاعات درون سیستم
**داده:** عنوان، پیام، نوع
**رابطه:** N-1 با users

### telegram_messages
**هدف:** پیام‌های Telegram
**داده:** chat_id، متن
**رابطه:** N-1 با users

### email_messages
**هدف:** ایمیل‌ها
**داده:** موضوع، متن
**رابطه:** N-1 با users

### bale_messages
**هدف:** پیام‌های Bale
**داده:** chat_id، متن
**رابطه:** N-1 با users

### screenshots
**هدف:** تصاویر معاملات
**داده:** مسیر فایل، اندازه
**رابطه:** N-1 با trades

### backtests
**هدف:** اجرای بک‌تست
**داده:** نتایج، معیارها
**رابطه:** N-1 با strategies

### optimization_results
**هدف:** نتایج بهینه‌سازی
**داده:** مجموعهٔ پارامتر، معیارها
**رابطه:** N-1 با backtests

### strategies
**هدف:** تعریف استراتژی‌ها
**داده:** نام، نوع، پارامترها
**رابطه:** 1-N با backtests

### strategy_configurations
**هدف:** پارامترهای استراتژی
**داده:** نام پارامتر، مقدار
**رابطه:** N-1 با strategies

### system_settings
**هدف:** تنظیمات سیستم
**داده:** کلید، مقدار
**رابطه:** تغییر توسط users

### audit_logs
**هدف:** ثبت تمام تغییرات
**داده:** عمل، entity_id، مقادیر
**شاخص:** user, entity, time

### error_logs
**هدف:** ثبت خطاها
**داده:** سرویس، پیام، Stack Trace
**شاخص:** service, level, time

---

**✅ تمام جداول توضیح داده شدند!**
