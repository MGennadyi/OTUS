# Установка Apache
### 1. Подмена встроенного web-сайта на свой собственный
```
apt install net-tools curl -y
netstat -tlpn
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      498/sshd: /usr/sbin
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      521/postgres
tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      1002/sshd: mgb@pts/
tcp        0      0 0.0.0.0:6432            0.0.0.0:*               LISTEN      478/pgbouncer
tcp6       0      0 :::22                   :::*                    LISTEN      498/sshd: /usr/sbin
tcp6       0      0 ::1:5432                :::*                    LISTEN      521/postgres
tcp6       0      0 ::1:6010                :::*                    LISTEN      1002/sshd: mgb@pts/
tcp6       0      0 :::6432                 :::*                    LISTEN      478/pgbouncer
```
### Уст Apache2 на Ubuntu
```
ifconfig
ip подставляем в браузер
sudo apt install apache2 -y
systemctl status apache2
curl localhost # Просмотр кода стартовой страницы
ip подставляем в браузер = стартовая страница apache2
```

### Публикация страницы
```
mkdir -p /var/www/test.loc
mkdir -p /var/www/demo.loc
vim /var/www/test.loc/index.html
<h1>Helo world</h1>
<h2>Test</h2>
vim /var/www/demo.loc/index.html
<h1>Helo world</h1>
<h2>Demo</h2>
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/test.loc.conf
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/demo.loc.conf
vim /etc/apache2/sites-available/test.loc.conf
ServerName test.loc
ServerAlias www.test
DocumentRoot /var/www/test.loc
vim /etc/apache2/sites-available/demo.loc.conf
ServerAlias www.demo
ServerName demo.loc
DocumentRoot /var/www/demo.loc
# Привязка конфига сайта к серверу apache2
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
a2ensite test.loc
a2ensite demo.loc
To activate the new configuration, you need to run:  systemctl reload apache2
```
### Права
```
sudo chmod -R 775 /var/www
vim /etc/hosts
192.168.0.17 firma1.comp
192.168.0.17 firma2.comp
sudo service apache2 restart
```
### Настройка apache
```
# Сертификаты:
openssl reg -new  
ls
cat server.key
cat privet.key
# вкл виртуального хоста:
sudo a2ensite default-ssl.conf
cd  /etc/apache2/sites-available/
sudo systemctl reload apache2 # перечитать конфиг
netstat -tlpn  # Проверка 443 порта
sudo systemctl restart apache2
```
### Уст nginx
```
sudo apt install nginx -y
systemctl status nginx
systemctl start nginx
systemctl status nginx
```
### Активация фаервола:
sudo ufw status
ip подставляем в браузер
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw allow 22
sudo ufw allow 'Apache Full'  # разрешает все 3 режима
sudo ufw enable
sudo ufw status
sudo ufw allow 'Nginx Full'
sudo ufw status
```
### Настройка nginx
```
vim /etc/nginx/nginx.conf
```
