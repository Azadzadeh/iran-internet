---
title: "آموزش ساختن پروکسی MTProto برای تلگرام"
draft: false
weight: 8
tags: ["proxy", "پراکسی", "telegram", "تلگرام", "MTProto", "پروکسی", "prometheus"]
---

# آموزش ساختن پروکسی MTProto تلگرام
در این نوشتار، یک پراکسی با پروتکل MTProto را گام به گام می‌سازیم.

![MTProto-chad](/docs/guides/mtproto-chad.jpeg)


## انواع پیاده‌سازی‌ها
در سمت سرور پیاده‌سازی‌های گوناگونی در زبان‌های مختلف برای این پروتکل نوشته شده است.
برای هر کدام از این راهکار‌ها، اسکریپت‌های نصبی وجود دارد که فرآیند راه‌اندازی سمت سرور را راحت می‌کند.

| **پروژه** | **#مشارکت‌کنندگان** | **تازه‌ترین کامیت** |  **#ستاره‌های گیت‌هاب** | **زبان** |
|:---:|:---:|:---:|:---:|:---:|
| [TelegramMessenger/MTProxy](https://github.com/TelegramMessenger/MTProxy) رسمی | 11 | Oct. 2019 | 4.2k | C |
| [alexbers/mtprotoproxy](https://github.com/alexbers/mtprotoproxy) | 14 | فعال | 1.4k | Python |
| [seriyps/mtproto_proxy](https://github.com/seriyps/mtproto_proxy) | 3 | فعال | 0.36k | Erlang |
| [9seconds/mtg](https://github.com/9seconds/mtg) | 18 | فعال | 1.6k | Go |


## پیاده‌سازی mtg

ما در اینجا از پروژه‌ی 9seconds/mtg بدون اسکریپت استفاده می‌کنیم.
ویژگی‌های این پروژه که در [README](https://github.com/9seconds/mtg/blob/master/README.md) آن قابل خواندن است شامل موارد زیر می‌شود:
- مقاومت در برابر حمله‌ی تکرار بسته (replay attack)
- جلونهادن دامنه‌ی استتار (domain fronting)
- faketls

تفاوت این پروژ‌ه با سایرین، تاکید طراحش بر موارد زیر است:
- کم مصرفی برنامه تا در VPS های با منابع پایین و در نتیجه ارزان هم قابل راه‌اندازی باشد
- راحتی در deploy کردن
- تک رمزه‌ بودن (single secret)
- پشتیبانی نکردن از adtag در نسخه‌ی ۲
- نداشتن رابط کاربری تحت وب برای مدیریت
- لیست مسدود‌کردن IPهای مشکوک، پیاده‌سازی شده به صورت محلی (native)
- قابلیت ترکیب با v2ray, trojan, Gost و کاربری به شکل کتابخانه

‌نویسنده‌ی این پیاده‌سازی فلسفه‌ای دارد مبنی بر اینکه پراکسی چیزی است که به صورت خصوصی بین چند ده نفر پخش می‌شود.
تا جای ممکن باید سعی شود از سامانه‌ی سانسور پنهان بماند. در صورت کشف هم باید بتوان به سرعت در سرور دیگری راه‌اندازی مجدد کرد.
در نتیجه در نسخه‌ی ۲، دیگر از adtag پشتیبانی نمی‌شود.

## گام‌های راه‌اندازی

فرض می‌شود که دامنه‌ و سرور VPS با دسترسی `root` دارید.

1. ایجاد زیردامنه در یک سرویس DNS مثل CloudFlare و مرتبط ساختن زیردامنه به IP سرور

به [این راهنما](https://github.com/iranxray/hope/blob/main/create-tsl-certificate.md) مراجعه شود.

پس از این مرحله، در سمت کلاینت یک `ping` بگیرید تا از درستی تنظیمات مطمئن شوید:
<pre dir="ltr"><code># this should point to your server IP
ping my.domain.name
</code></pre>

2. بالا آوردن وبسایت استتار

یک وبسایت معمولی تحت HTTPS در پورت ۴۴۳ بالا بیاورید. پورت ۸۰ را هم به همینجا redirect کنید.
هر روشی که بلدید را می‌توانید انجام دهید (مثل nginx, apache, etc)
یا اینکه از روشی که در ادامه می‌آید استفاده کنید:

مراحل ۲ و ۳ از [این بخش]({{< ref "dirty-ip/#بررسی-کثیف-نبودن-یک-دامنهآیپی" >}})
را انجام دهید با این تفاوت‌ها:
- محتویات ریشه‌ی سایت (`root`) باید در آدرس `/var/www/` باشد:
<pre dir="ltr"><code>hugo --minify --themesDir ../.. --destination /var/www/
</code></pre>

- سرور HTTP با systemd مدیریت شود:

این کار از این جهت انجام می‌شود که اگر سرور خاموش/روشن شد، سایت دوباره بالا بیاید.

تنظیم caddy:
<pre dir="ltr"><code>$ cat /etc/caddy/Caddyfile
my.domain.name {
    root * /var/www/
    # Enable the static file server.
    file_server
    log {
        output file /var/log/caddy/decoy-site.log
    }
}</code></pre>

روشن کردن سرویس caddy:

<pre dir="ltr"><code>systemctl daemon-reload
systemctl enable caddy
systemctl restart caddy
# caddy should be up and running
systemctl status caddy
# test my.domain.name from your client to make sure website is up
</code></pre>

3. نصب mtg در سرور

<pre dir="ltr"><code># download proper version from https://github.com/9seconds/mtg/releases 
wget "https://github.com/9seconds/mtg/releases/download/v2.1.7/mtg-2.1.7-linux-amd64.tar.gz"
# extract and move mtg binary to /usr/local/bin/mtg
</code></pre>

4. تنظیم mtg

رمز را بسازید:
<pre dir="ltr"><code>mtg generate-secret my.domain.name
</code></pre>

تنظیمات mtg را بسازید:
<pre dir="ltr"><code>$ cat /etc/mtg.toml
secret = "your-secret-here"
bind-to = "0.0.0.0:2086"
doh-ip = "1.1.1.1"

# prometheus metrics integration.
[stats.prometheus]
# enabled/disabled
enabled = true
# host:port where to start http server for endpoint
bind-to = "127.0.0.1:1212"
# prefix of http path
http-path = "/"
# prefix for metrics for prometheus
metric-prefix = "mtg"
</code></pre>

این تنظیمات بیانگر این است که پورت دسترسی عمومی پراکسی روی ۲۰۸۶ است. برای DNS سمت سرور از سرویس DNS-over-HTTPS کلادفلر استفاده می‌شود.
همچنین برای مانیتورینگ، prometheus را به کار می‌گیریم.

سپس برای mtg یک فایل سرویس بسازید:
<pre dir="ltr"><code>$ cat /etc/systemd/system/mtg.service
[Unit]
Description=mtg - MTProto proxy server
Documentation=https://github.com/9seconds/mtg
After=network.target

[Service]
ExecStart=/usr/local/bin/mtg run /etc/mtg.toml
Restart=always
RestartSec=3
DynamicUser=true
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
</code></pre>

سرویس را روشن کنید:

<pre dir="ltr"><code>systemctl daemon-reload
systemctl enable mtg
systemctl start mtg
systemctl status mtg
</code></pre>

5. دیدن‌ تنظیم برای کلاینت

در آخرین مرحله، برای مشاهده‌ی کانفیگ برای set کردن در سمت کلاینت تلگرام از دستور زیر استفاده کنید:

`mtg access /etc/mtg.toml`

کافیست لینک‌ `tmu_url` را به کاربران بدهید.
اگر دقت کنید، تنظیمات به شکل کد QR هم برای share کردن راحت‌تر ایجاد شده است.
می‌توانید به جای `server` در سمت کلاینت، از زیر‌دامنه‌ای که ایجاد کرده‌اید استفاده کنید تا در صورت کثیف شدن IP، و انتقال به سرور دیگر، کاربر مجبور نباشد مجدداً تنظیمات تازه دریافت کند.

در ادامه‌ گام‌های ایجاد کردن dashboard برای رصد پارامتر‌های تعداد اتصال، میزان ترافیک مصرفی و... پراکسی با استفاده از prometheus می‌آید.

6. نصب [prometheus](https://prometheus.io/)
<pre dir="ltr"><code># install proper version from https://prometheus.io/docs/prometheus/latest/installation/
wget "https://github.com/prometheus/prometheus/releases/download/v2.37.5/prometheus-2.37.5.linux-amd64.tar.gz"
# extract and move prometheus binary to /usr/local/bin/
</code></pre>

7. تنظیم prometheus

<pre dir="ltr"><code>$ cat /etc/prometheus-mtg.yml
# my global config
global:
  scrape_interval: 20s
  evaluation_interval: 20s
# A scrape configuration containing exactly one endpoint to scrape:
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "Monitoring_MTProto"
    # metrics_path defaults to '/metrics'
    metrics_path: "mtg"
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["localhost:1212"]
</code></pre>


7. ساخت Hash رمز‌عبور برای داشبورد مانیتورینگ تحت وب
<pre dir="ltr"><code>$ apt install python3-bcrypt
$ cat /root/bin/gen-pass-prometheus.py
import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())

$ python3 ./gen-pass.py

</code></pre>
9. تنظیم داشبورد تحت وب

دقت کنید که داشبورد را تحت پروتکل HTTPS بالا می‌آوریم پس به certificate نیاز است.

<pre dir="ltr"><code>$ cat /etc/prometheus-mtg-web.yml
# TLS and basic authentication configuration example.
tls_server_config:
  cert_file: /etc/letsencrypt/live/my.domain.name/fullchain.pem
  key_file: /etc/letsencrypt/live/my.domain.name/privkey.pem
basic_auth_users:
  your_username: YOUR_HASHED_PASSWORD_HERE
</code></pre>


10. ساخت سرویس فایل برای prometheus

<pre dir="ltr"><code>$ cat /etc/systemd/system/prometheus-mtg.service
[Unit]
Description=Prometheus Server for MTProto Telegram Proxy
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
Restart=on-failure
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus-mtg.yml \
  --web.config.file=/etc/prometheus-mtg-web.yml \
  --web.listen-address=:5050 \
  --storage.tsdb.path=/opt/prometheus/data

[Install]
WantedBy=multi-user.target
</code></pre>

11. روشن کردن سرویس prometheus

<pre dir="ltr"><code>systemctl daemon-reload
systemctl enable prometheus-mtg.service
systemctl start prometheus-mtg.service
systemctl status prometheus-mtg.service
</code></pre>

حال می‌توانید به https://my.domain.name:5050 رفته و وضعیت پراکسی را مانیتور کنید.
مثلاً می‌توانید نمودار میزان ترافیک مصرفی کاربران بر حسب زمان یا تعداد حملات replay-attack را رسم کنید.
