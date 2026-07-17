# 📁 ساختار دایرکتوری

## 14 فولدر اصلی:

### 1. core/ - هسته سیستم
engine.py | config_manager.py | event_dispatcher.py | logger.py | constants.py

### 2. agents/ - عاملین هوش مصنوعی
decision_agent.py | risk_agent.py | opportunity_agent.py | learning_agent.py | monitor_agent.py

### 3. strategies/ - استراتژی‌های تجاری
base_strategy.py | smc_strategy.py | price_action_strategy.py | ict_strategy.py

### 4. analysis/ - موتورهای تحلیل
price_action/ | smc/ | ict/ | technical/ | sentiment/

### 5. infrastructure/ - زیرساخت
database/ | cache/ | messaging/ | mt5/ | storage/

### 6. services/ - خدمات
licensing/ | subscription/ | account_management/ | notifications/ | reporting/ | backtesting/

### 7. integrations/ - یکپارچه‌سازی‌ها
news_api/ | exchange_api/ | payment/ | external/

### 8. dashboard/ - رابط کاربری
frontend/ | backend/ | public/

### 9. tests/ - آزمون‌ها
unit/ | integration/ | fixtures/

### 10. config/ - تنظیمات
default.yaml | development.yaml | production.yaml | ftmo.yaml | prop_firm.yaml

### 11. docs/ - مستندات
user_guide/ | developer_guide/ | architecture/ | api_reference/

### 12. scripts/ - اسکریپت‌های خودکار
setup.sh | start.sh | backup.py | cleanup.py

### 13. deploy/ - استقرار
docker/ | kubernetes/ | ansible/

### 14. logs/ - گزارش‌ها
trading.log | system.log | errors.log | performance.log
