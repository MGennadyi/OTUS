# Над пропастью WAL-G.
```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
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

    "WALG_DELTA_MAX_STEPS": "3",

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

    "WALG_DELTA_MAX_STEPS": "3",

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
###### 7. Создадим тестовую базу данных с данными:
```
# Включаем показ тайминга выполнения команд:
\timing
psql -c "CREATE DATABASE otus1;"
psql otus -c "create table test(i int);"
psql otus -c "insert into test values (10), (20), (30);"
psql otus -c "select * from test;"
```
###### 6. Делаем каталог для бекапов:
```
sudo mkdir /home/backups/
chown -R postgres /home/backups/
root@backup:/home/mgb# ls -la /home/
drwxr-xr-x  4 root     root     4096 дек  6 17:32 .
drwxr-xr-x 19 root     root     4096 мар 21  2022 ..
drwxr-xr-x  2 postgres postgres 4096 дек  6 17:32 backups
drwxr-xr-x 14 mgb      mgb      4096 дек  6 17:19 mgb
```
###### 6. Делаем backup-push:
```
su postgres
time wal-g backup-push /var/lib/postgresql/14/main
# Ответ: Couldn't find previous backup.
```
```
wal-g backup-list
wal-g backup-list --pretty
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
# Удаляем, если есть: 
pg_dropcluster 14 main2
Error: This cluster is still running. Stop it or supply the --stop option
pg_ctlcluster 14 main2 stop
pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
pg_createcluster 14 main2
# Ответ:
Warning: systemd does not know about the new cluster yet. Operations like "service postgresql start" will not handle it. To fix, run:
sudo systemctl daemon-reload
Ver Cluster Port Status Owner    Data directory               Log file
14  main2   5433 down   postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
pg_lsclusters
# Ответ:
14  main    5432 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main2   5433 down   postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
pg_ctlcluster 14 main2 start
# Ответ:
14  main    5432 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main2   5433 online postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log
pg_ctlcluster 14 main2 stop
# sudo systemctl start postgresql@14-main2
# sudo systemctl stop postgresql@14-main2
# Удаляем содержимое main2 :
rm -rf /var/lib/postgresql/14/main2/*
# А что с бекапами? :
wal-g backup-fetch /var/lib/postgresql/14/main2 LATEST
# Ответ: Backup extraction complete.
sudo systemctl daemon-reload
pg_ctlcluster 14 main2 start
root@wal-g2:/home/mgb# pg_ctlcluster 14 main2 start
Job for postgresql@14-main2.service failed because the service did not take the steps required by its unit configuration.
See "systemctl status postgresql@14-main2.service" and "journalctl -xe" for details.

# Смотрим журнал:
tail /var/lib/postgresql/14/main/log/restore_command.log
journalctl -xe
дек 06 17:45:30 backup postgresql@14-main2[1162]: Error: /usr/lib/postgresql/14/bin/pg_ctl /usr/lib/postgresql/14/bin/pg_ctl start -D /var/lib/postgresql/14/ma>
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.278 MSK [1167] СООБЩЕНИЕ:  запускается PostgreSQL 14.6 (Debian 14.6-1.pgdg110+1) on x86_6>
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.279 MSK [1167] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::1" открыт порт 5433
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.279 MSK [1167] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "127.0.0.1" открыт порт >
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.279 MSK [1167] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/>
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.283 MSK [1168] СООБЩЕНИЕ:  работа системы БД была прервана; последний момент работы: 2022>
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.604 MSK [1168] СООБЩЕНИЕ:  неверная запись контрольной точки
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.604 MSK [1168] ВАЖНО:  не удалось считать нужную запись контрольной точки
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.604 MSK [1168] ПОДСКАЗКА:  Если вы восстанавливаете резервную копию, создайте "/var/lib/p>
дек 06 17:45:30 backup postgresql@14-main2[1162]:         В других случаях попытайтесь удалить файл "/var/lib/postgresql/14/main2/backup_label".
дек 06 17:45:30 backup postgresql@14-main2[1162]:         Будьте осторожны: при восстановлении резервной копии удаление "/var/lib/postgresql/14/main2/backup_la>
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.605 MSK [1167] СООБЩЕНИЕ:  стартовый процесс (PID 1168) завершился с кодом выхода 1
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.605 MSK [1167] СООБЩЕНИЕ:  прерывание запуска из-за ошибки в стартовом процессе
дек 06 17:45:30 backup postgresql@14-main2[1162]: 2022-12-06 17:45:30.607 MSK [1167] СООБЩЕНИЕ:  система БД выключена
дек 06 17:45:30 backup postgresql@14-main2[1162]: pg_ctl: не удалось запустить сервер
дек 06 17:45:30 backup postgresql@14-main2[1162]: Изучите протокол выполнения.
# Не поможет:
sudo systemctl daemon-reload
pg_ctlcluster 14 main2 start
# Контрольные суммы:



su postgres
/usr/lib/postgresql/14/bin/pg_ctl restart -D /var/lib/postgresql/14/main2
pg_ctl: файл PID "/var/lib/postgresql/14/main2/postmaster.pid" не существует
Запущен ли сервер?
производится попытка запуска сервера в любом случае
ожидание запуска сервера....(pid=169899) СООБЩЕНИЕ:  запускается PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(pid=169899) СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::1" открыт порт 5433
(pid=169899) СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "127.0.0.1" открыт порт 5433
(pid=169899) СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5433"
(pid=169900) СООБЩЕНИЕ:  работа системы БД была прервана; последний момент работы: 2022-12-10 18:22:01 MSK
(pid=169900) СООБЩЕНИЕ:  неверная запись контрольной точки
(pid=169900) ВАЖНО:  не удалось считать нужную запись контрольной точки
(pid=169900) ПОДСКАЗКА:  Если вы восстанавливаете резервную копию, создайте "/var/lib/postgresql/14/main2/recovery.signal" и задайте обязательные параметры восстановления.
        В других случаях попытайтесь удалить файл "/var/lib/postgresql/14/main2/backup_label".
        Будьте осторожны: при восстановлении резервной копии удаление "/var/lib/postgresql/14/main2/backup_label" приведёт к повреждению кластера.
(pid=169899) СООБЩЕНИЕ:  стартовый процесс (PID 169900) завершился с кодом выхода 1
(pid=169899) СООБЩЕНИЕ:  прерывание запуска из-за ошибки в стартовом процессе
(pid=169899) СООБЩЕНИЕ:  система БД выключена
 прекращение ожидания
pg_ctl: не удалось запустить сервер
Изучите протокол выполнения.
```
##### Теплое резервирование:
```
touch "/var/lib/postgresql/14/main2/recovery.signal"



```
###### 10. Исправляем отсутствие checksums, инициализацияся на выкл.кластере, из-под postgres:
```
systemctl stop postgresql
su postgres
/usr/lib/postgresql/14/bin/pg_checksums -D /var/lib/postgresql/14/main --enable
exit
systemctl start postgresql
```
###### ПОДСКАЗКА:  Если вы восстанавливаете резервную копию, создайте
```
touch /var/lib/postgresql/14/main2/recovery.signal
vim /var/lib/postgresql/14/main2/recovery.signal
```
###### Получаем: Selecting the latest backup...
```
INFO: 2022/04/09 16:44:03.325798 LATEST backup is: 'base_00000001000000000000001E_D_000000010000000000000006'
INFO: 2022/04/09 16:44:05.834564 Backup extraction complete. Бэкап сформировался за 0,02 сек.
```
##### 10. На точку времени создать файл для восстановления из архивов wal:
```
touch "/var/lib/postgresql/14/main2/recovery.signal"
exit
sudo systemctl start postgresql@14-main2
pg_ctlcluster 14 main2 start
```

##### CRON расписания резервного копирования:
```
# Просмотр заданий:
crontab -l
# Редактирование задание через vim
crontab -e
# Удаление
crontab -r
```
```
#!/bin/bash

echo "15 4 * * *    /usr/local/bin/wal-g backup-push /var/lib/postgresql/14/main >> /var/log/postgresql/walg_backup.log 2>&1" >> /var/spool/cron/crontabs/postgres
# задаем владельца и выставляем правильные права файлу
chown postgres: /var/spool/cron/crontabs/postgres
chmod 600 /var/spool/cron/crontabs/postgres
```
###### Скрипт data.sh добавляет запись Бэкап началася: 
```
touch /home/mgb/data.sh
chmod +x /home/mgb/data.sh
chown postgres: /home/mgb/data.sh
vim /home/mgb/data.sh
-----------------------------
#!/bin/bash
myvar=$(date '+%d-%m-%Y_%H-%M-%S')
echo "Бэкап началася в $myvar" >> /home/backups/test_postgres.txt
touch /home/backups/$myvar.txt
-----------------------------
#!/bin/bash
# Интервал раз в 10 минут. разделитель косая черта - "/":
 echo 'date +\%y-\%m-\%d' > /home/backups/test_postgres.txt
 echo 'date +\%y-\%m-\%d' '%H:%M:%S' > /home/backups/test_postgres.txt
---------------------------
#!/bin/bash
DATE=`date +%d-%m-%y`
date | cut -d " " -f2-4 | tr " " "-" 
datevar=$(date +'%Y-%m-%d : %H-%M')
echo $datevar
```
###### Скрипт backup_wal-g.sh: Создание резервных копий :
```
touch /home/mgb/backup_wal-g.sh
chmod +x /home/mgb/backup_wal-g.sh
chown postgres: /home/mgb/backup_wal-g.sh
vim /home/mgb/backup_wal-g.sh

#!/bin/bash

# Интервал раз в 10 минут. разделитель косая черта - "/":
*/10 * * * * echo 'data +\%y-\%m-\%d' '%H:%M:%S' > /home/backups/test_postgres.txt
*/10 * * * * touch /home/backups/test_postgres.txt
*/10 * * * * wal-g backup-push /var/lib/postgresql/14/main

?
/usr/local/bin/wal-g backup-push /var/lib/postgresql/14/main /var/log/postgresql/walg_delete.log
```
###### Скрипт del_wal-g.sh: Удаление старых резервных копий :
```
touch /home/mgb/del_wal-g.sh
chmod +x /home/mgb/del_wal-g.sh
chown postgres: /home/mgb/del_wal-g.sh
vim /home/mgb/del_wal-g.sh

#!/bin/bash

# Настройка: Время в днях: -mtime
age_d=1
dir="/home/backups/"
find "$dir" -mtime "+$age_d" -delete && find "$dir" -type d -empty -delete
################
# Настройка: Время в минутах: -mmin
age_m=60
dir="/home/backups/"
find "$dir" -mmin "+$age_m" -delete && find "$dir" -type d -empty -delete
```
###### Пример не работает:
```
#!/bin/bash
echo "30 6 * * *    /usr/local/bin/wal-g delete before FIND_FULL \$(date -d '-5 days' '+\\%FT\\%TZ') --confirm >> /var/log/postgresql/walg_delete.log 2>&1" >> /var/spool/cron/crontabs/postgres
```
###### Настраиваем планировщик:
```
su postgres
crontab -e
*/10 * * * * touch /home/backups/test_pg_cr.txt
*/10 * * * * /home/mgb/del_wal-g.sh
*/10 * * * * /home/mgb/backup_wal-g.sh
*/10 * * * * /home/mgb/data.sh
```
```
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




wal-g backup-list --pretty

```
