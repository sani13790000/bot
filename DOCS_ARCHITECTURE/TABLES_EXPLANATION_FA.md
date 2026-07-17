# 📋 TABLES_EXPLANATION_FA.md - توضیح دقیق جداول

**نسخه:** 2.0 | **تاریخ:** 1403/04/27

---

## 🏗️ توضیح هر جدول

### Layer 1: Authentication

#### **users**
- **هدف:** ذخیره کاربران و اطلاعات احراز هویت
- **داده:** ایمیل، رمز‌عبور (hash)، 2FA، تنظیمات شخصی
- **رابطه:** 1-به-N با licenses, trading_accounts, signals, backtests, audit_logs
- **مقیاس‌پذیری:** شاخص‌های سریع برای ایمیل و username

#### **roles**
- **هدف:** تعریف نقش‌های دسترسی (admin, trader, analyst)
- **داده:** نام، توضیح، علامت سیستم
- **رابطه:** N-به-N با permissions

#### **user_roles**
- **هدف:** تخصیص نقش‌های متعدد به کاربران
- **داده:** user_id, role_id
- **رابطه:** پیوند N-به-N
- **مقیاس‌پذیری:** UNIQUE constraint برای جلوگیری از تکرار

#### **permissions**
- **هدف:** تعریف مجوزهای دقیق (trading.buy, analysis.view)
- **داده:** نام، ماژول، عمل
- **رابطه:** N-به-N با roles

#### **role_permissions**
- **هدف:** تخصیص مجوزها به نقش‌ها
- **داده:** role_id, permission_id
- **رابطه:** پیوند N-به-N

---

### Layer 2: Licensing

#### **licenses**
- **هدف:** مدیریت لایسنس و Hardware Binding
- **داده:** کلید لایسنس، نوع محصول، Hardware ID، تاریخ انقضا
- **رابطه:** 1-به-N با subscriptions و hardware_bindings
- **مقیاس‌پذیری:** شاخص برای وضعیت و تاریخ انقضا

#### **subscriptions**
- **هدف:** مدیریت پلن‌های اشتراک
- **داده:** پلن، قیمت، دورهٔ شامل، تجدید خودکار
- **رابطه:** N-به-1 با licenses

#### **hardware_bindings**
- **هدف:** ذخیره دستگاه‌های متصل برای Hardware Binding
- **داده:** hardware_id، نام دستگاه، تاریخ فعال‌سازی
- **رابطه:** N-به-1 با licenses
- **مقیاس‌پذیری:** UNIQUE برای هر دستگاه

---

### Layer 3: Accounts

#### **broker_accounts**
- **هدف:** اتصال به Brokers (MT5، Binance، etc.)
- **داده:** نام broker، شماره حساب، login_id، API Key (رمزگذاری‌شده)
- **رابطه:** 1-به-N با trading_accounts
- **امنیت:** API Key و رمز رمزگذاری‌شده

#### **trading_accounts**
- **هدف:** حساب‌های تجاری و موجودی
- **داده:** موجودی، Equity، Margin Level، حد روزانه
- **رابطه:** N-به-1 با users و broker_accounts
- **بروزرسانی:** هر تغییر موجودی

---

### Layer 4: Trading

#### **trades**
- **هدف:** ذخیره تمام معاملات
- **داده:** جفت ارزی، نوع، قیمت ورود/خروج، سود/ضرر
- **رابطه:** N-به-1 با trading_accounts, N-به-1 با signals و strategies
- **شاخص:** account, pair, status, entry_time

#### **open_positions**
- **هدف:** معاملات بازکن فعلی
- **داده:** جفت، جهت، تعداد، قیمت جاری، P&L بی‌تحقق
- **رابطه:** 1-به-1 با trades
- **بروزرسانی:** مستمر برای قیمت جاری

#### **closed_positions**
- **هدف:** معاملات بسته‌شده
- **داده:** سود/ضرر تحقق‌یافته، مدت‌زمان
- **رابطه:** 1-به-1 با trades
- **آرشیو:** برای تجزیه‌وتحلیل تاریخی

---

### Layer 5: Analysis

#### **signals**
- **هدف:** سیگنال‌های تولید‌شده
- **داده:** جفت، نوع سیگنال، قوت، منبع، تأیید AI
- **رابطه:** 1-به-N با trades و ai_decisions
- **شاخص:** pair, source, status, created_at

#### **ai_decisions**
- **هدف:** تصمیمات AI (تأیید/رد/تعدیل)
- **داده:** تصمیم، اعتماد، دلیل، عوامل
- **رابطه:** 1-به-1 با signals
- **تحلیل:** برای بهبود مدل

#### **ai_explanations**
- **هدف:** توضیح معاملات AI برای کاربر
- **داده:** متن توضیح، عوامل تأثیرگذار
- **رابطه:** N-به-1 با trades و ai_decisions
- **ترجمه:** چند زبانی

#### **smart_money_analysis**
- **هدف:** تحلیل SMC (Order Blocks، Liquidity)
- **داده:** Order Block، Liquidity Void، Institutional Bias
- **رابطه:** 1-به-1 با signals

#### **price_action_analysis**
- **هدف:** تحلیل Price Action (الگو، روند)
- **داده:** الگو، Support/Resistance، روند
- **رابطه:** 1-به-1 با signals

---

### Layer 6: Market Data

#### **news_events**
- **هدف:** اخبار و تأثیرات بازار
- **داده:** عنوان، منبع، تأثیر، احساس بازار
- **شاخص:** scheduled_time, affected_pairs
- **فیلتر:** برای خروج معاملات

#### **economic_calendar**
- **هدف:** تقویم اقتصادی
- **داده:** کشور، شاخص، پیش‌بینی، واقعی
- **شاخص:** scheduled_time

---

### Layer 7: Risk

#### **risk_logs**
- **هدف:** رویدادهای ریسک (حد روزانه، Margin Call)
- **داده:** نوع رویداد، شدت، پیام، اقدام
- **رابطه:** N-به-1 با trading_accounts

#### **trading_journal**
- **هدف:** یادداشت‌های تجاری
- **داده:** احساسات، درس‌ها، بهبودها
- **رابطه:** N-به-1 با trades و users

#### **performance_metrics**
- **هدف:** معیارهای عملکرد (Win Rate, Drawdown, Sharpe)
- **داده:** آمار دوره‌ای
- **رابطه:** 1-به-N با trading_accounts
- **محاسبه:** خودکار از trades

---

### Layer 8: Notifications

#### **notifications**
- **هدف:** اطلاعات درون سیستم
- **داده:** عنوان، پیام، نوع، خوانده نشدن
- **رابطه:** N-به-1 با users

#### **telegram_messages**
- **هدف:** پیام‌های Telegram
- **داده:** chat_id، متن، وضعیت ارسال
- **رابطه:** N-به-1 با users

#### **email_messages**
- **هدف:** ایمیل‌ها
- **داده:** موضوع، متن، وضعیت ارسال
- **رابطه:** N-به-1 با users

#### **screenshots**
- **هدف:** تصاویر معاملات
- **داده:** مسیر فایل، اندازه، MIME
- **رابطه:** N-به-1 با trades و users
- **انقضا:** خودکار

---

### Layer 9: Backtesting

#### **backtests**
- **هدف:** اجرای بک‌تست
- **داده:** دوره، تنظیمات، نتایج (Sharpe, Drawdown)
- **رابطه:** N-به-1 با strategies و users

#### **optimization_results**
- **هدف:** نتایج بهینه‌سازی
- **داده:** مجموعهٔ پارامتر، معیارها، رتبه
- **رابطه:** N-به-1 با backtests

---

### Layer 10: Strategies

#### **strategies**
- **هدف:** تعریف استراتژی‌ها
- **داده:** نام، نوع، پارامترها
- **رابطه:** 1-به-N با backtests

#### **strategy_configurations**
- **هدف:** پارامترهای استراتژی
- **داده:** نام پارامتر، مقدار، نوع
- **رابطه:** N-به-1 با strategies

#### **system_settings**
- **هدف:** تنظیمات سیستم
- **داده:** کلید، مقدار، نوع
- **رابطه:** بروزرسانی‌شده توسط users

---

### Layer 11: Audit

#### **audit_logs**
- **هدف:** ثبت تمام تغییرات
- **داده:** user_id، عمل، entity_id، مقادیر قدیم/جدید
- **شاخص:** user, entity
- **اهمیت:** قانونی و امنیتی

#### **error_logs**
- **هدف:** ثبت خطاها
- **داده:** سرویس، پیام، Stack Trace، سطح
- **شاخص:** service, level
- **تجزیه:** برای مراقبت و خطایابی

---

**تمام جداول برای تولید آماده هستند!**
