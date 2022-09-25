# Backup-restore штатными средствами
###### 0.
```
Не беря во внимание утилиты файлового копирования PostgreS комплектуется 2-мя штатными утилитами для резервного копирования, 
это pg_dump/pg_dumpall и pg_basebackup. 



```
##### 1. Установка postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt update
apt install postgresql-14 -y
pg_isready
# Ответ: /var/run/postgresql:5432 - принимает подключения
```
# PG_DUMP
```
su postgres
mkdir /home/backups/1

```
```
# Задача теста: сравнение по времени, по потокам, загрузки CPU, загрузки hdd:
# При -Fc практически не будет сжатия; d (directory); -j (потоки); -F d (указ.формат.вывода d=директория) c (custom); -f (вывод в директорию путь обязателен);
# Простой вариант бекапа, в SQL-скрипте :
pg_dump -d otus --create > /home/backups/3.sql
# Простой со сжатием, в 2,8 раза меньше весит: 
pg_dump -d otus --create | gzip > /home/backups/otus3.gz
# Архив с оглавлением для pg_restore. Минимальная степень сжатия <10% -Fc: 
pg_dump -d otus -Fc > /home/backups/otus4.gz
# Восстановление-1:
drop database otus;
\q
psql < /home/backups/3.sql
```

##### Восстановление-2 (drop/create/pg_restore):
```
# pg_restore не отработает, если нет БД:
echo "drop database otus;" | sudo -u postgres psql
echo "create database otus;" | sudo -u postgres psql
echo "drop database otus;" | psql
echo "create database otus;" | psql
# Тестим на скорость выполнения в различных вариантах --jobs:
sudo -u postgres pg_restore otus3.gz -d otus
sudo -u postgres pg_restore -j 1 -d otus /home/backups/otus4.gz
sudo -u postgres pg_restore -j 10 -d otus /home/backups/otus4.gz
```
##### 2. Установка pg_probackup
```
sudo sh -c 'echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb/ $(lsb_release -cs) main-$(lsb_release -cs)" > /etc/apt/sources.list.d/pg_probackup.list'
sudo wget -O - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG_PROBACKUP | sudo apt-key add - && sudo apt-get update
sudo apt update
sudo apt-get install pg-probackup-14 -y
# sudo apt-get install pg-probackup-{14,13,12,11,10,9.6} -y
sudo apt-get install pg-probackup-14-dbg -y
# sudo apt-get install pg-probackup-{14,13,12,11,10,9.6}-dbg -y
apt install postgresql-contrib -y
# По умолчанию postgresql уст.без checksums: требует остановки и enable:
apt install postgresql-14-pg-checksums -y
```
##### 3. Создание каталога для бекапов. Права для теста можно 777:
```
sudo rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
```
##### 4. Добавляем в переменную BACKUP_PATH путь, из-под postgres:
```
su postgres
echo "BACKUP_PATH=/home/backups/">>~/.bashrc
echo "export BACKUP_PATH">>~/.bashrc
cd $HOME
```
###### Применим оболочку с новым профилем:
```
. .bashrc
```
###### Просмотр переменной:
```
echo $BACKUP_PATH
```
###### Ответ: /home/backups/
##### 5. Создать пользователя backup с правами:
```
su postgres
psql
create user backup;

ALTER USER backup WITH PASSWORD '12345';
ALTER ROLE backup NOSUPERUSER;
ALTER ROLE backup WITH REPLICATION;
GRANT USAGE ON SCHEMA pg_catalog TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.current_setting(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_is_in_recovery() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_start_backup(text, boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_stop_backup(boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_create_restore_point(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_switch_wal() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_last_wal_replay_lsn() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current_snapshot() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_snapshot_xmax(txid_snapshot) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_control_checkpoint() TO backup;

\q
```
##### 6.1  Инициализируем наш probackup, из-под postgres:
```
pg_probackup-14 init
```
###### Ответ: Backup catalog '/home/backups' successfully inited
###### Что внутри директории бекапов:
```
cd $BACKUP_PATH
ls -la
```
###### Ответ: создалось 2 каталога:
```
drwxrwxrwx 4 root root 4096 фев  2 08:41 .  
drwxr-xr-x 4 root root 4096 фев  1 15:36 ..  
drwx------ 2 postgres postgres 4096 фев  2 08:41 backups  
drwx------ 2 postgres postgres 4096 фев  2 08:41 wal
```
##### Уточнить, почему всем? в v_2022 нет:
```
sudo chmod -R 777 /home/backups
```
##### 6.2  Добавить инстанс в наш probackup из-под postgres:
```
pg_probackup-14 add-instance --instance 'main' -D /var/lib/postgresql/14/main
```
###### Ответ: INFO: Instance 'main' successfully inited   Теперь probackup знает где инстанс "main"

##### 7. Создаем и заполнение новой БД (не входя PSQL):
```
su postgres
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```
###### Ответ:
```
 i
----
 10
 20
 30
```
###### Проcмотр конфига для инстанса main:
```
pg_probackup-14 show-config --instance main
```
#####  Ответ: Backup instance information  
pgdata = /var/lib/postgresql/14/main  
system-identifier = 7076338028174019592  
xlog-seg-size = 16777216  
###### # Connection parameters  
pgdatabase = root  
###### # Replica parameters  
replica-timeout = 5min  
###### # Archive parameters  
archive-timeout = 5min  
###### # Logging parameters  
log-level-console = INFO  
log-level-file = OFF  
log-filename = pg_probackup.log  
log-rotation-size = 0TB  
log-rotation-age = 0d  
###### # Retention parameters  
retention-redundancy = 0  
retention-window = 0  
wal-depth = 0  
###### # Compression parameters  
compress-algorithm = none  
compress-level = 1  
###### # Remote access parameters  
remote-proto = ssh  
##### 8. Испольуем PGPASS через Луну в Юпитере: host:port:db_name:user_name:password
```
rm ~/.pgpass
echo "localhost:5432:otus:backup:12345">>~/.pgpass
chmod 600 ~/.pgpass
pg_ctlcluster 14 main stop
pg_ctlcluster 14 main start
```
##### 9. Делаем бекап из-под POSTGRES с параметрами: FULL, потоковая репликация+временный слот:
```
# v_2022
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot
# Ответ:
INFO: Backup start, pg_probackup version: 2.5.8, instance: main, backup ID: RIEQ                                                                                                                                TF, backup mode: FULL, wal mode: STREAM, remote: false, compress-algorithm: none                                                                                                                                , compress-level: 1
WARNING: This PostgreSQL instance was initialized without data block checksums.                                                                                                                                 pg_probackup have no way to detect data block corruption without them. Reinitial                                                                                                                                ize PGDATA with option '--data-checksums'.
WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_p                                                                                                                                robackup under superuser.
INFO: Database backup start
INFO: wait for pg_start_backup()
INFO: Wait for WAL segment /home/backups/backups/main/RIEQTF/database/pg_wal/000                                                                                                                                000010000000000000002 to be streamed
INFO: PGDATA size: 33MB
INFO: Current Start LSN: 0/2000028, TLI: 1
INFO: Start transferring data files
INFO: Data files are transferred, time elapsed: 0
INFO: wait for pg_stop_backup()
INFO: pg_stop backup() successfully executed
INFO: stop_lsn: 0/20020D0
INFO: Getting the Recovery Time from WAL
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 0
INFO: Validating backup RIEQTF
INFO: Backup RIEQTF data files are valid
INFO: Backup RIEQTF resident size: 50MB
INFO: Backup RIEQTF completed
# Предупреждения: checksumm - инициализация+start\stop; superuser исключить.
```
###### v_2021 было:
```
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432
```
###### Ответ: WARNING: Curent PostgreSQL role is superuser. Исправляемся, следующий бекап из-под пользователя backup. Что сейчас получилось с бекапами? :
```
pg_probackup-14 show
```
```
# V_2022
BACKUP INSTANCE 'main'
================================================================================================================================
 Instance  Version  ID      Recovery Time           Mode  WAL Mode  TLI  Time  Data   WAL  Zratio  Start LSN  Stop LSN   Status
================================================================================================================================
 main      14       RIEQTF  2022-09-18 16:29:41+03  FULL  STREAM    1/0   11s  34MB  16MB    1.00  0/2000028  0/20020D0  OK
```
```
# V_2021
###### BACKUP INSTANCE 'main'
###### ==========================================================================================================
###### Instance   Version   ID       Recovery  Time            Mode   WAL  Mode   TLI   Time   Data    WAL   Zratio   Start  LSN   Stop  LSN    Status
###### ==========================================================================================================
###### main      14       R93D33  2022-03-21 13:57:04+03  FULL  STREAM    1/0   10s  34MB  16MB    1.00  0/2000028  0/2005B50  OK
```
###### 10. Исправляем отсутствие checksums, инициализацияся на выкл.кластере, из-под postgres:
```
systemctl stop postgresql
su postgres
/usr/lib/postgresql/14/bin/pg_checksums -D /var/lib/postgresql/14/main --enable
exit
systemctl start postgresql
```
###### Ответ: Контрольные суммы в кластере включены.
##### 11. Добавляем данные:                                                                                                               
```
psql otus -c "insert into test values (40);"
```
###### 12. Делаем дельта-backup с хостовым пользователем backup. Другой путь: Установим пароль на backup в БД :
```
# V_2022
psql -c "ALTER USER backup PASSWORD '12345';"
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -h localhost -U backup -W
# Ответ:
INFO: Backup start, pg_probackup version: 2.5.8, instance: main, backup ID: RIESAV, backup mode: DELTA, wal mode: STREAM, remote: false, compress-algorithm: none, compress-level: 1
Password for user backup:
INFO: This PostgreSQL instance was initialized with data block checksums. Data block corruption will be detected
INFO: Database backup start
INFO: wait for pg_start_backup()
INFO: Parent backup: RIEQTF
INFO: Wait for WAL segment /home/backups/backups/main/RIESAV/database/pg_wal/000000010000000000000004 to be streamed
INFO: PGDATA size: 33MB
INFO: Current Start LSN: 0/4000028, TLI: 1
INFO: Parent Start LSN: 0/2000028, TLI: 1
INFO: Start transferring data files
INFO: Data files are transferred, time elapsed: 1s
INFO: wait for pg_stop_backup()
INFO: pg_stop backup() successfully executed
INFO: stop_lsn: 0/40001A0
INFO: Getting the Recovery Time from WAL
INFO: Syncing backup files to disk
INFO: Backup files are synced, time elapsed: 0
INFO: Validating backup RIESAV
INFO: Backup RIESAV data files are valid
INFO: Backup RIESAV resident size: 21MB
INFO: Backup RIESAV completed

# Примечание: Из-под авторизованного linux-user backup пароль не затребует
```
```
# V_2021
psql -c "ALTER USER backup PASSWORD '12345';"
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432

psql otus -c "insert into test values (50);"
```
##### Получаем ошибку:
```
ERROR: query failed: ОШИБКА:  нет доступа к функции pg_start_backup query was: SELECT pg_catalog.pg_start_backup($1, $2, false)
```
##### Исправляем ошибку:
```
psql -d otus
GRANT USAGE ON SCHEMA pg_catalog TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.current_setting(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_is_in_recovery() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_start_backup(text, boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_stop_backup(boolean, boolean) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_create_restore_point(text) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_switch_wal() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_last_wal_replay_lsn() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_current_snapshot() TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.txid_snapshot_xmax(txid_snapshot) TO backup;
GRANT EXECUTE ON FUNCTION pg_catalog.pg_control_checkpoint() TO backup;
```
##### Повторяем бекап OTUS
```
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432
psql otus -c "insert into test values (60);"
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432
```
```
pg_probackup-14 show
# Ответ:
 Instance  Version  ID      Recovery Time           Mode   WAL Mode  TLI  Time   Data   WAL  Zratio  Start LSN  Stop LSN   Status
==================================================================================================================================
 main      14       RIG0VK  2022-09-19 09:04:38+03  DELTA  STREAM    1/1   16s  167kB  16MB    1.00  0/8000028  0/80001A0  OK
 main      14       RIG0QG  2022-09-19 09:01:33+03  FULL   STREAM    1/0   37s   34MB  16MB    1.00  0/6000028  0/6009CC8  OK
 main      14       RIG05W  ----                    DELTA  STREAM    0/1    5s      0     0    1.00  0/0        0/0        ERROR
 main      14       RIG03V  ----                    FULL   STREAM    0/0    4s      0     0    1.00  0/0        0/0        ERROR
 main      14       RIFZZ0  2022-09-19 08:45:12+03  DELTA  STREAM    1/1   23s  183kB  16MB    1.00  0/4000028  0/4000168  OK
 main      14       RIFZLG  2022-09-19 08:36:54+03  FULL   STREAM    1/0   11s   34MB  32MB    1.00  0/2000028  0/2009C90  OK

```
###### Добавляем данные. :
```
psql otus -c "insert into test values (70);"
```
##### 13. Восстановление копию в новый кластер:
```
# Создаем инстанс main2
sudo pg_createcluster 14 main2
# Ответ: Ver Cluster Port Status Owner    Data directory               Log file
14  main2   5433 down   postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
# Смотрим статус main2
pg_ctlcluster 14 main2 status
# Ответ: pg_ctl: сервер не работает. Удаляем данные из main2:
sudo rm -rf /var/lib/postgresql/14/main2
#  Попытаемся восстановиться на последний бекап, смотрим его id: RIG0VK, однако:
```
##### Что бы заработало PITR wal-файлы должны быть непрерывное архивирование:
```
psql -c 'alter system set archive_mode = on'
psql
alter system set archive_command = 'pg_probackup-14 archive-push -B /home/backups/ --instance=main --wal-file-path=%p --wal-file-name=%f --compress';
pg_ctlcluster 14 main restart
psql -c 'show archive_mode'
# Ответ: archive_mode
--------------
 on
 psql -c 'show archive_command'
 # Ответ: archive_command
 # Показ всех 3-х параметров:
postgres=# SELECT name, setting FROM pg_settings WHERE name IN ('archive_mode','archive_command','archive_timeout');
      name       |                                                     setting
-----------------+-----------------------------------------------------------------------------------------------------------------
 archive_command | pg_probackup-14 archive-push -B /home/backups/ --instance=main --wal-file-path=%p --wal-file-name=%f --compress
 archive_mode    | on
 archive_timeout | 0
(3 строки)
vim /etc/postgresql/14/main/postgresql.conf
archive_timeout = 600
-----------------------------------------------------------------------------------------------------------------
 pg_probackup-14 archive-push -B /home/backups/ --instance=main --wal-file-path=%p --wal-file-name=%f --compress
```
###### Аристов: мы перешли в другой режим бекапирования и лучше начинать с FULL бекапа:
```
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432
psql otus -c "select * from test;"
# Ответ: 
 i
----
 10
 20
 30
 40
 50
 60
 70
(7 строк)
pg_probackup-14 show
# Добавляем данные. Значение 80 после восстановления должны потерять:
psql otus -c "insert into test values (80);"
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot -h localhost -U backup --pgdatabase=otus -p 5432

pg_probackup-14 restore --instance 'main' -D /var/lib/postgresql/14/main2 --recovery-target-time="2022-09-19 12:40:03"
# Ответ:
INFO: Validating backup RIG979
INFO: Backup RIG979 data files are valid
INFO: Backup validation completed successfully on time 2022-09-19 12:48:14+03, xid 769 and LSN 0/B000520
INFO: Backup RIG979 is valid.
INFO: Restoring the database from backup at 2022-09-19 12:04:21+03
INFO: Start restoring backup files. PGDATA size: 49MB
INFO: Backup files are restored. Transfered bytes: 49MB, time elapsed: 0
INFO: Restore incremental ratio (less is better): 100% (49MB/49MB)
INFO: Syncing restored files to disk
INFO: Restored backup files are synced, time elapsed: 31s
INFO: Restore of backup RIG979 completed.
```
##### Стартуем и смотрим востановленный кластер main2:
```
sudo pg_ctlcluster 14 main2 start
psql otus -p 5433 -c 'select * from test;'
# Ответ: 80 нет, т.к. запись создавалась в 12-50, а восстановление было на 12-40:
 i
----
 10
 20
 30
 40
 50
 60
 70
(7 строк)
```


# PG_BASEBACKUP
###### Работа по протоколу репликации. Cоздаёт копию файлов кластера целиком. Проверим, что есть настройки по умолчанию для физич-го резервирования:
```
postgres=# SELECT name, setting FROM pg_settings WHERE name IN ('wal_level','max_wal_senders');
      name       | setting
-----------------+---------
 max_wal_senders | 10
 wal_level       | replica
(2 строки)
# Параметры ph_hba.conf касательно репликации:
SELECT type, database, user_name, address, auth_method FROM pg_hba_file_rules() WHERE database = '{replication}';

 type  |   database    | user_name |  address  |  auth_method
-------+---------------+-----------+-----------+---------------
 local | {replication} | {all}     |           | peer
 host  | {replication} | {all}     | 127.0.0.1 | scram-sha-256
 host  | {replication} | {all}     | ::1       | scram-sha-256
(3 строки)
# Создаем инстанс main2
sudo pg_createcluster 14 main2
pg_lsclusters
# Ответ:
14  main    5432 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main2   5433 down   postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log

# Удаляем данные из main2:
sudo rm -rf /var/lib/postgresql/14/main2
# Сделаем бэкап нашей БД
su postgres
pg_basebackup -p 5432 -D /var/lib/postgresql/14/main2
#Зададим другой порт - не заработает
# echo 'port = 5433' >> /var/lib/postgresql/14/main2/postgresql.auto.conf - не заработает
# Стуртуем кластер
pg_ctlcluster 14 main2 start
# Смотрим как стартовал
pg_lsclusters
14  main    5432 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main2   5433 online postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
psql -p5433
\c otus
otus=# select * from test;
 i
----
 10
 20
 30
(3 строки)
```
##### Потоковый архив wal-файлов
```
# Смотри параметры:
SELECT name, setting FROM pg_settings WHERE name IN ('archive_mode','archive_command','archive_timeout');
      name       |  setting
-----------------+------------
 archive_command | (disabled)
 archive_mode    | off
 archive_timeout | 0
(3 строки)
```
```
# Исправляем https://habr.com/ru/post/525308/ - не отвечает
vim /etc/postgresql/14/main/postgresql.conf 
archive_mode: on
archive_command: /usr/local/bin/copy_wal.sh %p %f
archive_timeout: 600
# вводим команды: 
psql -c 'alter system set archive_mode = on'  --правка postgresql.auto.conf
systemctl restart postgresql
psql -c 'show archive_mode'


```
```
vim ~/.pgpass
localhost:5432:*:postgres:12345
chmod 0600 ~/.pgpass
```


```
sudo -u postgres pg_dump --no-password --format=directory -v --host=localhost -p 5432 --username=postgres --dbname=otus -f /home/backups/2/otus.dmp
sudo -u postgres pg_dump -Fc -v --host=localhost --username=postgres --dbname=otus -f testdb.dump
sudo -u postgres pg_dump -d otus -Fc > /home/backups/1
sudo -u postgres pg_dump -d otus -j 1 -Fc -F d -f /home/backups/1
```
```
vim dump.sh
```
```
#!/bin/sh

PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

PGPASSWORD=
export PGPASSWORD
pathB=/home/backups/1
dbUser=postgres
database=otus

find $pathB \( -name "*-1[^5].*" -o -name "*-[023]?.*" \) -ctime +61 -delete
pg_dump -U $dbUser $database | gzip > $pathB/pgsql_$(date "+%Y-%m-%d").sql.gz

unset PGPASSWORD
```
```
# Делаем исполняемый файл:
chmod u+x dump.sh
sudo chown -R postgres /home/mgb/dump.sh
sudo chown -R postgres /home/backups/1
sudo chown -R postgres /home/backups/1
su postgres
/home/mgb/dump.sh
```
```
# Востановление pg_restore -v (сообщения)
pg_restore -h localhost -p 5432 -U postgres -d otus -v "/home/backups/1/psql-2022-09-21.sql.gz"
pg_restore -j 2 --verbose --clean --no-acl --no-owner --host=localhost -p 5432 --dbname=otus --username=postgres "/home/backups/1/psql-2022-09-21.sql.gz"
pg_restore -j 2 -h localhost -U postgres -F c -d otus "/home/backups/1/psql-2022-09-21.sql.gz"
```
```
```























