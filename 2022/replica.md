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
vim /etc/postgresql/14/main/postgresql.conf
shared_buffers = 6GB
maintenance_work_mem = 1536MB
work_mem = 15728kB
max_connections = 1100
```
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
-----------------------------
vim /etc/postgresql/14/main/pg_hba.conf
host all all 192.168.0.0/24            trust
host postgres postgres 127.0.0.1/32 trust
systemctl restart postgresql
psql -p 5432 -h 192.168.0.14 -U postgres

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
psql -p 5432 -d otus -h 192.168.0.14 -U postgres
```
###### На реплике 
```
sudo -u postgres rm -rf /var/lib/postgresql/14/main/*
```

###### Восстановим cluster from master 
```
# Restor на реплике:
sudo -u postgres pg_basebackup --host=192.168.0.17 --port=5432 --username=replica --pgdata=/var/lib/postgresql/14/main/ --progress --write-recovery-conf --create-slot --slot=replica1
pass:
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
ls -la /var/lib/postgresql/14/main/ | grep standby
-rw-------  1 postgres postgres      0 янв 10 17:44 standby.signal
```
###### На slave
```
pg_ctlcluster 14 main start
pg_lsclusters
# Ответ: online,recovery
14  main    5432 online,recovery postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```
###### На master check replication slots:
```
su - postgres
psql
postgres=# select * from pg_replication_slots \gx
-[ RECORD 1 ]-------+----------
slot_name           | replica1
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | t
active_pid          | 104288
xmin                |
catalog_xmin        |
restart_lsn         | 0/B000148
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase           | f
```
```
 select * from pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 104288
usesysid         | 16384
usename          | replica
application_name | 14/main
client_addr      | 192.168.0.18
client_hostname  |
client_port      | 32982
backend_start    | 2023-01-10 17:58:18.20271+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/B000148
write_lsn        | 0/B000148
flush_lsn        | 0/B000148
replay_lsn       | 0/B000148
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-01-10 18:07:10.171855+03
```
###### На мастер: Создаем и заполнение новую таблицу :
```
su postgres
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test00(i int);"
psql otus -c "insert into test00 values (10), (20), (30);"
psql otus -c "select * from test;"
 i
----
 10
 20
 30
```
###### На SLAVE смотрим изменения:
```
psql otus -c "select * from test;"
 i
----
 10
 20
 30
```
#### Удаление слота репликации:
```
# Что есть сейчас:
postgres=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+-----------
slot_name           | replica1
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | f
active_pid          |
xmin                |
catalog_xmin        |
restart_lsn         | 0/6AD7D7E8
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase           | f
-----------------------------------
postgres=# select pg_drop_replication_slot('replica1');
ОШИБКА:  слот репликации "replica1" занят процессом с PID 1003
# Гасим реплику и повторяем
--------------------------------------------
postgres=# select pg_drop_replication_slot('replica1');
 pg_drop_replication_slot
--------------------------
(1 строка)
postgres=# SELECT * FROM pg_replication_slots \gx
(0 строк)
# Итог: нет слота репликации
------------------------------
# На реплике:
sudo -u postgres pg_basebackup --host=192.168.0.17 --port=5432 --username=replica --pgdata=/var/lib/postgresql/14/main/ --progress --write-recovery-conf --create-slot --slot=replica1

Пароль:
248272/248272 КБ (100%), табличное пространство 1/1
---------------------------
# На Мастере:
postgres=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+-----------
slot_name           | replica1
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | f
active_pid          |
xmin                |
catalog_xmin        |
restart_lsn         | 0/89000000
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase  
# Слот репликации опять появился.
```

##### Генерация милионов записей
```
\c otus
CREATE TABLE test111(i int);
INSERT INTO test111 SELECT s.id FROM generate_series(1,1000000000) AS s(id);
sudo apr inatall pgtop
sudu -u postgres pg_top
# Q - текст запроса; E - план; L - блокировки; 
```










