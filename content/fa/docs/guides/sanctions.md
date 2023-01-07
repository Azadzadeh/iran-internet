---
title: دور زدن تحریم‌های خارجی
type: docs
weight: 10
tags: ["تحریم", "تحریم لینوکس", "عبور از تحریم", "دور زدن تحریم", "sanction", "dns", "تنظیم dns در اوبونتو"]
draft: false
---

# دور زدن تحریم‌های خارجی

گاهی مهندسان برنامه نویس ایرانی نیاز به دسترسی به سایت‌ها و خدماتی دارند که فیلتر نشده‌اند ولی تحریم شده‌اند (مثلاً docker، VirtualBox, etc)

برای اینکه کل برنامه‌های تحت سیستم را از تحریم عبور دهید می‌توانید از سرویس تغییر DNS [شکن](https://shecan.ir/) و [بگذر](https://begzar.ir/) استفاده کنید.

<pre dir="ltr"><code># set your system's DNS to
# 178.22.122.100 or 185.51.200.2 for shecan
# 185.55.226.26 or 185.55.225.25 for begzar
# if systemd sets DNS in your Linux distribution:
sudo systemd-resolve -i enp0s25 --set-dns="178.22.122.100"
</code></pre>

دقت شود که با restart شدن سیستم، تنظیم با روش فوق به روتر reset می‌شود.

خوبی دستور بالا این است که می‌توان *صرفاً برای مدتی که به دورزدن تحریم نیاز دارید*، از سرویس‌های مذکور استفاده کنید. مثلاً قبل از `sudo apt update && sudo apt upgrade -y`. برای بازگرداندن به حالت عادی (روتر) :

<pre dir="ltr"><code>sudo systemd-resolve -i enp0s25 --set-dns="192.168.1.1"
</code></pre>


- به جای `enp0s25` نام interface خود را قرار دهید (دستور `ip address` )

{{< hint warning >}}
**توجه 🔎**  
هیچ ایده‌ای از امنیت خدمات‌ مذکور نداریم.
{{< /hint >}}
