# ZABBIX 6.1.

###### 1. Установка Postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
apt install postgresql-14 -y
```
##### 2. уст. ZABBIX
```
wget https://repo.zabbix.com/zabbix/6.1/debian/pool/main/z/zabbix-release/zabbix-release_6.1-1%2Bdebian11_all.deb
dpkg -i zabbix-release_6.0-1+debian11_all.deb
apt update
```











