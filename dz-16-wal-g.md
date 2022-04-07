# Над пропастью WAL-G.

###### 1. Установка Postgresql-13
###### 2. Установка WAL-G
```
wget https://github.com/wal-g/wal-g/releases/download/v1.1.2-rc/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -zxvf /home/mgb/wal-g-pg-ubuntu-20.04-amd64.tar.gz
mkdir /usr/local/bin/wal-g
chown -R postgres /usr/local/bin/wal-g
mv /home/mgb/wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g
ls -l /usr/local/bin/wal-g
rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
su postgres
mkdir /var/lib/postgresql/13/main/log
vim /var/lib/postgresql/.walg.json
{
    "WALG_FILE_PREFIX": "/home/backups",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "5",

    "PGDATA": "/var/lib/postgresql/13/main",

    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}
```
###### DESC .walg.json: каталог для бекапов; сжатие=brotli; delta=5; плдключение через linux-socket;


###### 4. Правим  postgresql.conf через auto.conf:
```
echo "wal_level=replica" >> /var/lib/postgresql/13/main/postgresql.auto.conf
echo "archive_mode=on" >> /var/lib/postgresql/13/main/postgresql.auto.conf
echo "archive_command='wal-g wal-push \"%p\" >> /var/lib/postgresql/13/main/log/archive_command.log 2>&1' " >> /var/lib/postgresql/13/main/postgresql.auto.conf 
echo "archive_timeout=60" >> /var/lib/postgresql/13/main/postgresql.auto.conf 
echo "restore_command='wal-g wal-fetch \"%f\" \"%p\" >> /var/lib/postgresql/13/main/log/restore_command.log 2>&1' " >> /var/lib/postgresql/13/main/postgresql.auto.conf
```
###### 5. Создадим тестовую базу данных с данными:
```
psql -c "CREATE DATABASE otus;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```

###### 6.Делаем бэкап:
```
wal-g backup-push /var/lib/postgresql/13/main
wal-g backup-list
```
### wal-g: command not found

###### 7. Добавляем данные:
```
psql otus -c "UPDATE test SET i = 3 WHERE i = 30"
```
###### 8.Делаем бэкап:
 ```
wal-g backup-push /var/lib/postgresql/13/main
wal-g backup-list
```
### wal-g: command not found











