# 📤 دستور‌العمل آپلود به GitHub

## 🎯 هدف
آپلود تمام فایل‌های معماری به repository GitHub

---

## 📋 فایل‌های آمادۀ آپلود

```
✅ SYSTEM_OVERVIEW_FA.md       (409 lines, 14 KB)
✅ FOLDER_STRUCTURE_FA.md      (806 lines, 25 KB)
✅ DEPENDENCY_MAP_FA.md        (664 lines, 21 KB)
✅ ARCHITECTURE_FA.md          (942 lines, 34 KB)
✅ INDEX_FA.md                 (450 lines, 15 KB)
✅ README_FA.md                (280 lines, 9 KB)
✅ QUICK_REFERENCE.md          (280 lines, 7 KB)
✅ DELIVERY_SUMMARY.txt        (400 lines, 18 KB)

کل: 4,231 lines، 142 KB
```

---

## 🔧 روش ۱: استفاده از GitHub Web Interface

### مرحلهٔ ۱: برو به GitHub Repository
```
https://github.com/sani13790000/bot
```

### مرحلهٔ ۲: برای هر فایل:
1. کلیک کن بر "Add file" ▼
2. انتخاب کن "Upload files"
3. فایل را drag و drop کن
4. Message بنویس: "Add [filename] - Architecture documentation"
5. کلیک کن "Commit changes"

---

## 🔧 روش ۲: استفاده از Git Command Line (بهتر)

### مرحلهٔ ۱: نصب Git
```bash
# macOS
brew install git

# Ubuntu/Debian
sudo apt-get install git

# Windows
# Download from https://git-scm.com
```

### مرحلهٔ ۲: Clone Repository
```bash
cd ~/projects
git clone https://github.com/sani13790000/bot.git
cd bot
```

### مرحلهٔ ۳: کپی فایل‌ها
```bash
# Copy all .md files to bot directory
cp /home/definable/galaxy-trading-bot/*.md .
cp /home/definable/galaxy-trading-bot/*.txt .
```

### مرحلهٔ ۴: Commit و Push
```bash
# Check status
git status

# Add all files
git add SYSTEM_OVERVIEW_FA.md
git add FOLDER_STRUCTURE_FA.md
git add DEPENDENCY_MAP_FA.md
git add ARCHITECTURE_FA.md
git add INDEX_FA.md
git add README_FA.md
git add QUICK_REFERENCE.md
git add DELIVERY_SUMMARY.txt

# Commit
git commit -m "Add Galaxy Vast Trading Bot™ - Complete Architecture (4,562 lines, 142 KB)

- SYSTEM_OVERVIEW_FA.md: 10 main components explained
- FOLDER_STRUCTURE_FA.md: 14 folders with details
- DEPENDENCY_MAP_FA.md: Service dependencies and flows
- ARCHITECTURE_FA.md: Technical architecture design
- INDEX_FA.md: Master index and reading guide
- README_FA.md: Quick start guide
- QUICK_REFERENCE.md: Fast lookup reference
- DELIVERY_SUMMARY.txt: Complete summary

Total: 4,562 lines of comprehensive Persian documentation"

# Push to GitHub
git push origin main
```

---

## 🔐 اگر GitHub Access نداری:

### مرحلهٔ ۱: GitHub Setup
```bash
# First time setup
git config --global user.email "your_email@example.com"
git config --global user.name "Your Name"
```

### مرحلهٔ ۲: Generate SSH Key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
# Press Enter 3 times
# Copy key:
cat ~/.ssh/id_ed25519.pub
```

### مرحلهٔ ۳: اضافه کردن کلید به GitHub
1. برو به: https://github.com/settings/keys
2. "New SSH key" کلیک کن
3. کلید را paste کن
4. Save کن

### مرحلهٔ ۴: Clone using SSH
```bash
git clone git@github.com:sani13790000/bot.git
cd bot
```

---

## 🔧 روش ۳: استفاده از GitHub Desktop (آسان‌تر)

### مرحلهٔ ۱: دانلود
```
https://desktop.github.com
```

### مرحلهٔ ۲: Clone Repository
1. File → Clone Repository
2. انتخاب: sani13790000/bot
3. کلیک: "Clone"

### مرحلهٔ ۳: کپی فایل‌ها
```bash
# Copy all files to cloned folder
```

### مرحلهٔ ۴: Commit
1. Changes tab کلیک کن
2. فایل‌ها را انتخاب کن
3. "Summary" بنویس
4. "Commit to main" کلیک کن
5. "Push origin" کلیک کن

---

## 📊 فایل‌های برای GitHub

### DOCS_ARCHITECTURE/ میں منتقل کن:
```
DOCS_ARCHITECTURE/
├── SYSTEM_OVERVIEW_FA.md
├── FOLDER_STRUCTURE_FA.md
├── DEPENDENCY_MAP_FA.md
├── ARCHITECTURE_FA.md
├── INDEX_FA.md
├── README_FA.md (یا root میں)
├── QUICK_REFERENCE.md
└── DELIVERY_SUMMARY.txt
```

---

## ✅ بعد از Commit

### تأیید کن:
1. برو به: https://github.com/sani13790000/bot
2. فایل‌ها را ببین در DOCS_ARCHITECTURE/
3. محتوای فایل‌ها را بررسی کن

### Update README.md:
```markdown
# Galaxy Vast Trading Bot™

Architecture documentation in Persian.

## 📚 Documentation

- [INDEX_FA.md](DOCS_ARCHITECTURE/INDEX_FA.md) - Start here
- [SYSTEM_OVERVIEW_FA.md](DOCS_ARCHITECTURE/SYSTEM_OVERVIEW_FA.md) - System overview
- [FOLDER_STRUCTURE_FA.md](DOCS_ARCHITECTURE/FOLDER_STRUCTURE_FA.md) - Folder organization
- [DEPENDENCY_MAP_FA.md](DOCS_ARCHITECTURE/DEPENDENCY_MAP_FA.md) - Module dependencies
- [ARCHITECTURE_FA.md](DOCS_ARCHITECTURE/ARCHITECTURE_FA.md) - Technical architecture
- [QUICK_REFERENCE.md](DOCS_ARCHITECTURE/QUICK_REFERENCE.md) - Quick lookup
```

---

## 🆘 اگر خطا داد:

### Authentication Failed
```bash
# Check credentials
git config --global user.email
git config --global user.name

# Try again with credentials
git push origin main
```

### Permission Denied
```bash
# Check SSH key
ssh -T git@github.com
# Should show: "Hi sani13790000! ..."
```

### Branch Conflict
```bash
# Pull first
git pull origin main

# Then push
git push origin main
```

---

## 📈 Next Steps After Push

1. ✅ Verify files are on GitHub
2. ✅ Update repository README.md
3. ✅ Add GitHub topics: trading, bot, mt5, architecture
4. ✅ Pin QUICK_REFERENCE.md or INDEX_FA.md
5. ✅ Create GitHub Pages (optional)

---

## 🎯 خلاصه

| روش | زمان | سختی | بهترین برای |
|-----|------|------|----------|
| Web Interface | 15 min | آسان | یک بار |
| Git CLI | 5 min | متوسط | توسعه‌دهندگان |
| GitHub Desktop | 10 min | آسان | مبتدیان |

---

**آپلود کنید و اطلاع دهید! 🚀**
