# pg_probackup на postgresql 14
###### 1. Установка postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt install postgresql-14
```
###### 2. Установка pg_probackup
```
sudo sh -c 'echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb/ $(lsb_release -cs) main-$(lsb_release -cs)" > /etc/apt/sources.list.d/pg_probackup.list'
sudo wget -O - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG_PROBACKUP | sudo apt-key add - && sudo apt-get update
sudo apt-get install pg-probackup-{14,13,12,11,10,9.6}
sudo apt-get install pg-probackup-{14,13,12,11,10,9.6}-dbg
apt install postgresql-contrib -y
```
###### По умолчанию postgresql без checksums, а pg-probackup работает с кластерами c checksums:
```
apt install postgresql-14-pg-checksums -y
```
###### Создаем каталог с бекапами:
```
sudo rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod 777 /home/backups
```
###### Есть баг: в каталоге по умолчанию var/lib/postgresql/14/main/backups/ создается только первый backup, следующие с error.
###### Добавляем переменную BACKUP_PATH Что бы не указывать каждый раз куда делать бекапы:
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
###### Создать пользователя backup с правами:
```
su postgres
psql
create user backup;
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
exit
```
###### Инициализируем наш probackup для v_13 или v_14 :
```
pg_probackup-14 init
pg_probackup-13 init
```
###### Ответ :
INFO: Backup catalog '/home/backups' successfully inited
###### Смотрим что внутри директории бекапов:
```
cd $BACKUP_PATH
ls -la
```
###### Ответ:
total 16

drwxrwxrwx 4 root root 4096 фев  2 08:41 .

drwxr-xr-x 4 root root 4096 фев  1 15:36 ..

drwx------ 2 root root 4096 фев  2 08:41 backups

drwx------ 2 root root 4096 фев  2 08:41 wal


###### Добавить инстанс в наш probackup:
```
pg_probackup-14 add-instance --instance 'main' -D /var/lib/postgresql/14/main
```
###### Ответ:
INFO: Instance 'main' successfully inited
###### Теперь probackup знает где инстанс "main"

###### Создаем новую БД не входя PSQL:

```
su postgres
psql -c "CREATE DATABASE otus;"

```
####### Заполним БД тестовыми данными:

psql otus -c "create table test(i int);"

psql otus -c "insert into test values (10), (20), (30);"

psql otus -c "select * from test;"

###### Ответ:

 i
-
 10
 20
 30
 
(3 строки)
```
pg_probackup-14 show-config --instance main
```








