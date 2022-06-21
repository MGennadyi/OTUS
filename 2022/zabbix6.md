# ZABBIX 6.1 на POSTGRESQL 14.

###### 0. Правим $PATH
```
vim /root/.bashrc
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
addgroup --system --quiet zabbix
adduser --quiet --system --disabled-login --ingroup zabbix --home /var/lib/zabbix --no-create-home zabbix
sudo shutdown -r now
```
###### 1. Установка Postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
apt install postgresql-14 -y
systemctl status postgresql
psql -V
locale -a | grep ru
```
##### 2. Установка ZABBIX 6
```
wget https://repo.zabbix.com/zabbix/6.1/debian/pool/main/z/zabbix-release/zabbix-release_6.1-2%2Bdebian11_all.deb
dpkg -i zabbix-release_6.1-2+debian11_all.deb
apt update
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent jq sysstat -y
или
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent jq sysstat -y
```
###### When installation is complete, verify the Zabbix server installed:
```
apt-cache policy zabbix-server-pgsql
```
##### 3. Создание базы и пользователя с паролем (по док-и из template0):
```
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix -T template0 zabbix
или
su postgres
psql
CREATE DATABASE zabbix;
CREATE USER zabbix WITH PASSWORD '12345';
GRANT ALL PRIVILEGES ON DATABASE zabbix to zabbix;
```
###### 4. Импортируем схему БД:
```
zcat /usr/share/doc/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```
##### 5. Настройка конфига zabbix_server:
```
vim /etc/postgresql/14/main/pg_hba.conf
host    zabbix          zabbix          127.0.0.1/32            md5
```
```
vim /etc/zabbix/zabbix_server.conf
DBHost=localhost
ListenPort=10051
DBHost=localhost
DBPassword=12345
DBName=zabbix
DBUser=zabbix
DBPassword=12345
DBPort=5432
```
```
systemctl stop zabbix-server
systemctl start zabbix-server
systemctl enable zabbix-server
systemctl status zabbix-server
systemctl status zabbix-agent
systemctl status apache2
```
###### Проверка версии:
```
dpkg -l | grep zabbix
```
##### 6. Настроим Nginx, если установлено: раскомментируем и настроим две верхние позиции- нет таких директорий:
```
vim /etc/zabbix/nginx.conf !
vim /etc/nginx/conf.d/zabbix.conf
        listen          80;
        server_name     192.168.0.17;
vim  /etc/httpd/conf.d/zabbix.conf
? или
vim /etc/zabbix/php-fpm.conf
php_value[date.timezone] = Europe/Moscow

systemctl restart nginx
systemctl restart zabbix-server zabbix-agent nginx php7.2-fpm
systemctl enable zabbix-server zabbix-agent nginx php7.2-fpm
```

##### 7. Настройка через web-интерфейс:
```
http://192.168.5.161/zabbix/setup.php
Default language=RU
login=Admin
password=zabbix
```
##### Агент
```
apt-get purge --auto-remove zabbix-agent
wget https://repo.zabbix.com/zabbix/6.1/debian/pool/main/z/zabbix/zabbix-agent2_6.2.0~alpha2-1%2Bdebian11_amd64.deb
dpkg -i zabbix-agent2_6.2.0~alpha2-1+debian11_amd64.deb

```

























