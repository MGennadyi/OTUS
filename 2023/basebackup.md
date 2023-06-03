# BASEBACKUP
#### Подготовка директорий:
```
mkdir -p /backup
chown -R postgres:postgres /backup
mkdir -p /postgres
chown -R postgres:postgres /postgres
mkdir -p /log
chown -R postgres:postgres /log
mkdir -p /wal
chown -R postgres:postgres /wal
mkdir -p /tempdb
chown -R postgres:postgres /tempdb
vim /etc/fstab
tmpfs /tempdb tmpfs size=500M,uid=postgres,gid=postgres 0 0
mount /tempdb
vim /postgres/scripts/atom_basebackup.sh
chmod +x /postgres/scripts/atom_basebackup.sh
```
```
sudo -i -u postgres
mkdir -p /backup/SRK
mkdir -p /postgres/scripts
mkdir -p /log/pg_log
mkdir -p /log/llog  # Для ротирования логов
mkdir -p /wal/pg_wal
mkdir -p /backup/wal_arc_archive

```
```
ALTER SYSTEM SET log_directory = '/log/pg_log';
ALTER SYSTEM SET log_filename = 'postgresql-%u.log';
ALTER SYSTEM SET log_filename = 'postgresql.log';
ALTER SYSTEM SET log_filename = 'postgresql-%u.log';
ALTER SYSTEM SET logging_collector = 'on';
ALTER SYSTEM SET wal_compression = 'on';
ALTER SYSTEM SET stats_temp_directory = '/tempdb';
ALTER SYSTEM SET archive_mode = 'on';
ALTER SYSTEM SET archive_timeout = '600';
ALTER SYSTEM SET archive_command = 'test ! -f /backup/wal_arc_archive/%f && cp %p /backup/wal_arc_archive/%f';
ALTER USER postgres WITH PASSWORD '12345';
CREATE USER expert WITH PASSWORD '12345';  user-с правом входа
CREATE USER evsemkin LOGIN password '12345';
CREATE USER avesenin LOGIN password '12345';
ALTER USER avesenin WITH LOGIN password '12345';
psql -c "CREATE DATABASE demo owner expert;"
SELECT pg_reload_conf();
SELECT pg_switch_wal();
```
#### Создание групповой роли create_arwd_group.sh на БД demo;
```
# Правим строку пути к бинарникамиЭ
vim /postgres/scripts/create_group.config
bin_path=/usr/lib/postgresql/14/bin
# Вешаем роль на бд demo:
bash create_arwd_group.sh --list demo
postgres=# \l+ demo
                                                      Список баз данных
 Имя  | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |      Права доступа      | Размер  | Табл. пространство | Описание
------+----------+-----------+-------------+-------------+-------------------------+---------+--------------------+----------
 demo | expert   | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =Tc/expert             +| 8577 kB | pg_default         |
      |          |           |             |             | expert=CTc/expert      +|         |                    |
      |          |           |             |             | arwd_role_demo=c/expert |         |                    |
postgres=# grant readonly_role_demo to avesenin;
GRANT ROLE
postgres=# grant arwd_role_demo to evsemkin;
GRANT ROLE

```
#### СКРИПТ basebackup.sh
```
#!/bin/bash
# скрипт делает basebackup СУБД
# Функционал:
# - его можно включить в crontab
# - можно наблюдать за процессом
# - есть лог выполнения с "прогрессом" (процентами выполнения)

version=0.2
LOG_FILE=/postgres/scripts/atom_basebackup.log
# Директория сжатого архива данных:
backup=/backup/SRK/"$(date '+%Y_%m_%d')"

mkdir -p $backup  # -p не выдает ошибку, если такой каталог уже существует

echo "НАЧАЛО: "$(date '+%d-%m-%Y_%H:%M:%S') >> $LOG_FILE
pg_basebackup --format=tar --gzip -U postgres --checkpoint=fast --progress --pgdata=$backup 2>&1 | tee &>> $LOG_FILE

# Для создания реплики, которая потом запустится
# pg_basebackup -h 1trck-s-psql03 --format=plain -U postgres --checkpoint=fast --progress --pgdata="/data/pg_data" 2>&1 | tee &>> $LOG_FILE

echo "КОНЕЦ: "$(date '+%d-%m-%Y_%H:%M:%S') >> $LOG_FILE
exit 0

# Наблюдать за процессом
watch -n 1 "ps ax | grep atom_basebackup | grep -v grep; tail -n 20 /postgres/scripts/atom_basebackup.log; df /data; df -h /data; du -s /backup/*"

# Инсталляция
#vim /postgres/scripts/atom_basebackup.sh
#chmod 0770 /postgres/scripts/atom_basebackup.sh  #или:
#chmod +x /postgres/scripts/atom_basebackup.sh
#chown postgres:postgres /postgres/scripts/atom_basebackup.sh

# Установить задачу atom_basebackup.sh в crontab -e
# m h  dom mon dow   command
# 00 21 21  *   *     /postgres/scripts/atom_basebackup.sh
```
###### Перенос данных в паралельный кластер:
```
# 1. Создание паралельного кластера:
pg_createcluster 14 main2
pg_lsclusters
# main2 создан, но не запущен.
# 2. Удаляем данные из main2:
rm -rf /var/lib/postgresql/14/main2
# 3. Перенос данных в main2:
pg_basebackup -p 5432 --pgdata=/var/lib/postgresql/14/main2
# 4. Стуртуем кластер main2:
pg_ctlcluster 14 main2 start # Ругается, но стартует:
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
systemctl start postgresql@14-main2
# 5. Смотрим как стартовал:
pg_lsclusters # или
systemctl status postgresql@14-main2
# 5. Смотрим в main2:
psql -p 5433
otus=# select * from test;
 i
----
 10
 20
 30
(3 строки)
# 6. Заполняем в main2 БД otus тестовой инф-ии:
\c otus
CREATE table test_main2(i int);
INSERT INTO test_main2 values (1), (2), (3), (10), (20), (30);
select * from test_main2;
 i
----
  1
  2
  3
 10
 20
 30
(6 строк)
```
### Логический перенос содержимого БД otus из кластера main2 в БД otus_main кластера main:
```
# 7. Создаем место куда переносим данные:
psql -p 5432
CREATE DATABASE otus_main;

CREATE DATABASE otus_main owner expert;
\q
date && pg_dump -p 5433 otus | psql -p 5432 --set ON_ERROR_STOP=on otus_main >> /backup/restore_db.log 2>&1 && date 
cat /backup/restore_db.log
psql -p 5432
postgres=# \c otus_main
select * from test_main2;
 i
----
  1
  2
  3
 10
 20
 30
(6 строк)
```
#### Работа с wal
```
archive_command = 'test ! -f /backup/wal_arc_archive/%f && cp %p /backup/wal_arc_archive/%f'
```
```
CREATE DATABASE ttt;
ALTER DATABASE ttt RENAME TO qwerty;

```
### Перенос DEMO_SMALL.sql demo_big.sql
```
wget --quiet https://edu.postgrespro.ru/demo-big.zip
chown postgres:postgres /home/mgb/demo-big.zip
unzip demo-big.zip -d /backup
mv demo-big-20170815.sql demo_big.sql
chown postgres:postgres /backup/demo_big.sql
sudo wget --quiet https://edu.postgrespro.ru/demo_small.zip
chown postgres:postgres /home/mgb/demo_small.zip
unzip demo_small.zip
chown postgres:postgres /home/mgb/demo_small.sql
chown postgres /home/mgb/demo_small.sql
```
#### Предварительная загрузка данных:
```
sudo -i -u postgres
psql -p 5433 -c 'drop DATABASE demo';  
psql -p 5433 -c 'CREATE DATABASE demo owner expert'; -указать OWNER !!!
# Заливка исходных данных demo_small.sql demo_big.sql:
# time psql -p 5433 -d demo < /backup/demo_small.sql
time psql -p 5433 -d demo < /backup/demo_big.sql
time psql -p 5432 -d demo < /backup/demo_big.sql
real    3m6,641s
real    3m46,939s
user    0m6,404s
sys     0m1,253s
root@backup-restore:/backup# df -h
Файловая система Размер Использовано  Дост Использовано% Cмонтировано в
udev               2,4G            0  2,4G            0% /dev
tmpfs              477M         980K  476M            1% /run
/dev/sda1           16G          11G  4,0G           73% /
df -h /var/lib/postgresql/14/main2/pg_wal
# Выгрузка на скорость
time pg_dump -C -p 5433 -h localhost -U postgres 'demo' > /backup/demo_main.sql
real    0m19,442s
postgres@backup-restore:~$ time pg_dump -d demo > /backup/demo_main.sql
real    0m1,801s
user    0m0,417s
sys     0m0,470s
postgres@backup-restore:~$ ls -la /backup
итого 202868
drwxr-xr-x  4 postgres postgres      4096 мая 25 14:34 .
drwxr-xr-x 24 root     root          4096 мая 19 19:18 ..
-rw-r--r--  1 postgres postgres 103857451 мая 25 14:34 demo_main.sql
-rw-rw-r--  1 postgres postgres 103857328 ноя 11  2016 demo_small.sql
```
```
create database demo_main2;
time psql -p 5433 -d demo_main2 < /backup/demo_main.sql
real    0m18,920s
```
###### Тест psq: Загрузка/выгрузка 1,8/18,9 сек. 186s/19,442s=9.8 раз
```
postgres@backup-restore:~$ time pg_dump -C -h localhost -U postgres 'demo' > /backup/demo_main.sql
Пароль:
real    0m4,585s
user    0m0,585s
sys     0m0,318s
time psql -p 5433 -U postgres < /backup/demo_main.sql > /backup/dump_in.log 2>&1
```
#### Тест -Fd SIZE=888/233MB=3.8 раза 220/34сек=6.5 34/18=1.8 220/186=1.2
```
# Филатов Евгений утверждал pg_restore медленнее -?? 
psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity"
psql -p 5432 -c 'DROP DATABASE demo';    -указать OWNER !!!
psql -p 5432 -c 'CREATE DATABASE demo';
time pg_dump -p 5433 -C -h localhost -U postgres -j 4 -d demo -Fd -f /backup/dump
Пароль:
real    0m34,332s
u -sh /backup/dump
233M    /backup/dump
postgres@backup-restore:/postgres/scripts$ time pg_restore -p 5432 -h localhost -j 4 -d demo /backup/dump
Пароль:
real    3m40,516s
```
#### Тест пайплан demo_big: 185 сек/186+19=205 сек. Пайплан чуть быстрее.
```

time pg_dump -p 5432 demo | psql -p 5433 --set ON_ERROR_STOP=on demo
real    3m5,470s
```
#### BASH-скрипт PGDUMP = 23 сек:
```
vim /postgres/scripts/dump.sh
#!/bin/bash
# скрипт делает dummp БД
pg_dump -C -h localhost -U postgres 'demo' > /backup/demo_main.sql && psql -p 5433 -U postgres < /backup/demo_main.sql
real    0m23,693s
```
```
time pg_dump -C -h localhost -U postgres -d demo -Fd > /backup/dump
time pg_dump -C -h localhost -U postgres -j 4 -d demo -Fd -f /backup/dump
# Восстановление в 1 поток:
postgres@backup-restore:~$ time pg_restore -p 5433 -j 1 -d newdb /backup/dump
real    0m19,033s
```
```
postgres=# drop database newdb;
DROP DATABASE
postgres=# create database newdb;
CREATE DATABASE
```
### BASH-скрипт: DUMP-RESTORE -Fd = 25 сек:
```
#!/bin/bash
# скрипт делает dummp БД
rm -rf /backup/dump/*
psql -c "DROP DATABASE demo;"
psql -c "CREATE DATABASE demo owner expert;"    -указать OWNER !!!
pg_dump -p 5432 -C -h localhost -U postgres -j 4 -d demo -Fd -f /backup/dump && pg_restore -p 5433 -h localhost -j 4 -d demo /backup/dump
---------------------
# Восстановление: 1. Создать БД. 2. Восстановить туда данные
psql -c "CREATE DATABASE demo owner expert;"  -указать OWNER !!!
pg_restore -p 5432 -h localhost -j 4 -d demo /backup/dump
real    3m12,585s

postgres@backup-restore:~$ time /postgres/scripts/dump_d.sh
Пароль:
Пароль:
real    0m25,629s
user    0m5,457s
sys     0m0,244s
```
### SSH
```
vim /etc/ssh/sshd_config
PasswordAuthentication yes
PermitRootLogin yes
sudo systemctl restart sshd
# Первый раз лучше зайти, чтоб прописались ключи: 
ssh 192.168.0.19
# Копирование директории:
time scp -r /backup/dump root@192.168.0.16:/backup
time scp -r /backup/dump root@192.168.0.19:/backup
# Копирование содержимого директории:
time scp /backup/dump/* root@192.168.0.19:/backup/dump
time scp /backup/dump/* root@192.168.0.19:/backup/dump
```
```
postgres@backup-restore:~$ time scp -r /backup/dump root@192.168.0.17:/backup
root@192.168.0.17's password:
3402.dat.gz                                                                                                                                                               100%   69MB  66.5MB/s   00:01
3403.dat.gz                                                                                                                                                               100%   87MB  47.8MB/s   00:01
3395.dat.gz                                                                                                                                                               100%  321   293.2KB/s   00:00
3401.dat.gz                                                                                                                                                               100% 3047     2.2MB/s   00:00
3397.dat.gz                                                                                                                                                               100%   53MB  48.5MB/s   00:01
toc.dat                                                                                                                                                                   100%   40KB  21.9MB/s   00:00
3398.dat.gz                                                                                                                                                               100%   20MB  45.2MB/s   00:00
3399.dat.gz                                                                                                                                                               100% 3319KB  47.6MB/s   00:00
3396.dat.gz                                                                                                                                                               100% 5682     5.8MB/s   00:00

real    0m8,192s
user    0m1,076s
sys     0m1,915s


```
#### Выгрузка ролей:
```
pg_restore -p 5432 -h localhost -j 4 -d demo /backup/dump
pg_dumpall -p 5432 -h localhost --globals-only > /backup/roles_and_users.sql
```
