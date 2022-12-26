---
title: تکنولوژی‌های عبور از فیلترینگ
type: docs
weight: 1
---

# تکنولوژی‌های عبور از فیلترینگ

## تکنولوژی‌های دورزدن سامانه‌های سانسور GFW

| **پروژه** | **#مشارکت‌کنندگان** | **تازه‌ترین کامیت** |  **#ستاره‌های گیت‌هاب** | **اسکریپت نصب** |
|:---:|:---:|:---:|:---:|:---:|
| [V2ray](https://github.com/v2ray/v2ray-core) | 89 | Nov. 2020 | 41.2k | https://github.com/reeceyng/v2ray-agent |
| [Xray](https://github.com/XTLS/Xray-core) | 66 | فعال | 11.1k | https://github.com/NidukaAkalanka/x-ui-english<br>https://github.com/HirbodBehnam/V2Ray-Installer/<br>https://github.com/trojanpanel/install-script |
| [Trojan](https://github.com/trojan-gfw/trojan) | 20 | Nov. 2020 | 17k | https://github.com/reeceyng/v2ray-agent |
| [Trojan-Go](https://github.com/p4gefau1t/trojan-go) | 18 | Sep. 2021 | 5.7k | https://github.com/trojanpanel/install-script |
| [Hysteria](https://github.com/apernet/hysteria) | 9 | فعال | 4.7k | https://github.com/trojanpanel/install-script |
| [Naiveproxy](https://github.com/klzgrad/naiveproxy) | 9 | فعال | 4k | https://github.com/trojanpanel/install-script<br>https://gitlab.com/rwkgyg/naiveproxy-yg/ |


## بهترین روش کارگر در ایران


خلاصه اینکه در شرایط فعلی ایران تنها به گزینه‌های زیر می‌توان فکر کرد:
- Xray + TLS + Trojan
- Trojan-Go (کار نمی‌کند)
- Naiveproxy (کار نمی‌کند)

- پروتکل UDP روی اکثر شبکه و ISP ها بسته شده. در نتیجه پروتکل‌های WireGuard و Hysteria و سایر فناوری‌های مبتنی بر UDP در ایران بی‌فایده هستند.

- به نظر می‌رسد بهترین اسکریپت راه‌انداز در شرایط فعلی، [reeceyng/v2ray-agent](https://github.com/reeceyng/v2ray-agent) و به کارگیری Xray + TLS + Trojan باشد:
  - پشتیبانی از زبان انگلیسی
  - راه‌اندازی وبسایت camouflage
  - پشتیبانی از TLS + Trojan
  - پشتیبانی و تنظیم کردن BBR
  - نداشتن رابط کاربری در بستر وب (پانل)
  
- اگر یک ارتباط ssh با سروری در خارج دارید و به ساده‌ترین روش می‌خواهید یک پراکسی SOCKS درست کنید، [این راهنما](https://github.com/HirbodBehnam/V2Ray-Installer/blob/master/Guides/SSH.md) را دنبال نمایید.
- [این راهنما](https://github.com/iranxray/hope) هم توضیحات و مقالات جامعی به فارسی دارد.
- در یوتیوب کانال‌هایی در این رابطه تولید محتوا می‌کنند که بعضی روش‌ها را به صورت تصویری در محیط ویندوز و به فارسی اجرا می‌کنند: مثل [MrBluepoint](https://www.youtube.com/@MrBluepoint) و  [4rahecomputerfa](https://www.youtube.com/@4rahecomputerfa)

{{< hint warning >}}
**هشدار**  
اسکریپت reeceyng/v2ray-agent یک fork از اسکریپت [mack-a](https://github.com/mack-a/v2ray-agent) است. اسکریپت upstream حدود ۷ هزار ستاره در گیت‌هاب کسب کرده بود.
اگر پژوهشگر امنیت و شبکه هستید، لطفاً اسکریپت reeceyng (و همچنین سایر کدبیس‌ها و اسکریپت‌های این حوزه) را بررسی کنید.
{{< /hint >}}

{{< hint warning >}}
**درخواست**  
لطفاً از روش‌های ناامن و راه‌هایی که ثابت شده منجر به «[کثیف شدن»]({{< ref "dirty-ip" >}}) IP می‌شود استفاده نکنید.
تقریباً می‌توان گفت اگر روشی در جدول بالا نباشد، ناامن است.
{{< /hint >}}
