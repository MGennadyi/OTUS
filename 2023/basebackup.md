# BASEBACKUP
#### Подготовка:
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
SELECT pg_reload_conf();

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
# Директория архивирования данных:
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
# Создание паралельного кластера:
pg_createcluster 14 main2
pg_lsclusters
# main2 не запущен:
# Удаляем данные из main2:
rm -rf /var/lib/postgresql/14/main2
# Перенос данных в main2:
pg_basebackup -p 5432 --pgdata=/var/lib/postgresql/14/main2
# Стуртуем кластер
pg_ctlcluster 14 main2 start # Ругается, но стартует:
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
systemctl start postgresql@14-main2
# Смотрим как стартовал
pg_lsclusters # или
systemctl status postgresql@14-main2
# Смотрим в main2:
psql -p 5433
otus=# select * from test;
 i
----
 10
 20
 30
(3 строки)
# Заполняем в main2 БД otus тестовой инф-ии:
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
### Логический перенос из main2 otus в main otus_main:
```
psql -p 5432
CREATE DATABASE otus_main;
date && pg_dump -p 5433 otus | psql -p 5432 --set ON_ERROR_STOP=on otus_main >> /backup/restore_db.log 2>&1 && date 
cat /backup/restore_db.log







```











