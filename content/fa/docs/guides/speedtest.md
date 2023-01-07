---
title: آزمون سرعت و پهنای‌باند
type: docs
weight: 4
tags: ["تست سرعت", "سرعت فیلترشکن", "speedtest"]
---

# آزمون سرعت ارتباط

## سایت‌های تست سرعت
معمولاً در سطح نت دیده‌ می‌شود که کاربران برای benchmark کردن وضعیت اتصالشان از روش‌های زیر بهره می‌برند:
- [Ookla Speedtest](https://speedtest.net/)
- [Cloudflare Speedtest](https://speed.cloudflare.com/)
- [Netflix Speedtest](https://fast.com/)

ولی شیوه‌ی کار این سایت‌ها دقیقاً معلوم نیست. برای اینکه آزمون‌ها تکرارشدنی (reproducible) و قاعده‌مند (systematic) باشد، روش `iperf3` توصیه می‌شود.

## روش iper3
کافیست روی رایانه‌ی client خود، نرم‌افزار `iperf3` را نصب کنید:
<pre dir="ltr"><code># install iperf3 on client
sudo apt install iperf3
</code></pre>

یک تست-سرور را از میان [تست-سرورهای عمومی](https://iperf.fr/iperf-servers.php) [اختصاص داده‌شده](https://github.com/R0GGER/public-iperf3-servers) به `iperf` برگزینید.
<pre dir="ltr"><code># we choose speedtest-nl-oum.hybula.net in NL Amsterdam

# client download speed
iperf3 -c speedtest-nl-oum.hybula.net -p 5202-5206 -R

# client upload speed
iperf3 -c speedtest-nl-oum.hybula.net -p 5202-5206
</code></pre>

- ممکن است سرور شلوغ باشد. در این صورت اندکی بعد امتحان کنید.

- با افزودن سوئیچ `u-` می‌توانید سرعت پروتکل UDP را بیازمایید.

- پیشنهاد می‌شود با سوئیچ `2 O-`از دو ثانیه‌ی اول آزمون درگذرید. این مسأله به خصوص در سرعت آپلود نمایان است.

- امکان تست مستقیم بین سرور شخصی خودتان و کلاینت وجود دارد. [راهنمای](https://iperf.fr/iperf-doc.php) `iperf3` را بخوانید.
