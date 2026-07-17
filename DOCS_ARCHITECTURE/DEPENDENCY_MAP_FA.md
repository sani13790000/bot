# 🔗 نقشهٔ وابستگی‌ها

## جریان داده اصلی:
MetaTrader 5 → Data Collector → Cache → Analysis → Signals → Agents → Risk → Execution → Monitor → Database

## نقاط بحرانی:
۱. MT5 Connection - Backup: Database + Cache
۲. Database - Backup: Redis Cache
۳. Order Execution - Retry: ۳ بار

## ترتیب شروع (Bootstrap):
Config → Logger → Database → Cache → Queue → MT5 → Analysis → Agents → Services → Dashboard → Ready
