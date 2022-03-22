# pg_probackup на postgresql 13, 14
##### 1. Установка postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt update
apt install postgresql-14 -y
apt install postgresql-13 -y
# Проверить наличие пользователя
cat /etc/passwd | grep backup
```
##### 2. Установка pg_probackup
```
sudo sh -c 'echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb/ $(lsb_release -cs) main-$(lsb_release -cs)" > /etc/apt/sources.list.d/pg_probackup.list'
sudo wget -O - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG_PROBACKUP | sudo apt-key add - && sudo apt-get update
sudo apt update
sudo apt-get install pg-probackup-14 -y
sudo apt-get install pg-probackup-{14,13,12,11,10,9.6} -y
sudo apt-get install pg-probackup-14-dbg -y
sudo apt-get install pg-probackup-{14,13,12,11,10,9.6}-dbg -y
apt install postgresql-contrib -y
# Т.к probackup работает только с кластерами с checksums:
apt install postgresql-14-pg-checksums -y
apt install postgresql-13-pg-checksums -y
```
##### 3. Создание каталог для бекапа, т.к. в каталоге с БД бекапы создать нельзя. Для теста можно 777:
```
sudo rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
```
###### Добавляем в переменную BACKUP_PATH путь, чтобы не указывать его каждый раз куда делать бекапы::
```
echo "BACKUP_PATH=/home/backups/">>~/.bashrc
echo "export BACKUP_PATH">>~/.bashrc
cd $HOME
```
###### Применим оболочку:
```
. .bashrc
```
###### Просмотр переменной:
```
echo $BACKUP_PATH
```
##### 4. Создать пользователя backup с правами:
```
###### #sudo useradd backup - пользователь имеется
passwd backup
12345
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
##### 5.1  Инициализируем наш probackup :
```
pg_probackup-14 init
pg_probackup-13 init
```
###### Ответ: Backup catalog '/home/backups' successfully inited
###### 5.2  Активируем установленнный ранее checksums:
```
systemctl stop postgresql
/usr/lib/postgresql/14/bin/pg_checksums -D /var/lib/postgresql/14/main --enable
/usr/lib/postgresql/13/bin/pg_checksums -D /var/lib/postgresql/13/main --enable
systemctl start postgresql
```
##### 5.3  Добавить инстанс в наш probackup:
```
pg_probackup-14 add-instance --instance 'main' -D /var/lib/postgresql/14/main
pg_probackup-13 add-instance --instance 'main' -D /var/lib/postgresql/13/main
```
###### Ответ: Instance 'main' successfully inited
###### Теперь probackup знает где инстанс "main"
###### Смотрим что внутри директории бекапов:
```
cd $BACKUP_PATH
ls -la
```
###### Ответ: total 16

drwxrwxrwx 4 root root 4096 фев  2 08:41 .

drwxr-xr-x 4 root root 4096 фев  1 15:36 ..

drwx------ 2 root root 4096 фев  2 08:41 backups

drwx------ 2 root root 4096 фев  2 08:41 wal
```
sudo chmod -R 777 /home/backups
```

##### 7. Создаем и заполнение новой БД (не входя PSQL):

```
su postgres
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```
###### Ответ на последний запрос:

id|
:----:
10  
20  
30  
 
(3 строки)
###### Проcмотр конфига для инстанса main:
```
pg_probackup-14 show-config --instance main
```
#####  Ответ:
###### # Backup instance information
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

##### 8. Делаем полный бекап, потоковой репликации через временный слот  :
```
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot
pg_probackup-13 backup --instance 'main' -b FULL --stream --temp-slot
```
Ответ: WARNING: Curent PostgreSQL role is superuser. Исправляемся, следующий бекап из-под пользователя backup. Так что сейчас получилось? :
```
pg_probackup-14 show
pg_probackup-13 show
```

###### BACKUP INSTANCE 'main'
###### ==========================================================================================================
###### Instance   Version   ID       Recovery  Time            Mode   WAL  Mode   TLI   Time   Data    WAL   Zratio   Start  LSN   Stop  LSN    Status
###### ==========================================================================================================
###### main      14       R93D33  2022-03-21 13:57:04+03  FULL  STREAM    1/0   10s  34MB  16MB    1.00  0/2000028  0/2005B50  OK

###### Добавляем данные:
```
psql otus -c "insert into test values (40);"
```
##### 9. Делаем дельту-копию из=под backup:
```
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -U backup
pg_probackup-13 backup --instance 'main' -b DELTA --stream --temp-slot -U backup
```
Исправляем ошибку "no connect to database" backup is running:
``` 
psql -c "ALTER USER backup PASSWORD '12345';"
```
```
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -U backup -W
pg_probackup-13 backup --instance 'main' -b DELTA --stream --temp-slot -U backup -W
```
WARNING; backup R38402 has status ERROR

###### PGPASS Луна в юпитере:
```
echo "localhost:5432:otus:backup:12345">>~/.pgpass
chmod 600 ~/.pgpass
pg_probackup-14 backup --instance 'main' -b FULL --stream --temp-slot -h localhost -U backup --pgdatabase=otus
```
Ответ: Backup R6VP59 completed
```
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -h localhost -U backup --pgdatabase=otus
```
Ответ: Parent backup: R6VP59; Backup R6VP7D completed
###### Добавляем данные:                                                                                                               
```
psql otus -c "insert into test values (4);"
psql otus -c "insert into test values (5);"
pg_probackup-14 backup --instance 'main' -b DELTA --stream --temp-slot -h localhost -U backup --pgdatabase=otus
```
##### 10. Восстановление копию в новый кластер:
```
sudo pg_createcluster 14 main2
```
Ответ: Ver Cluster Port Status Owner    Data directory               Log file
14  main2   5433 down   postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
```
sudo rm -rf /var/lib/postgresql/14/main2
sudo pg_probackup-14 restore --instance 'main' -i 'R6VP7D' -D /var/lib/postgresql/14/main2 -B /home/backups
```
###### Ответ: Restore of backup R6VP7D completed.
```
sudo pg_ctlcluster 14 main2 start
```
###### Ответ: Data directory /var/lib/postgresql/14/main2 must not be owned by root
```
sudo chown -R postgres /var/lib/postgresql/14/main2
```
###### Проверка восстановления:
```
sudo -u postgres psql otus -p 5433 -c 'select * from test;'
```
###### Ответ: 
 i
----
 10
 20
 30
  4
(4 строки)
###### Восстановление по id бекапа:
```
sudo pg_createcluster 14 main2 stop
sudo rm -rf /var/lib/postgresql/14/main2
sudo pg_probackup-14 restore --instance 'main' -i 'R6VP7D' -D /var/lib/postgresql/14/main2 -B /home/backups
sudo pg_ctlcluster 14 main2 start
```
###### Восстановление на определенное время, id не указываем:
```
psql -c 'show archive_mode'
psql -c 'alter system set archive_mode = on'
psql
alter system set archive_command = 'pg_probackup-14 archive-push -B /home/backups/ --instance=main --wal-file-path=%p --wal-file-name=%f --compress';
\q
sudo pg_ctlcluster 14 main restart
psql -c 'show archive_mode'
psql -c 'show archive_command'
psql otus -c "insert into test values (9);"
date
pg_probackup-14 restore --instance 'main' -D /var/lib/postgresql/14/main2 -B /home/backups --recovery-target-time="2022-02-06 17:55:03+00"
```
###### Ответ: ERROR: Restore destination is not empty: "/var/lib/postgresql/14/main2"
```
sudo rm -rf /var/lib/postgresql/14/main2 && sudo mkdir /var/lib/postgresql/14/main2 && sudo chmod -R 777 /var/lib/postgresql/14/main2
```
INFO: Validating backup R6W17D
WARNING: Invalid CRC of backup file "/home/backups/backups/main/R6W17D/database/pg_wal/000000010000000000000010" : AD6C62A9. Expected 50EA82D
WARNING: Backup R6W17D data files are corrupted
ERROR: Backup R6W17D is corrupt.





































