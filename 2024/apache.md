# Установка Apache
### 1. Уст сетевых утилит:
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
```
ifconfig      # ip подставляем в браузер
url localhost # Просмотр кода стартовой страницы
```
### Уст Apache2 на Ubuntu
```
sudo apt install apache2 -y
systemctl status apache2
/var/www/html - страница заглушки
```
### Прверка Debian
```
vim /etc/apache2/apache2.conf   # в самом конце должно:
IncludeOptional sites-enabled/*.conf
```
### Уст Apache2 Centos
```
yum install -y httpd
systemctl enable httpd
systemctl start httpd
netstat -tulnp | grep httpd
tcp6       0      0 :::80                   :::*                    LISTEN      3530/httpd
```
### Проверка Centos:
```
vim /etc/httpd/conf/httpd.conf # в самом конце должно:
IncludeOptional conf.d/*.conf
NameVirtualHost *:80  # в начале
```
### Создание страницы site1
```
mkdir /web
mkdir -p /web/site1
mkdir -p /web/site1/log
mkdir -p /web/site2
mkdir -p /web/site2/log
chown -R www-data:www-data /web/ - DEBIAN
chown -R apache:apache /web/     - CENTOS
sudo chmod -R 770 /web/
vim /web/site1/index.html
<html>
<head>
<title>Welcome to SITE1!</title>
</head>
<body>
<h1>Success! </h1>
</body>
</html>
```
### Создание страницы site2
```
mkdir -p /web/site2
mkdir -p /web/site2/log
# Здесь все по другому:
chown -R apache:apache /web/
sudo chmod -R 770 /web/
vim /web/site2/index.html
<html>
<head>
<title>Welcome to SITE2!</title>
</head>
<body>
<h1>Success! </h1>
</body>
</html>
```
### Debian: Созд. конфига вирт.хостов для каждого домена/поддомена. site1:
```
vim /etc/apache2/sites-available/site1.conf
<VirtualHost *:80>
DocumentRoot /web/site1
ServerName site1
ServerAlias www.site1

<Directory /web/site1/>
Options FollowSymLinks
AllowOverride All
Order allow,deny
allow from all

</Directory>
ErrorLog /web/site1/log/site1-error.log
CustomLog /web/site1/log/site1-access.log common
</VirtualHost>
```
### Debian: Созд. вирт.хостов для каждого домена/поддомена domain2.ru:
```
vim /etc/apache2/sites-available/site2.conf
<VirtualHost *:80>
DocumentRoot /web/site2
ServerName site2
ServerAlias www.site2

# enter other directives here, e.g. :

<Directory /web/site2/>
Options FollowSymLinks
AllowOverride All
Order allow,deny
allow from all

</Directory>
ErrorLog /web/site2/log/site2-error.log
CustomLog /web/site2/log/site2-access.log common
</VirtualHost>
```
### Centos: Созд. вирт.хостов для каждого домена/поддомена. domain1.ru и domain2.ru:
```
vim /etc/httpd/conf.d/site1.conf
<VirtualHost *:80>
 ServerName site1
 ServerAlias www.site1
 DocumentRoot /web/site1
 <Directory /web/site1>
 Options FollowSymLinks
 AllowOverride All
 Require all granted
 </Directory>

 ErrorLog /web/site1/log/error.log
 CustomLog /web/site1/log/access.log common
</VirtualHost>
```

### 
```
sudo mkdir /etc/httpd/sites-available
sudo mkdir /etc/httpd/sites-enabled
chown -R apache:apache /etc/httpd/sites-available
chown -R apache:apache /etc/httpd/sites-enabled
sudo chmod -R 777 /etc/httpd/sites-available
sudo chmod -R 777 /etc/httpd/sites-enabled

vim /etc/httpd/sites-available/site1.conf
<VirtualHost *:80>
ServerName www.site1
ServerAlias site1
DocumentRoot /web/site1
ErrorLog /web/site1/log/error.log
CustomLog /web/site1/log/requests.log combined
</VirtualHost>

apachectl restart
```
### Index.html обратитесь к серверу по имени домена (при этом А-запись домена должна быть настроена и указывать на ваш веб-сервер).
```
touch /var/www/site1.loc/index.html
<body><h1>OK!</h1></body>
```
### Замена дефолтной страницы:
```
cp /var/www/html/index.html /var/www/html/index_0.html
> /var/www/html/index.html # Дятел для очистки
vim /var/www/html/index.html
<html>
<head>
<title>Welcome to SITE-1!</title>
</head>
<body>
<h1>Success! The virtual host is working!</h1>
</body>
</html>
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
```
### Debian Привязка конфига сайта к серверу Apache2 через a2ensite:
```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
a2ensite site1
a2ensite site2
a2ensite test.loc
a2ensite demo.loc
# Деактивируем дефолтный конфиг.
sudo a2dissite 000-default.conf
systemctl reload apache2
```
### Centos Привязка конфига сайта к серверу Apache2 через симлинк ln:
```
sudo ln -s /etc/httpd/sites-available/site1.conf /etc/httpd/sites-enabled/site1.conf
sudo ln -s /etc/httpd/sites-available/site2.conf /etc/httpd/sites-enabled/site2.conf
```
### html 1
```
vim /web/site1/index.html
<html>
<head>
<title>Welcome to SITE-1!</title>
</head>
<body>
<h1>Success! The virtual host is working!</h1>
</body>
</html>
```
### HTML2
```
vim /web/site2/index.html
<html>
<head>
<title>Welcome to SITE-2!</title>
</head>
<body>
<h1>Success! The virtual host is working!</h1>
</body>
</html>
```
### Debian Для установки правильных разрешений:
```
chown www-data:www-data -R /web/*
sudo chown -R www-data:www-data /var/www/site1.loc
```
### CentOS права:
```
chown apache:apache -R /web/*
```
### Настройка
```
vim /etc/apache2/ports.conf -80 порт открыт
sudo chmod -R 770 /web
vim /etc/hosts
127.0.0.1 www.example1.com
127.0.0.1 www.example2.com
192.168.0.17 site1
192.168.0.17 site2
sudo service apache2 restart
w3m www.site1.loc - проверка
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
```
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
```
192.168.0.141   site1
192.168.0.141   site2
grep -inRH "ServerName " /etc/apache2/ | grep "\s#"*
vim /etc/apache2/apache2.conf 
ServerName localhost

           Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted

 systemctl restart apache2
```
```
ping site1
ping site2
mkdir /var/www/html/site1
mkdir /var/www/html/site2
sudo chmod -R 777 /var/www/html/site1
sudo chmod -R 777 /var/www/html/site2
```
### Уст php
```
apt-get install php
root@ubun:/etc/apache2/sites-available# php --version
PHP 8.1.2-1ubuntu2.14 (cli) (built: Aug 18 2023 11:41:11) (NTS)
apt-get install php-gd
apt-get install imagemagick php-imagick -y
service apache2 reload
```
### Проверка php:
```
vim /var/www/examlpe.com/info.php
<?php
phpinfo();
?>
```
```
http://example.com/info.php
```















