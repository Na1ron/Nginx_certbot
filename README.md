# Nginx + SSL configure virtualhost

### 1.Зашёл на машину, прокинул туда свой другой ключ ed25519:

`` ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEYOk/yxqlYln4pGaEoRugZ+baTnquUx0Ae5a5ROFDub vladi@DESKTOP-3GC0E3L ``
### 2. Cделал apt update, затем установил установил nginx,certbot произведя:

``apt install nginx -y

apt install certbot -y

apt install python3-certbot-nginx ``
