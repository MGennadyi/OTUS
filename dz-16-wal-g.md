# Над пропастью WAL-G.

###### 1. Установка Postgresql-14
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
# Добавляем репозиторий:
echo "deb [signed-by=/usr/share/keyrings/postgresql-keyring.gpg] http://apt.postgresql.org/pub/repos/apt/ bullseye-pgdg main" | sudo tee /etc/apt/sources.list.d/postgresql.list
sudo apt update
apt install gnupg2 -y
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /usr/share/keyrings/postgresql-keyring.gpg
sudo apt-cache search postgresql | grep postgresql
apt install postgresql-14 -y
pg_isready
# Ответ:
/var/run/postgresql:5432 - accepting connections
```
```
vim /etc/hosts
192.168.5.162 etcd1.kolomna.centr.oe	etcd1
192.168.5.163 etcd2.kolomna.centr.oe	etcd2
192.168.5.164 etcd3.kolomna.centr.oe	etcd3
192.168.5.165 pg1.kolomna.centr.oe	pg1
192.168.5.166 pg2.kolomna.centr.oe	pg2
192.168.5.167 pg3.kolomna.centr.oe	pg3
192.168.5.168 haproxy1.kolomna.centr.oe	haproxy1
192.168.5.169 haproxy2.kolomna.centr.oe	haproxy2
192.168.5.170 ansible.kolomna.centr.oe	ansible
192.168.5.171 wal-g.kolomna.centr.oe	wal-g
192.168.5.180 keepalived.kolomna.centr.oe	keepalived
#
10.128.0.61 etcd1.ru-central1.internal etcd1
10.128.0.62 etcd2.ru-central1.internal etcd2
10.128.0.63 etcd2.ru-central1.internal etcd3
10.128.0.64 patroni1.ru-central1.internal patroni1
10.128.0.65 patroni2.ru-central1.internal patroni2
10.128.0.66 patroni3.ru-central1.internal patroni3
10.128.0.67 haproxy1.ru-central1.internal	haproxy1
10.128.0.68 haproxy2.ru-central1.internal	haproxy2
10.128.0.69 keepalived.ru-central1.internal	keepalived
```
###### 2. Установка WAL-G
```
# v.1.1.2
wget https://github.com/wal-g/wal-g/releases/download/v1.1.2-rc/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -zxvf /home/mgb/wal-g-pg-ubuntu-20.04-amd64.tar.gz
mv /home/mgb/wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g
# v.2.0.1
wget https://github.com/wal-g/wal-g/releases/download/v2.0.1/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -zxvf /home/mgb/wal-g-pg-ubuntu-20.04-amd64.tar.gz
mv /home/mgb/wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g
ls -la /usr/local/bin/
# Ответ:
drwxr-xr-x  2 root root     4096 авг 16 08:50 .
drwxr-xr-x 10 root root     4096 июн 21 17:19 ..
-rwxr-xr-x  1 root root      215 июн 29 11:48 patroni
-rwxr-xr-x  1 root root      218 июн 29 11:48 patroni_aws
-rwxr-xr-x  1 root root      208 июн 29 11:48 patronictl
-rwxr-xr-x  1 root root      222 июн 29 11:48 patroni_raft_controller
-rwxr-xr-x  1 root root      227 июн 29 11:48 patroni_wale_restore
-rwxr-xr-x  1 root root 41443344 ноя 26  2021 wal-g
-rwxr-xr-x  1 root root      124 июн 29 11:48 ydiff

chown -R postgres /usr/local/bin/wal-g
ls -la /usr/local/bin/
# Ответ:
-rwxr-xr-x  1 postgres root 41443344 ноя 26  2021 wal-g
```
###### Проверка версии wal-g:
```
wal-g --version
# Ответ:
wal-g version v1.1.2-rc 6af461f 2021.11.26_14:09:59     PostgreSQL
wal-g version v2.0.1    b7d53dd 2022.08.25_09:34:20     PostgreSQL
```
##### 3. Создаем каталог для бекапов:
```
rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
chown -R postgres /home/backups
```
##### 4. Создать директорию для логов WAL-G:
```
su postgres
# v.14
mkdir /var/lib/postgresql/14/main/log
ls -la /var/lib/postgresql/14/main/log
```
##### 5. Из-под postgres создать скрытый конфиг wal-g; подключение через linux-socket:
```
# Бекапы не делаются из-под root, поэтому в домашнем каталоге должен быть:
su postgres
vim ~/.walg.json
# или
vim /var/lib/postgresql/.walg.json
{
    "WALG_FILE_PREFIX": "/home/backups",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "6",

    "PGDATA": "/var/lib/postgresql/14/main",

    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}

ls -la /var/lib/postgresql/
```
#### 5.2 YANDEX Из-под postgres:
```
# Аристов указ.доп.параметр расш.лог:    "WALG_LOG_LEVEL": "DEVEL"
# Аристов изменил "PGHOST": "/var/run/postgresql/.s.PGSQL.5432" на "localhost"
# Ругаться, если указать: "unix_socket_directories": "/var/run/postgresql/" , прописывается в конце .yml

su postgres
vim ~/.walg.json

{
    "WALG_FILE_PREFIX": "/home/backups",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "6",

    "PGDATA": "/var/lib/postgresql/14/main",

    "PGHOST": "localhost",
    
    "PGPORT": "5432"
    
}
```
##### Просмотр расположения unix_socket:
```
sudo vim /etc/patroni.yml
# В группе parametrs по дефолту:  unix_socket_directories: '.'
psql -h localhost
postgres=# show unix_socket_directories;
# Ответ:
unix_socket_directories
-------------------------
 .
(1 row)
# Изменить на !!!:  unix_socket_directories: '/var/run/postgresql/'

sudo patronictl -c /etc/patroni.yml edit-config
# Ответ:

loop_wait: 10
maximum_lag_on_failover: 1048576
postgresql:
  parameters: null
  use_pg_rewind: true
retry_timeout: 10
ttl: 30
pending restart

```
###### Правим конфиг: vim /etc/patroni.yml   
###### ru.stackoverflow.com советует, применим относительно patroni:
```
# В секции parameters:  
        wal_level=replica
        archive_mode=on
        archive_command='wal-g wal-push "%p"
        archive_timeout=60
        restore_command='wal-g wal-fetch "%f" "%p"
# В секции postgresql:
  shared_preload_libraries = 'pg_stat_statements'
  unix_socket_directories: /var/lib/postgresql/14/main
```
```
vim /var/lib/postgresql/14/main/postgresql.conf
vim /var/lib/postgresql/14/main/postgresql.base.conf
systemctl stop patroni
systemctl start patroni
```
```
Базовая настройка Patroni помещает файл SOCK UNIX в каталог данных Postgres. 
Вот почему вы столкнулись с приведенной выше ошибкой. 
Если вы настроите этот файл UNIX SOCK из каталога данных Postgres, все будет в порядке.
Вы можете использовать следующую конфигурацию, чтобы дать указание посетителям сделать это:
 parameters:
    unix_socket_directories: "/var/run/postgresql"
(I have: /var/run/postgresql directory owned by postgres:postgres)
Я имею /var/lib/postgresql/14/main
```

```
apt install net-tools -y
netstat -nlp | grep 5432
```
```
# Ответ:
drwxr-xr-x  3 postgres postgres 4096 авг 12 12:58 .
drwxr-xr-x 41 root     root     4096 июн 29 11:23 ..
drwx------  4 postgres postgres 4096 июн 29 13:53 14
-rw-------  1 postgres postgres  135 июл  1 09:43 .bash_history
-rw-------  1 postgres postgres    6 июн 30 15:30 .psql_history
-rw-------  1 postgres postgres 1308 авг 12 12:58 .viminfo
-rw-r--r--  1 postgres postgres  218 авг 12 12:58 .walg.json
```
###### DESC .walg.json: каталог для бекапов; сжатие=brotli; delta=6 хранимых копий; подключение через linux-socket порт 5432;
###### 6. Правим  postgresql.conf через auto.conf, все команды разом:
```
echo "wal_level=replica" >> /var/lib/postgresql/14/main/postgresql.auto.conf
echo "archive_mode=on" >> /var/lib/postgresql/14/main/postgresql.auto.conf
echo "archive_command='wal-g wal-push \"%p\" >> /var/lib/postgresql/14/main/log/archive_command.log 2>&1' " >> /var/lib/postgresql/14/main/postgresql.auto.conf 
echo "archive_timeout=60" >> /var/lib/postgresql/14/main/postgresql.auto.conf 
echo "restore_command='wal-g wal-fetch \"%f\" \"%p\" >> /var/lib/postgresql/14/main/log/restore_command.log 2>&1' " >> /var/lib/postgresql/14/main/postgresql.auto.conf
```
##### Смотрим, что все записалось:
```
cat ~/14/main/postgresql.auto.conf

wal_level=replica
archive_mode=on
archive_command='wal-g wal-push "%p" >> /var/lib/postgresql/14/main/log/archive_command.log 2>&1'
archive_timeout=60
restore_command='wal-g wal-fetch "%f" "%p" >> /var/lib/postgresql/14/main/log/restore_command.log 2>&1'
```
###### Рестарт
```
# Перезапуск, ругается :
pg_ctlcluster 14 main stop
pg_ctlcluster 14 main start
# Перезапуск, молчит:
sudo systemctl stop postgresql@14-main
sudo systemctl start postgresql@14-main
# Однако, управление через PATRONI, от сюда и 
systemctl stop patroni
systemctl start patroni
```
###### На master node:
```
sudo -u postgres psql -h localhost
select pg_switch_wal();
# Ответ:
--------------
 0/8000000
(1 строка)
select * from pg_stat_activity \gx
# Ответ: длинная портянка
```
###### Проверка доступности через netcat gp3 gp4 по портам:
```
sudo apt install netcat
gpadmin@gp1:/home/mgb$ nc -vz 10.128.0.54 7000
Connection to 10.128.0.54 7000 port [tcp/afs3-fileserver] succeeded!
gpadmin@gp1:/home/mgb$ nc -vz 10.128.0.54 6000
Connection to 10.128.0.54 6000 port [tcp/x11] succeeded!
```
###### Проверка доступности через netstat по портам:
```
apt install net-tools -y
netstat -nlp | grep 5432
# Ответ:
tcp        0      0 192.168.5.165:5432      0.0.0.0:*               LISTEN      490/postgres
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      490/postgres
unix  2      [ ACC ]     STREAM     LISTENING     13157    490/postgres         ./.s.PGSQL.5432
```

###### 7. Создадим тестовую базу данных с данными:
```
# Включаем показ тайминга выполнения команд:
\timing
psql -c "CREATE DATABASE otus1;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```
###### 6. Делаем backup-push:
```
time wal-g backup-push /var/lib/postgresql/14/main
# Ответ: Couldn't find previous backup.
```
```
wal-g backup-list
```
###### НЕ все так плохо: name     modified          wal_segment_backup_start
```
base_000000010000000000000003 2022-04-09T13:44:44+03:00 000000010000000000000003
```
###### 7. Добавляем данные:
```
psql otus -c "UPDATE test SET i = 3 WHERE i = 30"
```
###### 8. Повторяем backup-push:
 ```
wal-g backup-push /var/lib/postgresql/14/main
Ответ: LATEST backup is: 'base_000000010000000000000003'. Delta backup from base_000000010000000000000003 with LSN 3000028.
```
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
###### Настройка расписания резервного копирования
```
#!/bin/bash

echo "15 4 * * *    /usr/local/bin/wal-g backup-push /var/lib/postgresql/14/main >> /var/log/postgresql/walg_backup.log 2>&1" >> /var/spool/cron/crontabs/postgres
# задаем владельца и выставляем правильные права файлу
chown postgres: /var/spool/cron/crontabs/postgres
chmod 600 /var/spool/cron/crontabs/postgres
```
###### Удаление старых резервных копий
```
#!/bin/bash
echo "30 6 * * *    /usr/local/bin/wal-g delete before FIND_FULL \$(date -d '-5 days' '+\\%FT\\%TZ') --confirm >> /var/log/postgresql/walg_delete.log 2>&1" >> /var/spool/cron/crontabs/postgres
```
###### CRON
```
# Просмотр заданий:
crontab -l
# Редактирование задание через vim
crontab -e
# Удаление
crontab -r

postgres@wal-g2:/home/mgb$ wal-g backup-list --pretty
+---+----------------------------------------------------------+----------------------------------+--------------------------+
| # | NAME                                                     | MODIFIED                         | WAL SEGMENT BACKUP START |
+---+----------------------------------------------------------+----------------------------------+--------------------------+
| 0 | base_00000001000000000000007F_D_00000001000000000000007C | Saturday, 03-Dec-22 14:38:42 MSK | 00000001000000000000007F |
| 1 | base_000000010000000000000081_D_00000001000000000000007F | Saturday, 03-Dec-22 14:38:49 MSK | 000000010000000000000081 |
| 2 | base_000000010000000000000083_D_000000010000000000000081 | Saturday, 03-Dec-22 14:39:13 MSK | 000000010000000000000083 |
| 3 | base_000000010000000000000086_D_000000010000000000000083 | Saturday, 03-Dec-22 14:46:24 MSK | 000000010000000000000086 |
+---+----------------------------------------------------------+----------------------------------+--------------------------+
postgres@wal-g2:/home/mgb$


vim /var/spool/cron/crontabs/postgres
vim /var/log/postgresql/walg_delete.log
time wal-g backup-push /var/lib/postgresql/14/main


# Интервал раз в 10 минут. разделитель косая черта - "/":
echo "*/10 * * * * /usr/local/bin/wal-g backup-push /var/lib/postgresql/14/main" >> /var/spool/cron/crontabs/postgres





```
