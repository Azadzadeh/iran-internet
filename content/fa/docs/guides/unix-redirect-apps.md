---
title: گذراندن ترافیک برنامه‌ها از SOCKS در لینوکس
type: docs
weight: 4
tags: ["فیلترشکن لینوکس", "عبور ترافیک برنامه‌ها از پراکسی", "پراکسی", "socks", "ساکس", "اوبونتو"]
---

# گذراندن ترافیک برنامه‌ها تحت پراکسی در لینوکس

## طرح مشکل
معمولاً نرم‌افزارهای عبور از فیلترینگ به این صورت عمل می‌کنند که یک ارتباط امن بین کلاینت و سرور تحت پروتکل SOCKS و HTTP ایجاد می‌کنند.
مرورگرهای وب مثل فایرفاکس و کروم قابلیت تنظیم پورت دلخواه برای این پراکسی‌ها را دارند.
مشکل اینجاست که بعضی برنامه‌های تحت سیستم‌عامل‌های یونیکسی به صورت native این ویژگی را ندارند که ترافیکشان را از پورت خاصی تحت پراکسی عبور دهند.

## راه حل

اگر برنامه به صورت محلی SOCKS را پشتیبانی می‌کند، طبیعتاً بهترین راه استفاده از فراخوان مناسب خود برنامه است. مثلاً:

<pre dir="ltr"><code># check if the program supports SOCKS proxy
curl --help | grep -i -C 3 "socks"
</code></pre>

اما اگر اینگونه نباشد، یک راه حل کَلَک‌گونه (hack) استفاده از برنامه‌ی [`proxychains-ng`](https://github.com/rofl0r/proxychains-ng) است.

<pre dir="ltr"><code># install proxychains-ng
sudo apt install proxychains-ng
</code></pre>

کافیست `proxychains4` را با فایل کانفیگ خاصش پیش از برنامه‌ی دلخواه بخوانید تا ترافیک برنامه‌ی دلخواه از SOCKS proxy عبور کند.

<pre dir="ltr"><code># prepend invocation of proxychains4 to my_app
proxychains4 -f /path/to/proxychains.conf my_app --with-original "invocations"

# test download speed of currently active proxy
proxychains4 -f /path/to/proxychains.conf iperf3 -c speedtest-nl-oum.hybula.net -p 5202-5206 -R
</code></pre>

برای کانفیگ `proxychains4` کافیست تنظیمات [پیش‌فرض](https://github.com/rofl0r/proxychains-ng/raw/master/src/proxychains.conf) را کپی کرده و فقط پورت SOCKS را به پورتی که برنامه‌ی عبور از فیلترینگتان تعیین کرده set کنید (خط آخر).
