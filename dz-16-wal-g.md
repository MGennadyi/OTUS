# Над пропастью WAL-G.

###### 1. Установка Postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
apt install postgresql-14 -y
pg_isready
```
###### 2. Установка WAL-G
```
wget https://github.com/wal-g/wal-g/releases/download/v1.1.2-rc/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -zxvf /home/mgb/wal-g-pg-ubuntu-20.04-amd64.tar.gz
mv /home/mgb/wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g
ls -la /usr/local/bin/
chown -R postgres /usr/local/bin/wal-g
ls -la /usr/local/bin/
```
###### Ответ: -rwxr-xr-x  1 postgres root 41443344 ноя 26 17:11 wal-g

##### 3. Создаем каталог для бекапов:
```
rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
chown -R postgres /home/backups
```
##### 4. Создать директорию для логов:
```
su postgres
mkdir /var/lib/postgresql/14/main/log
ls -l /var/lib/postgresql/14/main/log
```
##### 5. Под postgres создать скрытый конфиг для wal-g подключение через linux-socket:
```
su postgres
vim /var/lib/postgresql/.walg.json
{
    "WALG_FILE_PREFIX": "/home/backups",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "5",

    "PGDATA": "/var/lib/postgresql/14/main",

    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}
ls -la /var/lib/postgresql/
```
###### DESC .walg.json: каталог для бекапов; сжатие=brotli; delta=5; подключение через linux-socket;
###### 6. Правим  postgresql.conf через auto.conf, все команды разом:
```
echo "wal_level=replica" >> /var/lib/postgresql/14/main/postgresql.auto.conf
echo "archive_mode=on" >> /var/lib/postgresql/14/main/postgresql.auto.conf
echo "archive_command='wal-g wal-push \"%p\" >> /var/lib/postgresql/14/main/log/archive_command.log 2>&1' " >> /var/lib/postgresql/14/main/postgresql.auto.conf 
echo "archive_timeout=60" >> /var/lib/postgresql/14/main/postgresql.auto.conf 
echo "restore_command='wal-g wal-fetch \"%f\" \"%p\" >> /var/lib/postgresql/14/main/log/restore_command.log 2>&1' " >> /var/lib/postgresql/14/main/postgresql.auto.conf

cat ~/14/main/postgresql.auto.conf
pg_ctlcluster 14 main stop
pg_ctlcluster 14 main start
```
###### 7. Создадим тестовую базу данных с данными:
```
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```
###### 6.Делаем backup-push:
```
wal-g backup-push /var/lib/postgresql/14/main
```
###### Ужас: Couldn't find previous backup.
```
wal-g backup-list
```
###### НЕ все так плохо: name     modified          wal_segment_backup_start
base_000000010000000000000003 2022-04-09T13:44:44+03:00 000000010000000000000003
###### 7. Добавляем данные:
```
psql otus -c "UPDATE test SET i = 3 WHERE i = 30"
```
###### 8. Повторяем backup-push:
 ```
wal-g backup-push /var/lib/postgresql/14/main
```
###### Получаем: LATEST backup is: 'base_000000010000000000000003'. Delta backup from base_000000010000000000000003 with LSN 3000028.
##### 9. Восстановление на инстансе main2:
```
su postgres
pg_createcluster 14 main2
rm -rf /var/lib/postgresql/14/main2
wal-g backup-fetch /var/lib/postgresql/14/main2 LATEST
```
###### Получаем: Selecting the latest backup...
INFO: 2022/04/09 16:44:03.325798 LATEST backup is: 'base_00000001000000000000001E_D_000000010000000000000006'

INFO: 2022/04/09 16:44:05.834564 Backup extraction complete. Бэкап сформировался за 0,02 сек.
##### 10. На точку времени создать файл для восстановления из архивов wal:
```
touch "/var/lib/postgresql/14/main2/recovery.signal"
exit
sudo systemctl start postgresql@14-main2

pg_ctlcluster 14 main2 start
```



#### Доп.Задание: снять бекап под нагрузкой

##### 9. Установка pgbanch:
```
apt-get install postgresql postgresql-contrib
sudo -u postgres pgbench -i -s 10 otus
sudo -u postgres pgbench -c 10 -j 2 -t 10000 otus
```


