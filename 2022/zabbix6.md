# ZABBIX 6.1.

###### 0. Правим $PATH
```
vim /root/.bashrc
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
sudo shutdown -r now
```
###### 1. Установка Postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
apt install postgresql-14 -y
```
##### 2. Установка ZABBIX 6
```
wget https://repo.zabbix.com/zabbix/6.1/debian/pool/main/z/zabbix-release/zabbix-release_6.1-2%2Bdebian11_all.deb
dpkg -i zabbix-release_6.1-2+debian11_all.deb
apt update
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent -y
```
```
su postgres
psql
CREATE DATABASE zabbix;
CREATE USER zabbix WITH PASSWORD '12345';
```
###### Импортируем схему:
```
zcat /usr/share/doc/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
vim /etc/zabbix/zabbix_server.conf
```
###### DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=12345
```
systemctl start zabbix-server
systemctl enable zabbix-server
```




























