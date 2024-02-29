# Установка Apache
### 1. Подмена встроенного web-сайта на свой собственный
```
apt install net-tools
netstat -tlpn
Active Internet connections (only servers)
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
```
root@wal-g2:/home/mgb# curl localhost
curl: (7) Failed to connect to localhost port 80: В соединении отказано
```
### Уст ubuntu
```
# Проверка, что не установлен
ifconfig
ip подставляем в браузер
sudo apt install apache2 -y
systemctl status apache2
curl localhost
ip подставляем в браузер = стартовая страница apache2
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
###
```
sudo service apache stop
ubuntu.com/сохранить как/index.html/обновить - не сработает.
sudo mc
/var/www/html/index.html - переименовать в _index.html
вставить из загрузок сохраненную ubuntu
sudo service apache start
```
###
```
firma1.comp firma2.comp -создал папки
sudo thunar  # файловый менеджер. Из загрузок копировал 2 сохраненнных сайта от авито и microsoft
/var/www/firma1.comp/index.html
/var/www/firma2.comp/index.html
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/firma1.comp.conf
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/firma2.comp.conf
mcedit /etc/apache2/sites-available/firma1.comp.conf
Servername firma1.comp            -добавить
DocumentRoot /var/www/firma1.comp -изменить
mcedit /etc/apache2/sites-available/firma2.comp.conf
Servername firma2.comp            -добавить
DocumentRoot /var/www/firma2.comp -изменить
# Привязка конфига сайта к серверу apache2
a2ensite firma1.comp
a2ensite firma2.comp
```
###
```
vim /etc/hosts
192.168.0.17 firma1.comp
192.168.0.17 firma2.comp

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
