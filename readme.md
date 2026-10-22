# حذف فایل `.env` از تاریخچه‌ی Git (ویندوز)

این راهنما نشان می‌دهد چطور فایل `.env` را با ابزار `git-filter-repo` از **کل commitهای** یک repo پاک کنید.

> ⚠️ **هشدار امنیتی:** اگر `.env` شامل کلید API، رمز دیتابیس یا توکن واقعی بوده و روی remote (مثل GitHub) push شده، همه‌ی آن secretها را **عوض (rotate)** کنید. حذف از تاریخچه جلوی کپی‌های قبلی را نمی‌گیرد.

## پیش‌نیازها

- Git
- Python 3 (هنگام نصب، گزینه‌ی **Add Python to PATH** فعال باشد)

## مراحل

### ۱. نصب ابزار

```cmd
python -m pip install git-filter-repo
```

ابزار `git-filter-repo` را نصب می‌کند. استفاده از `python -m pip` باعث می‌شود نیازی به تنظیم PATH نباشد.

### ۲. بررسی نصب

```cmd
python -m git_filter_repo --version
```

فقط نسخه‌ی ابزار را نشان می‌دهد و به repo دست نمی‌زند.

### ۳. پشتیبان‌گیری از `.env`

```cmd
copy .env ..\env-backup
```

ممکن است `.env` از پوشه پاک شود، پس قبل از شروع یک کپی نگه دارید.

### ۴. حذف `.env` از کل تاریخچه

```cmd
python -m git_filter_repo --invert-paths --path .env --force
```

| بخش دستور | توضیح |
|---|---|
| `--path .env` | فایل `.env` را هدف می‌گیرد |
| `--invert-paths` | همه چیز را نگه می‌دارد **به‌جز** `.env` |
| `--force` | اجازه می‌دهد روی repo قدیمی (غیر clone تازه) هم اجرا شود |

این دستور تاریخچه را بازنویسی می‌کند: hash همه‌ی commitها عوض می‌شود.

### ۵. بررسی نتیجه

```cmd
git log --oneline --all -- .env
```

اگر خروجی **خالی** بود، `.env` دیگر در هیچ commitی نیست.

برای اطمینان از سالم بودن commitها:

```cmd
git log --oneline --all
```

### ۶. برگرداندن `.env` و ignore کردن آن

```cmd
copy ..\env-backup .env
echo .env >> .gitignore
git add .gitignore
git commit -m "Ignore .env"
```

### ۷. (در صورت داشتن remote) push اجباری

`filter-repo` بعد از اجرا remote را حذف می‌کند، پس دوباره اضافه کنید:

```cmd
git remote add origin https://github.com/USER/REPO.git
git push origin --force --all
git push origin --force --tags
```

## عیب‌یابی

| مشکل | راه‌حل |
|---|---|
| `'pip' is not recognized` | از `python -m pip` استفاده کنید |
| `git: 'filter-repo' is not a git command` | از `python -m git_filter_repo` استفاده کنید، یا پوشه‌ی `Scripts` پایتون را به PATH اضافه کنید |
| `Refusing to destructively overwrite repo history` | دستور را با `--force` اجرا کنید، یا از repo یک clone تازه بگیرید |
| `git log` هنوز commit نشان می‌دهد | دستور مرحله‌ی ۴ اجرا نشده یا با خطا متوقف شده؛ خروجی آن را بررسی کنید |

## نکات پایانی

- هم‌تیمی‌ها باید repo را **دوباره clone** کنند، نه pull.
- forkها و cacheهای GitHub ممکن است هنوز نسخه‌ی قدیمی را داشته باشند.
- بهتر است یک فایل `.env.example` (بدون مقدارهای واقعی) commit کنید.