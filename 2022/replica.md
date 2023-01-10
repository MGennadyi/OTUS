# Физическая репликация

###### Уст. postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt update
apt install postgresql-14 -y
apt install net-tools -y
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
###### На обоих нодах:
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
###### Применим изменения на всех нодах:
```
pg_ctlcluster 14 main status
pg_ctlcluster 14 main stop
pg_ctlcluster 14 main start
```
```
create replica and rewind users with password 12345  - на видео пропускает
sudo -u postgres psql -c "create user replica with replication encrypted password '12345'"
sudo -u postgres psql -c "CREATE USER rewind SUPERUSER encrypted PASSWORD '12345'"
```
```
sudo -u postgres psql -c "CREATE DATABASE otus"
sudo -u postgres pgbench -i -s 10 otus
```
###### Проверка доступности себя и соседа:
```
# Через nc
nc -vz 192.168.0.18 5432
nc -vz 192.168.0.17 5432
# Через netstat по портам:
apt install net-tools -y
netstat -nlp | grep 5432
```
###### На реплике 
```
sudo -u postgres rm -rf /var/lib/postgresql/14/main/*
```

###### Восстановим cluster from master (it will ask for 123 password of replica user, also note that it can take some time to backup restore 500mb)
```
# Restor на реплике:
sudo -u postgres pg_basebackup --host=192.168.0.17 --port=5432 --username=replica --pgdata=/var/lib/postgresql/14/main/ --progress --write-recovery-conf --create-slot --slot=replica1
# Ответ: 188126/188126 КБ (100%), табличное пространство 1/1  -синхронизация прошла успешно.
# В ответ на сообщение реплики "waiting  checkpoint", на мастере:
sudo -u postgres psql -c "checkpoint"
```
###### Проверим, что изменилось:
```
# На slave: 
cat /var/lib/postgresql/14/main/postgresql.auto.conf
primary_conninfo = 'user=replica password=12345 channel_binding=prefer host=192.168.0.17 port=5432 sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
primary_slot_name = 'replica1'
# Т.О. на MASTER создался слот репликации, а на SLAVE прописались параметры подключения.
```
###### Проверка создания standby.signal, который переводит режим работы ноды в slave: 
```
ls -la /var/lib/postgresql/12/main/ | grep standby
```
###### На slave
```
pg_ctlcluster 14 main start
pg_lsclusters
```
###### На master check replication slots:
```
sudo -u postgres psql -c "select * from pg_replication_slots"
sudo -u postgres psql -c "select * from pg_stat_replication"
```




















