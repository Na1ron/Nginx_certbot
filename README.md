# Nginx + SSL configure virtualhost

### 1.Зашёл на машину, прокинул туда свой другой ключ ed25519:

`` ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEYOk/yxqlYln4pGaEoRugZ+baTnquUx0Ae5a5ROFDub vladi@DESKTOP-3GC0E3L ``
### 2. Cделал apt update, затем установил установил nginx,certbot произведя:

```
apt install nginx -y

apt install certbot -y

apt install python3-certbot-nginx 
```
### Далее удалил дефолтный файлик (default) с виртуальным хостом и заменил своим:

```
root@prodavec:/etc/nginx/sites-available# cat prodavec
server {
    listen 80;
    server_name prodavec.uberlegenheit.ru;

    return 301 https://host$request_uri;

   }

server {
    listen 443 ssl;
    server_name prodavec.uberlegenheit.ru www.prodavec.uberlegenheit.ru;

    ssl_certificate /etc/letsencrypt/live/prodavec.uberlegenheit.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/prodavec.uberlegenheit.ru/privkey.pem;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    access_log /var/log/nginx/access.log myformat;


    location / {
        try_files $uri $uri/ =404;

   }

    error_page 404 403 401 400 /error.html;

    location = /error.html {
        internal;

   }
}
```

### 3. Подготовил 2 файлика (кишки сайта), 1 - обычная index.html с hello world, 2-й для ошибок с текстом - idite k adminy sait ymer:

```
root@prodavec:/var/www/html# ll
total 16
drwxr-xr-x 2 root root 4096 Mar  1 23:33 ./
drwxr-xr-x 3 root root 4096 Feb 20 18:36 ../
-rw-r--r-- 1 root root   26 Mar  1 23:33 error.html
-rw-r--r-- 1 root root   12 Mar  1 23:32 index.html
root@prodavec:/var/www/html# cat index.html
hello world
root@prodavec:/var/www/html# cat error.html
sait ymer, idite k adminy
root@prodavec:/var/www/html# pwd
/var/www/html
```
### 4. Выполнил curl -ILk 10.182.10.160 и curl без ключей:

```
root@prodavec:/var/www/html# curl -ILk 10.182.10.160
HTTP/1.1 301 Moved Permanently
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 03 Mar 2024 22:41:45 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: https://host/

curl: (6) Could not resolve host: host
```

```
root@prodavec:/var/www/html# curl 10.182.10.160
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```
Не отображается hello world через консоль, т.к. идёт редирект на https (который уже определил в confige

### 5. Поддомен получен - prodavec.uberlegenheit.ru и описан в конфиге, всё работает

```
root@prodavec:/var/www/html# curl prodavec.uberlegenheit.ru
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```
### 6. Директивы для доступа по https добавлены в конфиг virtual host'a:

```
server {
    listen 443 ssl;
    server_name prodavec.uberlegenheit.ru www.prodavec.uberlegenheit.ru;

    ssl_certificate /etc/letsencrypt/live/prodavec.uberlegenheit.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/prodavec.uberlegenheit.ru/privkey.pem;
```
### 7. Получил сертификаты с помощью certbot --nginx -d prodavec.uberlegenheit.ru:

```
root@prodavec:/etc/letsencrypt# certbot --nginx -d prodavec.uberlegenheit.ru
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Certificate not yet due for renewal
Deploying certificate
Successfully deployed certificate for prodavec.uberlegenheit.ru to /etc/nginx/sites-enabled/prodavec
Congratulations! You have successfully enabled HTTPS on https://prodavec.uberlegenheit.ru

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**Перед этим создал cli.ini в etc/letsencrypt с содержанием:***

```
email = jcatswill@gmail.com
agree-tos = True
non-interactive = True
preferred-challenges = http
```
**email = jcatswill@gmail.com** указал свою почту, похуй, пусть летят спам-письма от letsencrypt'a

**agree-tos = True:** Это означает, что пользователь согласен с условиями использования и политикой конфиденциальности Certbot.

**non-interactive = True:** Это указывает, что процесс получения сертификата должен происходить автоматически, без взаимодействия с пользователем.

**preferred-challenges = http:** Здесь указывается тип проверки, который будет использоваться для получения сертификата. В данном случае указан HTTP-протокол.

### 8. Проверил работу сайта в браузере, всё равботает c http на https редиректит:

![Logotype](./work.png)

**В связи с тем, что при просмотре access-логов не выдавался  реальный ip-шник реального посетителя сайта, скорректировал log format,**

в /etc/nginx/nginx.conf определил новый log fotmat:

```
        log_format myformat '$http_x_real_ip $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" ';

        error_log /var/log/nginx/error.log;
```

**Далее добавил этот формат (myformat) в свой virtual host /etc/nginx/sites-available/prodavec :**

```
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    access_log /var/log/nginx/access.log myformat;
```

Теперь при просмотрел логов (через tail -f /var/log/nginx/access.log) показывается реальные ip пользователей открывающих мой сайтец:

```
root@prodavec:/etc/nginx# tail -f /var/log/nginx/access.log
109.194.102.169 - [03/Mar/2024:22:38:21 +0000] "GET / HTTP/1.0" 304 0 "-"
109.194.102.169 - [03/Mar/2024:22:38:22 +0000] "GET / HTTP/1.0" 304 0 "-"
127.0.0.1 - - [03/Mar/2024:22:41:30 +0000] "HEAD / HTTP/1.1" 301 0 "-" "curl/7.81.0"
10.182.10.160 - - [03/Mar/2024:22:41:45 +0000] "HEAD / HTTP/1.1" 301 0 "-" "curl/7.81.0"
10.182.10.160 - - [03/Mar/2024:22:43:16 +0000] "GET / HTTP/1.1" 301 178 "-" "curl/7.81.0"
10.182.10.160 - - [03/Mar/2024:22:43:40 +0000] "GET / HTTP/1.1" 301 178 "-" "curl/7.81.0"
10.182.10.160 - - [03/Mar/2024:22:43:48 +0000] "GET / HTTP/1.1" 301 178 "-" "curl/7.81.0"
10.182.10.160 - - [03/Mar/2024:22:44:00 +0000] "GET / HTTP/1.1" 301 178 "-" "curl/7.81.0"
10.182.10.1 - - [03/Mar/2024:22:46:07 +0000] "GET / HTTP/1.0" 301 178 "-" "curl/7.81.0"
109.194.102.169 - [03/Mar/2024:22:47:22 +0000] "GET /favicon.ico HTTP/1.0" 404 26 "https://prodavec.uberlegenheit.ru/"
109.194.102.169 - [03/Mar/2024:23:04:06 +0000] "GET / HTTP/1.0" 200 12 "-"
109.194.102.169 - [03/Mar/2024:23:04:06 +0000] "GET / HTTP/1.0" 304 0 "-"

```

### 9. Редирект трафика с http на https был настроен с помощью корректировок в virtual host:

``  return 301 https://host$request_uri; ``

### 10. Автообновление сертификатов настроил с помощью systemd-юнита и таймера для него

```
root@prodavec:/etc/systemd/system# ll
total 64
drwxr-xr-x 14 root root 4096 Mar  2 00:24  ./
drwxr-xr-x  5 root root 4096 Feb 25 13:20  ../
-rw-r--r--  1 root root  140 Mar  2 00:24  certbot-semi-automatic.service
-rw-r--r--  1 root root  125 Mar  2 00:24  certbot-semi-automatic.timer
```

Содержимое юнита для обновления сертификата:

```
root@prodavec:/etc/systemd/system# cat certbot-semi-automatic.service
[Unit]
Description=Certbot Renewal

[Service]
ExecStart=/usr/bin/certbot renew --force-renewal --post-hook "systemctl reload nginx.service"
```

Содержимое юнита с таймером:

```
root@prodavec:/etc/systemd/system# cat certbot-semi-automatic.timer
[Unit]
Description=Timer for Certbot Renewal

[Timer]
OnBootSec=300
OnUnitActiveSec=1w

[Install]
WantedBy=multi-user.target
```

Не смог выполнить обновление сертификатов в ручную через renew , получил такую ошибку:

*certbot renew --allow-subset-of-names -d prodavec.uberlegenheit.ru*

```
Currently, the renew verb is capable of either renewing all installed certificates that are due to be renewed or renewing a single certificate specified by its name. If you would like to renew specific certificates by their domains, use the certonly command instead. The renew verb may provide other options for selecting certificates to renew in the future.
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /var/log/letsencrypt/letsencrypt.log or re-run Certbot with -v for more details.
```

Погуглил проблему и понял, что renew обновляет все серификаты, и не может обновить сертификат для конкретного домена

Выполнил certbot renew , сертификаты обновились :

```
root@prodavec:/etc/letsencrypt# certbot renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/prodavec.uberlegenheit.ru.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Certificate not yet due for renewal

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
The following certificates are not due for renewal yet:
  /etc/letsencrypt/live/prodavec.uberlegenheit.ru/fullchain.pem expires on 2024-05-30 (skipped)
No renewals were attempted.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```


**Обновить отдельно сертификат для отдельного домена можно с помощью:**

`` certbot certonly --webroot -w /var/www/html -d prodavec.uberlegenheit.ru  ``

Попытался обновить серт с помощью sudo certbot certonly --webroot -w /var/www/html -d prodavec.uberlegenheit.ru

Но он не обновился, т.к. по мнению сертбота ещё слишком рано для обновления (так серты действуют 90дн) :

```
root@prodavec:/etc/letsencrypt# sudo certbot certonly --webroot -w /var/www/html -d prodavec.uberlegenheit.ru
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Certificate not yet due for renewal

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Certificate not yet due for renewal; no action taken.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
Проверку сколько ещё будет действовать серт можно сделать так:

```
root@prodavec:/etc/letsencrypt# certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: prodavec.uberlegenheit.ru
    Serial Number: 49b1c06dbc456c929f5e6863dc90aa96f3f
    Key Type: RSA
    Domains: prodavec.uberlegenheit.ru
    Expiry Date: 2024-05-30 22:37:07+00:00 (VALID: 87 days)
    Certificate Path: /etc/letsencrypt/live/prodavec.uberlegenheit.ru/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/prodavec.uberlegenheit.ru/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
### Выполнил принудительное обновление серта в ручную с помощью ключа --force-renewal , всё обновилось:

```
root@prodavec:/etc/letsencrypt# sudo certbot certonly --webroot -w /var/www/html -d prodavec.uberlegenheit.ru --force-renewal
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Renewing an existing certificate for prodavec.uberlegenheit.ru

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/prodavec.uberlegenheit.ru/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/prodavec.uberlegenheit.ru/privkey.pem
This certificate expires on 2024-06-01.
These files will be updated when the certificate renews.
```
