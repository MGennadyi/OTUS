# Физическая репликация
### Подготовить 2 хоста
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
show huge_pages; = try;
show wal_level; = replica;
show synchronous_commit;
on
show wal_log_hints;
```
#### Конфигурируем:
```
# По умолчанию: listen_addresses = 'localhost' #wal_log_hints = off
ALTER SYSTEM SET listen_addresses = '0.0.0.0/0';
ALTER SYSTEM SET wal_log_hints = 'on';
# ALTER SYSTEM SET archive_command = 'gzip < %p > /backup/wal_arc_archive/%f.gz'
ALTER SYSTEM SET archive_command = 'pg_compresslog %p - | gzip > /backup/wal_arc_archive/%f.gz'
show listen_addresses;  # по умолчанию =  localhost. Подключиться из вне не получится!
ALTER SYSTEM SET listen_addresses = '*';   # restart service   !!!!!
ALTER SYSTEM SET listen_addresses = '0.0.0.0';   # restart service   !!!!!
------------------
На master создать пользователя, от имени которого будет реплицироваться БД, предоставить права подключения:
# host all rewind 0.0.0.0/0 md5
# host postgres postgres 127.0.0.1/32 trust
ALTER USER postgres WITH PASSWORD '12345';
CREATE USER replica with replication encrypted password '12345';  # на MASTER, на реплике все удалится.
CREATE USER rewind SUPERUSER encrypted PASSWORD '12345';  # не делал
-----------------------------
mcedit /var/lib/pgpro/std-15/data/pg_hba.conf
mcedit /etc/postgresql/14/main/pg_hba.conf
host postgres postgres 127.0.0.1/32 trust  # Проверить
host replication replica 192.168.0.16/32 md5
host all postgres 192.168.0.16/32 md5
systemctl restart postgresql
psql -p 5432 -h 192.168.0.17 -U postgres
```
###### Применим изменения на всех нодах:
```
pg_ctlcluster 14 main status
pg_ctlcluster 14 main stop
pg_ctlcluster 14 main start
systemctl status postgrespro-std-15
systemctl stop postgrespro-std-15
systemctl start postgrespro-std-15
```
### На master создать пользователя, от имени которого будет реплицироваться БД, предоставить права подключения, перечитать конфиг:
```
ALTER USER postgres WITH PASSWORD '12345';
create replica and rewind users with password 12345  - на видео пропускает
CREATE USER replica with replication encrypted password '12345';  # на MASTER, на реплике все удалится.
"CREATE USER rewind SUPERUSER encrypted PASSWORD '12345';  # не делал
```
###### Проверка доступности себя и соседа:
```
# Через nc
sudo -i -u postgres
nc -vz 192.168.0.18 5432
nc -vz 192.168.0.17 5432
# Ответ:
master [192.168.0.17] 5432 (postgresql) open
nc -vz 192.168.0.16 5432
# Через netstat по портам:
apt install net-tools -y
netstat -nlp | grep 5432
psql -p 5432 -d otus -h 192.168.0.17 -U postgres
```
### Заполнение тестовыми данными на master:
```
psql -p 5432
CREATE DATABASE otus;
\c otus
CREATE table test(i int);
INSERT INTO test values (1), (2), (3);
select * from test;
INSERT INTO test values (4);
```
###### На реплике удаляем содержимое pg_data:
```
systemctl stop postgrespro-std-14
rm -rf /var/lib/postgresql/14/main/*  # V_14
systemctl stop postgrespro-std-15
rm -rf /var/lib/pgpro/std-15/data/*   # V_15
```
###### Создание реплики from master=192.168.0.17: 
```
# Restor на реплике=192.168.0.16:
sudo -i -u postgres
pg_basebackup --host=192.168.0.17 --port=5432 --username=replica --pgdata=/var/lib/postgresql/14/main/ --progress --write-recovery-conf --create-slot --slot=replica1
sudo -i -u postgres
pg_basebackup --host=192.168.0.17 --port=5432 --username=replica --pgdata=/var/lib/pgpro/std-14/data/ --progress --write-recovery-conf --create-slot --slot=replica1
Вводим pass:
# Ответ: 188126/188126 КБ (100%), табличное пространство 1/1  -синхронизация прошла успешно.
# В ответ на сообщение реплики "waiting  checkpoint", на мастере:
sudo -u postgres psql -c "checkpoint"
cat /var/lib/pgpro/std-14/data/postgresql.auto.conf
primary_conninfo = 'user=replica password=12345 channel_binding=prefer host=192.168.0.17 port=5432 sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'

```
#### На реплике V_15:
```
systemctl start postgrespro-std-14
systemctl start postgrespro-std-15
/c otus
otus=# select * from test;
 i
---
 1
 2
 3
(3 строки)
```
###### Проверим, что изменилось V_14:
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
###### На slave V_14:
```
pg_ctlcluster 14 main start
pg_lsclusters
# Ответ: online,recovery
14  main    5432 online,recovery postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```
###### На master check replication slots:
```
postgres=# select * from pg_replication_slots \gx
-[ RECORD 1 ]-------+----------
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
restart_lsn         | 0/C000000
confirmed_flush_lsn |
wal_status          | reserved
safe_wal_size       |
two_phase           | f
```
```
 postgres=# select * from pg_stat_replication \gx
-[ RECORD 1 ]----+------------------------------
pid              | 5673
usesysid         | 16392
usename          | replica
application_name | walreceiver
client_addr      | 192.168.0.16
client_hostname  |
client_port      | 39674
backend_start    | 2023-06-10 16:52:55.103514+03
backend_xmin     |
state            | streaming
sent_lsn         | 0/E000060
write_lsn        | 0/E000060
flush_lsn        | 0/E000060
replay_lsn       | 0/E000060
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2023-06-10 16:59:59.824104+03
```
###### На мастер: Создаем и заполнение новую таблицу :
```
su postgres
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test(i int);"
insert into test values (10), (20), (30);
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
 insert into test values (4);
 в транзакции в режиме "только чтение" нельзя выполнить INSERT
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
# Гасим реплику
systemctl stop postgrespro-std-15
# и повторяем
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
# С другого хоста:
psql -p 5432 -d otus -h 192.168.0.16 -U postgres
CREATE DATABASE otus;
\c otus
CREATE TABLE test111(i int);
INSERT INTO test111 SELECT s.id FROM generate_series(1,1000000000) AS s(id);
sudo apr inatall pgtop
sudu -u postgres pg_top
# Q - текст запроса; E - план; L - блокировки; 
```
### Перевод реплики в stand by режим
```
pg_ctl -w -D  /var/lib/postgresql/14/main promoute
pg_ctl -w -D /var/lib/pgpro/std-15/data/ promote
select pg_promote();
```
### V_15 установится на 5432, поэтому
```
vim /var/lib/pgpro/std-14/data/postgresql.conf




```







```
sudo -u postgres pgbench -i -s 10 otus
# Протестить команду:
# select * from pg_is_in_recovery; - не работает
```









