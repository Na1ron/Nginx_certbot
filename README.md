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
