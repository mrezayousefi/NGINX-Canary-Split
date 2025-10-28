# NGINX Canary Split (10%)
نمونه‌ی مینیمال برای اجرای **Canary Deployment** با NGINX: ۱۰٪ ترافیک به نسخه‌ی جدید، ۹۰٪ روی نسخه‌ی پایدار. درخواست‌های کاربرانی که در گروه ۱۰٪ قرار می‌گیرند، **همیشه** به همان سرور کانری هدایت می‌شوند و **به سرورهای دیگر فوروارد/Retry نمی‌شوند**.

## ساختار فایل‌ها
```
/etc/nginx/
├─ nginx.conf                 # تعریف split_clients و متغیر $target_upstream
└─ conf.d/
   ├─ upstream.conf           # تعریف upstreamهای stable و canary
   └─ default.conf            # server block و proxy_pass + جلوگیری از retry
```

## فایل‌ها
- [`nginx.conf`](./nginx.conf)  
- [`conf.d/upstream.conf`](./conf.d/upstream.conf)  
- [`conf.d/default.conf`](./conf.d/default.conf)

## نحوه‌ی استفاده
1. فایل‌ها را در مسیرهای فوق قرار دهید (یا مسیرها را مطابق دایرکتوری خودتان اصلاح کنید).
2. صحت پیکربندی را بررسی و ری‌لود کنید:
   ```bash
   sudo nginx -t && sudo systemctl reload nginx
   ```

## تست و اعتبارسنجی
- برای اطمینان از **عدم فوروارد** شدن درخواست‌های کانری:
  - در `default.conf` مقدار زیر وجود دارد:
    ```nginx
    proxy_next_upstream off;
    proxy_next_upstream_tries 1;
    ```
- بررسی نرخ هدایت به کانری (نمونه‌ی ساده – با توجه به لاگ‌فرمت خودتان تغییر دهید):
  ```bash
  grep canary /var/log/nginx/access.log | wc -l
  ```

## نکات
- **چسبندگی (Stickiness):** این نمونه از IP کاربر (`${remote_addr}`) برای تقسیم استفاده می‌کند؛ برای محیط‌های پشت NAT یا موبایل ممکن است بهتر باشد از کوکی یا هدر سفارشی استفاده کنید.
- برای ترافیک بیشتر/کمتر، درصدها را در `split_clients` تغییر دهید.
- برای تحلیل بهتر، یک `log_format` اختصاصی اضافه کنید و `upstream_addr`/`upstream_response_time` را لاگ کنید.

---
**هشدار:** این یک نمونه‌ی آموزشی است. قبل از استفاده در تولید، آن را با نیازهای امنیتی/عملیاتی خودتان تطبیق دهید.
