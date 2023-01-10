# Физическая репликация

###### Уст. postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt update
apt install postgresql-14 -y
# Проверяем:
pg_isready
# Ответ: /var/run/postgresql:5432 - принимает подключения
pg_ctlcluster 14 main status
# Ответ:
pg_ctl: сервер работает (PID: 5691)
/usr/lib/postgresql/14/bin/postgres "-D" "/var/lib/postgresql/14/main" "-c" "config_file=/etc/postgresql/14/main/postgresql.conf"
pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
````
```
# По умолчанию: listen_addresses = 'localhost' #wal_log_hints = off
echo "listen_addresses = '*'" >> /etc/postgresql/14/main/postgresql.conf
echo "wal_log_hints = on" >> /etc/postgresql/14/main/postgresql.conf


```





























