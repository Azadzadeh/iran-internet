---
title: "آزمون کثیف نبودن IP"
draft: false
weight: 2
---

# بررسی کثیف نبودن IP
## «لیست» و کثیفی IP
یکی از مشکلاتی که گریبانگیر کاربران شده است، پاسخ به این پرسش است که «آیا IP سرورِ اجاره‌شده کثیف (dirty) است یا نه؟»

در [سامانه‌ی سانسور](https://en.wikipedia.org/wiki/Great_Firewall#Active_filtering)، گاهی اوقات یک دامنه/IP به صورت مطلق مسدود (blocked) نمی‌شود بلکه تنها گاهی کیفیت سرویس‌دهی ([Quality of Service : QoS](https://en.wikipedia.org/wiki/Quality_of_service)) به آن نشانی پایین می‌آید.

این افت کیفیت می‌تواند شامل موارد زیر شود:
- کاهش پهنای باند ارسالی و دریافتی از آن نشانی
- قطع ناگهانی ارتباط TCP توسط شخص ثالث ([TCP RST](https://en.wikipedia.org/wiki/TCP_reset_attack))
- زمانبَر کردن ارتباط بدون قطع کردن (timeout error)
- منع دست‌دادن در جلسه‌ی امن (TLS handshake error)

در صورت بروز چنین مشکلاتی (که تشخیص آن هم آسان نیست)، اصطلاحاً می‌گوییم آن نشانی اینترنتی کثیف شده است و در «لیست» رفته است.


برای مواجه‌نشدن با این مشکلات لازم است **پیش از** نصب و تنظیم نرم‌افزار proxy در سمت سرور، از الف: مسدود نبودن و ب: کثیف نبودن آن نشانی مطمئن شویم.

{{< hint info >}}
**... ولی ولییی من تازه این IP رو سفارش داده‌ بودمممم! 😢**

دقت شود اینکه برای نخستین بار از یک دیتاسنتر یک IP اجاره کرده‌اید یا اینکه مجدداً پس از مدتی کارکردن با یک نشانی دیگر، یک IP تازه سفارش داده‌اید، نشانگر پاک بودن آن نشانی تازه نیست! چراکه آن IP نو ممکن است توسط **سایر مشتریان ایرانی پیش از شما** به دلایل مختلف مثل عدم استفاده از پروتکل/پراکسی امن در *لیست* سامانه‌ی سانسور ایران قرار گرفته و کثیف شده باشد!
{{< /hint >}}

## بررسی مسدود‌ نبودن یک دامنه/آی‌پی

کافیست از دستور `ping` یا `mtr` استفاده کنید:

<pre dir="ltr"><code>ping 123.45.67.890
</code></pre>
اگر در خروجی بسته‌های ICMP رد و بدل نشد، نشانگر انسداد آن نشانی است.

<pre dir="ltr"><code>mtr -y2 my.domain.name
</code></pre>

اگر در آخرین خط خروجی، IP تنظیم شده به آن دامنه را ندیدید، نشانگر فیلترشدن آن دامنه یا DNS poisoning است. مورد دوم موضوع مستقلی از این نوشتار است.

## بررسی کثیف‌ نبودن یک دامنه/آی‌پی

از آنجایی که بسیاری از روش‌های کارگر در شرایط فعلی ایران، بسان یک وبسایت تحت پروتکل HTTPS رفتار می‌کنند تا از سامانه‌ی سانسور در امان بمانند، ما نیز برای آزمایش، یک وبسایت تحت HTTPS بالا می‌آوریم و بررسی می‌کنیم که آیا یک رایانه‌ی بدون پراکسی می‌تواند آن وبسایت را ببیند یا نه؟

1. ابتدا برای دامنه‌تان باید گواهی SSL را تنظیم کنید. [این راهنمایی](https://github.com/iranxray/hope/blob/main/create-tsl-certificate.md#%D8%B3%D8%A7%D8%AE%D8%AA-tls-certificate) مفید است.

2. یک وبسایت بسازید.
{{< hint warning >}}
این بخش در سمت سرور اجرا می‌شود
{{< /hint >}}

مثلاً با اجرای دستورات زیر یک وبسایت ایستا ساخته می‌شود که برای آزمون ما کافی است.

<pre dir="ltr"><code># install hugo-extended on your server
# https://github.com/gohugoio/hugo/releases
wget "https://github.com/gohugoio/hugo/releases/download/v0.109.0/hugo_extended_0.109.0_linux-amd64.deb"
sudo apt install ./hugo_extended_0.109.0_linux-amd64.deb

# make sure it's installed and available
hugo version

# choose a theme from here https://themes.gohugo.io
git clone https://github.com/kc0bfv/autophugo "$HOME/dummy-website/autophugo"
cd "$HOME/dummy-website/autophugo/exampleSite"
# edit config.toml to use your domain name (e.g. my.domain.name)
sed -i 's/example\.org/my\.domain\.name/g' config.toml
rm -rf $HOME/dummy-website/autophugo/exampleSite/public/
hugo --themesDir "../.."
</code></pre>

3. وبسایت را روی پورت ۴۴۳ تحت HTTP**S** بالا بیاورید.
{{< hint warning >}}
این بخش در سمت سرور اجرا می‌شود
{{< /hint >}}

مثلاً می‌توانید از Caddy که یک HTTP Web Server است استفاده کنید یا هر روش دیگری که بلدید (nginx, Apache, etc).
<pre dir="ltr"><code># install caddy on your server
# https://caddyserver.com/docs/install
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy

# make sure it's installed and available
caddy version

caddy file-server --domain my.domain.name --root $HOME/dummy-website/autophugo/exampleSite/public/
</code></pre>

{{< hint warning >}}
دقت کنید که باید پورت ۸۰ و ۴۴۳ برای دستور بالا باز باشد وگرنه اجرا نمی‌شود.
 برای اینکه بفهمید کدام برنامه در حال اشغال چه پورتی است:
(محدود به TCP)

<pre dir="ltr"><code>netstat -nlpt
</code></pre>
سپس برنامه‌ی اشغالگر را برای اجرای آزمون موقتاً متوقف کنید.
{{< /hint >}}


4. بدترین شبکه‌ و ISP-ای که می‌شناسید را برگزینید (معمولاً اپراتورهای موبایل فیلترینگ سختگیرانه‌تری دارند)

مثلاً اگر تست را روی رایانه انجام می‌دهید، سیستم عامل باید به اینترنت گوشی همراه وصل باشد.

5. روی اینترنت شبکه‌ی برگزیده، بدون هیچ پراکسی و VPNی یک HTTP load teser را علیه وبسایتی که بالا آورده‌اید، اجرا کنید
{{< hint warning >}}
این بخش در سمت client (نه سرور) اجرا ‌می‌شود
{{< /hint >}}

[گزینه‌های بسیاری](https://github.com/denji/awesome-http-benchmark#https-benchmark-tools) برای این امر وجود دارد.
مثلاً ما اینجا از [oha](https://github.com/hatoo/oha) استفاده می‌کنیم که یک رابط کاربری تحت console هم دارد و روند تست بار را به صورت انیمیشن نمایش می‌دهد.

![oha demo](/iran-internet/fa/docs/guides/oha-demo.gif)

<pre dir="ltr"><code># install oha on your client
# https://github.com/hatoo/oha/releases
wget -P /tmp/ "https://github.com/hatoo/oha/releases/download/v0.5.5/oha-linux-amd64"
chmod +x /tmp/oha-linux-amd64
mv /tmp/oha-linux-amd64 "$HOME/.local/bin/oha"

# make sure it's installed and available
oha --version

# baseline benchmark of a local website (should be fast)
oha https://telewebion.com -n 20 -c 3 -t 10sec

# typical foreign-hosted website
oha https://en.wikipedia.org/wiki/Tcpip -n 20 -c 3 -t 10sec

# our dummy website on our domain
oha https://my.domain.name -n 20 -c 3 -t 10sec
</code></pre>

- برای هر وبسایت تست، ۲۰ درخواست می‌فرستیم چراکه ما می‌خواهیم کیفیت دسترسی به سایت در شبکه را بیازماییم نه تکنولوژی وب-سرور یا سنگینی سایت را.

- بیشینه تعداد  کارگرهایی که در هر زمان درخواست می‌کنند را به عدد ۳ محدود می‌کنیم 

{{< hint info >}}
این عدد تقریباً می‌تواند پارامتر [Concurrency MUX](https://azadzadeh.github.io/trojan-go/en/advance/mux/) را مدل کند، هرچند از آن سختگیرانه‌تر است زیرا اینجا هر درخواست در یک TLS Session مجزا صورت می‌گیرد ولی در MUX درخواست‌های موازی درون یک TLS Session مشترک انجام می‌شوند
{{< /hint >}}

- به هر درخواست فقط ۱۰ ثانیه اجازه می‌دهیم تا پاسخ بگیرد

می‌توانید از این سوییچ‌ها استفاده نکنید یا oha را بدون سوئچ فراخوانی کنید.

پس از آزمایش، در گزارش به چند پارامتر توجه کنید:
- نرخ موفقیت (success rate) باید ۱ یا چیزی خیلی نزدیک به آن باشد
- متوسط زمان پاسخ باید چیز معقولی باشد (زیر ۱ ثانیه)
- در خط آخر، از n درخواست ارسالی، باید اکثراً خروجی ۲۰۰ داشته باشند

اگر دامنه/آی‌پی شما این مشخصات را در آزمون نشان ندهد مثلاً آزمون timeout شود یا تعداد error response ها زیاد باشد، می‌تواند نشانگر «کثیف بودن» دامنه/آی‌پی باشد.


{{< hint info >}}
**ای بابا! دامنه/IP اَم کثیف بود! (تست موفق نبود) 🥴**

پیش از هر کاری، در دشبورد دیتاسنتری که از آن خدمت می‌گیرید، یک VPS دیگر ایجاد کنید تا یک آی‌پی تازه به شما بدهد. دقت کنید تا پیدا کردن یک IP تمیز، هیچ VPSی را delete نکنید تا IP تکراری به شما ندهد!

بعد از ایجاد VPS جدید، باید آزمون پاکی را دوباره اجرا کنید. 
یادتان باشد که در دشبورد DNS providerتان (مثلاً Cloudflare) باید آی‌پی تازه را به دامنه مربوط سازید.
بهتر است اساساً یک زیردامنه‌ی تازه هم در همینجا ایجاد کنید چرا که ممکن است زیردامنه را هدف قرار داده‌ باشند نه IP.
{{< /hint >}}
{{< hint info >}}
آزمون HTTP load test مطرح شده، یک آزمون سطح بالاست. برای دقت بیشتر بهتر است از روش‌های سطح‌ پایین‌تری مثل تحلیل و مقایسه‌ی `tcpdump`ها در دو سمت کلاینت و سرور یا اجرای هوشمندانه‌ی پروب منفعلی مثل [`p0f`](https://lcamtuf.coredump.cx/p0f3/) در سمت سرور استفاده کرد.

{{< /hint >}}
پس از اینکه یک زوج آی‌پی/دامنه از آزمون پاکی سربلند بیرون آمدند، حالا باید یک نرم‌افزار پراکسی امن را روی سرور نصب کنید تا active probe دیگر نتواند آن را تشخیص دهد و در *"لیست"* بگذارد.

![The List](/iran-internet/fa/docs/guides/the-list.jpeg)
