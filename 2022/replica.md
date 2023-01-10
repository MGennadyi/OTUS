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
---------------
echo "archive_mode = on" >>  /etc/postgresql/14/main/postgresql.conf
echo "archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'" >>  /etc/postgresql/14/main/postgresql.conf
# echo "archive_cleanup_command = 'pg_archivecleanup /archive %r'" >>  /etc/postgresql/14/main/postgresql.conf
# echo "restore_command = 'cp /archive/%f %p'" >>  /etc/postgresql/14/main/postgresql.conf


echo "host replication replica 0.0.0.0/0 md5" >> /etc/postgresql/14/main/pg_hba.conf
echo "host all rewind 0.0.0.0/0 md5" >> /etc/postgresql/14/main/pg_hba.conf


mkdir /archive
chown -R postgres:postgres /archive
```
```
create replica and rewind users with password 12345
sudo -u postgres psql -c "create user replica with replication encrypted password '12345'"
sudo -u postgres psql -c "CREATE USER rewind SUPERUSER encrypted PASSWORD '12345'"

```




























