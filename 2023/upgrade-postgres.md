# Мажерное обновление POSTGRESQL
###### После создания ВМ и уст.postgres создать pg_stat_statements
```
ALTER SYSTEM set shared_preload_libraries = 'pg_stat_statements';
create extension pg_stat_statements;
# Необходимо перечитать конфигурацию, но лучше перестартовать:
psql -c "SELECT pg_reload_conf();"
pg_ctlcluster 13 main stop
pg_ctlcluster 13 main start
systemctl disable  postgresql
```
### Загрузка БД:
```
wget --quiet https://edu.postgrespro.ru/demo_small.zip
unzip demo_small.zip
ls -la
chown postgres /home/mgb/demo_small.sql
psql -c 'CREATE DATABASE demo';
psql -d demo < /home/mgb/demo_small.sql
\c demo
Вы подключены к базе данных "demo" как пользователь "postgres".
demo=# create extension pg_repack; - уст на конкретную БД, к примеру demo.
CREATE EXTENSION
```
##### 0. Проверим, что под капотом:
```
# Для postgres-pro: psql -c "select pgpro_version()"
postgres@zabbix:/home/mgb$ psql -c "select version()"
                                                           version
-----------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.4 (Debian 14.4-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
postgres@etcd:/home/mgb$ psql -c "select version()"
                                                          version
---------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 13.9 (Debian 13.9-0+deb11u1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(1 строка)

root@etcd:/home/mgb# lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 11 (bullseye)
Release:        11
Codename:       bullseye
-------------------
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
-------------------
root@etcd:/home/mgb# df -h
Файловая система Размер Использовано  Дост Использовано% Cмонтировано в
udev               1,9G            0  1,9G            0% /dev
tmpfs              394M         984K  393M            1% /run
/dev/sda1          6,9G         3,8G  2,8G           59% /
tmpfs              2,0G          16K  2,0G            1% /dev/shm
tmpfs              5,0M         4,0K  5,0M            1% /run/lock
tmpfs              394M          48K  394M            1% /run/user/1000
tmpfs              394M          44K  394M            1% /run/user/114
``` 
##### 00. Подготовка до простоя:
```
# Создать задание 2-го уровня на unix:
1. Подключить репозитории. 
# Просмотр доступных пакетов ДО:
apt list | grep postgresql
# mcedit /etc/apt/sources.list.d/pg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
# Обнавляем!!!
apt update
# Просмотр доступных пакетов ПОСЛЕ:
Проверим:
apt list | grep  postgresql
postgresql-pltcl-10-dbgsym/bullseye-pgdg 10.23-1.pgdg110+1 amd64
postgresql-pltcl-11-dbgsym/bullseye-pgdg 11.19-1.pgdg110+1 amd64
postgresql-pltcl-12-dbgsym/bullseye-pgdg 12.14-1.pgdg110+1 amd64
postgresql-pltcl-13-dbgsym/bullseye-pgdg 13.10-1.pgdg110+1 amd64
postgresql-pltcl-14-dbgsym/bullseye-pgdg 14.7-1.pgdg110+1 amd64
postgresql-pltcl-15-dbgsym/bullseye-pgdg 15.2-1.pgdg110+1 amd64
# Обновиться можно.
2. Предоставить права root
```
##### 0. Остановка потоковой архивации (если есть):
```
# Из-под перс.УЗ
sudo systemctl stop  pg_receivewal.core-s-pgaidb01.service
systemctl status pg_receivewal.core-s-pgaidb01.service
```
#### 0. Архивация собранных WAL 
```
sudo -i -u postgres
2 скрипта
```
#### 1. Создание режим обслуживания  в ZABBIX:
```
192.168.0.19:8080/zabbix.php
обслуживание/создать период (без сбора данных)
```

#### 2. Отключаем пользователей от баз данных:
```
# 1.0 Проверка клиентских подключений:
psql -c "SELECT count(*) FROM pg_stat_activity WHERE backend_type='client backend'"
postgres@etcd:/home/mgb$ psql -c "SELECT usename FROM pg_stat_activity WHERE backend_type='client backend'"
 usename
----------
 postgres
(1 строка)
-----------------
postgres@etcd:/home/mgb$ ps -fu postgres
UID          PID    PPID  C STIME TTY          TIME CMD
postgres     535       1  0 16:13 ?        00:00:03 /usr/lib/postgresql/13/bin/postgres -D /var/lib/postgresql/13/main -c config_file=/etc/postgresql/13/main/postgresql.conf
postgres     537     535  0 16:13 ?        00:00:00 postgres: 13/main: checkpointer
postgres     538     535  0 16:13 ?        00:00:00 postgres: 13/main: background writer
postgres     539     535  0 16:13 ?        00:00:00 postgres: 13/main: walwriter
postgres     540     535  0 16:13 ?        00:00:00 postgres: 13/main: autovacuum launcher
postgres     541     535  0 16:13 ?        00:00:00 postgres: 13/main: stats collector
postgres     542     535  0 16:13 ?        00:00:00 postgres: 13/main: logical replication launcher
postgres   77075   77074  0 19:37 pts/0    00:00:00 bash
postgres   77692   77075  0 19:38 pts/0    00:00:00 ps -fu postgres
```
```
# 1.1 Подготовка и замена предварительно откоректрированного pg_hba.conf
mkdir -p /data/backup/24.02.2022
chown -R postgres:postgres /data/backup
cp /etc/postgresql/13/main/pg_hba.conf /data/backup/24.02.2022/pg_hba.conf-old-cluster
cp /etc/postgresql/13/main/pg_hba.conf /data/backup/24.02.2022/pg_hba.conf-user-off
mcedit /data/backup/24.02.2022/pg_hba.conf-user-off
# TYPE  DATABASE        USER            ADDRESS                 METHOD
## Обслуживание СУБД:

local   all             postgres                                peer
host    all             postgres             127.0.0.1/32       md5
local   replication     postgres                                peer

cp /backup/24.02.2022/pg_hba.conf-user-off /etc/postgresql/13/main/pg_hba.conf
psql -c "SELECT pg_reload_conf()"
# Отключить клиентские соединения:
postgres@zabbix:/home/mgb$ psql -c "select pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname <> 'postgres'"
 pg_terminate_backend
----------------------
 t
(28 строк)
# 1.2 Проверка клиентских подключений:
psql -c "SELECT count(*) FROM pg_stat_activity WHERE backend_type='client backend'"
 count
-------
     2
(1 строка)
# Потому что открыто 2 сессии
```
#### 3. Отключаем задания в планировщике:
```
sudo -i -u postgres
# Закоментрировать:
crontab -e
```
#### 4. Запуск резервного копирования:
```
mkdir -p /postgres/scripts
chown -R postgres:postgres /postgres/scripts
vim /postgres/scripts/atom_basebackup.sh
vim /postgres/scripts/create_arwd_role.sh
chmod +x /postgres/scripts/atom_basebackup.sh
chmod +x /postgres/scripts/create_arwd_role.sh
su postgres
/postgres/scripts/atom_basebackup.sh
/postgres/scripts/create_arwd_role.sh
```
###### Рекомендации: pg_bouncer на паузу; checkpoint
#### 5. Остановка СУБД
```
systemctl status postgresql
# из каталога бинарника: /usr/bin
pg_ctlcluster 14 main status
# Ответ: pg_ctl: сервер работает (PID: 545)
/usr/lib/postgresql/14/bin/postgres "-D" "/var/lib/postgresql/14/main" "-c" "config_file=/etc/postgresql/14/main/postgresql.conf"

systemctl stop postgresql
# из каталога бинарника: /usr/lib/postgresql/14/bin
pg_ctl -D /data/pg_data stop
pg_ctl "-D" "/var/lib/postgresql/14/main" stop - не работает
----------------------
pg_ctlcluster 13 main status
pg_ctlcluster 13 main stop
systemctl stop postgresql@13-main
systemctl disable postgresql@13-main
```
#### 6. Установка пакетов новой версии
```
pg_ctlcluster 13 main stop
apt install postgresql-14 -y
apt ibstall postgresql-15 -y
pg_lsclusters
# На каком порту v_14 ???
# Если на 5433 то правим ниже:
```
#### Пропускаем перенос данных старого кластера
```
# Из-под УЗ postgres: sudo su -i -u postgres
mkdir -p /log/pg_log_13
mv /var/lib/postgresql/13/main /var/lib/postgresql/13/main_13
chmod 700 /var/lib/postgresql/13/main_13
mkdir -p /data/pg_data_13
mv /var/lib/postgresql/13/main /data/pg_data_13
# mv /data/pg_data/ /data/pg_data_13
chmod 700 /data/pg_data_13 
rm /data/pg_data_13/pg_wal
mv /wal/pg_wal
mv /log/pg_log
```
#### Пропускаем инициализация нового кластера
```
# PG_PRO
/usr/lib/postgresql/14/bin/pg-setup initdb --datachecksums --locale=en_US.utf.8 --pgdata=/data/pg_data --waldir=/wal/pg_wal
/opt/pgpro/ent-14/bin/pg-setup initdb --datachecksums --locale=en_US.utf.8 --pgdata=/data/pg_data --waldir=/wal/pg_wal
/opt/pgpro/ent-14/bin/pg-setup initdb --datachecksums --locale=en_US.utf.8 --pgdata=/data/pg_data --waldir=/wal/pg_wal
# Ванильный из пакетов не требует инициализации
pg_upgrade -b /usr/lib/postgresql/13/bin -B /usr/lib/postgresql/14/bin -d /var/lib/postgresql/13/main -D /var/lib/postgresql/14/main [option...]
```
#### 7. Копирование конфигов из РК
```
mkdir -p /pg_upgrade/v_13
chown -R postgres:postgres /pg_upgrade
cp /etc/postgresql/13/main/postgresql.conf /pg_upgrade/postgresql.conf
cp /var/lib/postgresql/13/main /pg_upgrade/postgresql.auto.conf
```
### Проверка выполнения обновления: --check:
#### V_13 и V_14 должны быть остановлены:
```
# v_13 должен быть уже остановлен:
root@etcd:/home/mgb# pg_ctlcluster 13 main status
root@etcd:/home/mgb# pg_ctlcluster 14 main stop
mkdir -p /pg_upgrade/1
chown -R postgres:postgres /pg_upgrade
sudo -u postgres -i
cd /pg_upgrade
#                          pg_upgrade -b старый_каталог_bin         -B новый_каталог_bin]         -d старый_каталог_конфигурации -D новый_каталог_конфигурации
postgres@pg:~$ /usr/lib/postgresql/14/bin/pg_upgrade -b /usr/lib/postgresql/13/bin -B /usr/lib/postgresql/14/bin -d /etc/postgresql/13/main/ -D /etc/postgresql/14/main/ --link --check
cd /pg_upgrade - обязательно! иначе:
не удалось открыть файл протокола "pg_upgrade_internal.log": Отказано в доступе
var/lib/postgresql/
Finding the real data directory for the source cluster      ok
Finding the real data directory for the target cluster      ok
Проведение проверок целостности
-------------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Checking for user-defined encoding conversions              ok
Checking for user-defined postfix operators                 ok
Checking for incompatible polymorphic functions             ok
Checking for presence of required libraries                 сбой

В вашей инсталляции есть ссылки на загружаемые библиотеки, отсутствующие
в новой инсталляции. Вы можете добавить эти библиотеки в новую инсталляцию
или удалить функции, использующие их, из старой. Список проблемных
библиотек приведён в файле:
    loadable_libraries.txt

Ошибка, выполняется выход
```
### Шеф, все пропало.
```
# Смотрим:
postgres@etcd:/pg_upgrade$ cat loadable_libraries.txt
загрузить библиотеку "$libdir/pg_repack" не удалось: ОШИБКА:  нет доступа к файлу "$libdir/pg_repack": Нет такого файла или каталога
В базе данных: demo
pg_ctlcluster 13 main start
su postgres
psql # Ругался на demo, значить:
\c demo
demo=# drop extension pg_repack;
DROP EXTENSION
```
#### Прверка можно на не выкл. кластере:
```
postgres@etcd:/pg_upgrade$ /usr/lib/postgresql/14/bin/pg_upgrade -b /usr/lib/postgresql/13/bin -B /usr/lib/postgresql/14/bin -d /etc/postgresql/13/main/ -D /etc/postgresql/14/main/ --link --check
Finding the real data directory for the source cluster      ok
Finding the real data directory for the target cluster      ok
Проверка целостности на старом работающем сервере
-------------------------------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Checking for user-defined encoding conversions              ok
Checking for user-defined postfix operators                 ok
Checking for incompatible polymorphic functions             ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok
Checking for new cluster tablespace directories             ok

*Кластеры совместимы*  -О чудо!!! Приступаем к обновлению!  Параметр --check - убираем.
```
### Стоп всех СУБД:
```
pg_lsclusters
pg_ctlcluster 13 main stop
root@etcd:/home/mgb/pg_repack-1.4.5# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 down   postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
14  main    5433 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

#### Выполнение обновления:
```
postgres@etcd:/pg_upgrade$ /usr/lib/postgresql/14/bin/pg_upgrade -b /usr/lib/postgresql/13/bin -B /usr/lib/postgresql/14/bin -d /etc/postgresql/13/main/ -D /etc/postgresql/14/main/ --link
Finding the real data directory for the source cluster      ok
Finding the real data directory for the target cluster      ok
Проведение проверок целостности
-------------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Checking for user-defined encoding conversions              ok
Checking for user-defined postfix operators                 ok
Checking for incompatible polymorphic functions             ok
Creating dump of global objects                             ok
Creating dump of database schemas
                                                            ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok
Checking for new cluster tablespace directories             ok

Если работа pg_upgrade после этого прервётся, вы должны заново выполнить initdb
для нового кластера, чтобы продолжить.

Выполнение обновления
---------------------
Analyzing all rows in the new cluster                       ok
Freezing all rows in the new cluster                        ok
Deleting files from new pg_xact                             ok
Copying old pg_xact to new server                           ok
Setting oldest XID for new cluster                          ok
Setting next transaction ID and epoch for new cluster       ok
Deleting files from new pg_multixact/offsets                ok
Copying old pg_multixact/offsets to new server              ok
Deleting files from new pg_multixact/members                ok
Copying old pg_multixact/members to new server              ok
Setting next multixact ID and offset for new cluster        ok
Resetting WAL archives                                      ok
Setting frozenxid and minmxid counters in new cluster       ok
Restoring global objects in the new cluster                 ok
Restoring database schemas in the new cluster
                                                            ok
Adding ".old" suffix to old global/pg_control               ok

Если вы захотите запустить старый кластер, вам нужно будет убрать
расширение ".old" у файла /var/lib/postgresql/13/main/global/pg_control.old.
Так как применялся режим "ссылок", работа старого кластера
после того, как будет запущен новый, не гарантируется.

Подключение файлов пользовательских отношений ссылками
                                                            ok
Setting next OID for new cluster                            ok
Sync data directory to disk                                 ok
Creating script to delete old cluster                       ok
Checking for extension updates                              notice

В вашей инсталляции есть расширения, которые надо обновить
командой ALTER EXTENSION. Скрипт
    update_extensions.sql
будучи выполненным администратором БД в psql, обновит все
эти расширения.

Обновление завершено
--------------------
Статистика оптимизатора утилитой pg_upgrade не переносится.
Запустив новый сервер, имеет смысл выполнить:
    /usr/lib/postgresql/14/bin/vacuumdb --all --analyze-in-stages

При запуске этого скрипта будут удалены файлы данных старого кластера:
    ./delete_old_cluster.sh

```
#### Стартуем новый кластер
```
pg_ctlcluster 14 main start
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 down   postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
14  main    5433 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
Новый инстанс работает на порту 5433
```
#### Правим postgresql.conf установленной 14 версии:
```
vim /etc/postgresql/14/main/postgresql.conf
root@etcd:/home/mgb# pg_ctlcluster 14 main stop
root@etcd:/home/mgb# pg_ctlcluster 14 main start
root@etcd:/home/mgb# pg_ctlcluster 14 main status
pg_ctl: сервер работает (PID: 32476)
/usr/lib/postgresql/14/bin/postgres "-D" "/var/lib/postgresql/14/main" "-c" "config_file=/etc/postgresql/14/main/postgresql.conf"
```
#### Выполнение рекомендаций pg_upgrade:
#### 1. update_extensions.sql через пересоздание расширения:
```
psql
drop extension pg_stat_statements;
create extension pg_stat_statements;
```
#### 2. vacuumdb --all --analyze
```
/usr/lib/postgresql/14/bin/vacuumdb --all --analyze-in-stages
postgres@etcd:/home/mgb$ /usr/lib/postgresql/14/bin/vacuumdb --all --analyze-in-stages
vacuumdb: обработка базы данных "demo": Вычисление минимальной статистики для оптимизатора (1 запись)
vacuumdb: обработка базы данных "postgres": Вычисление минимальной статистики для оптимизатора (1 запись)
vacuumdb: обработка базы данных "template1": Вычисление минимальной статистики для оптимизатора (1 запись)
vacuumdb: обработка базы данных "demo": Вычисление средней статистики для оптимизатора (10 записей)
vacuumdb: обработка базы данных "postgres": Вычисление средней статистики для оптимизатора (10 записей)
vacuumdb: обработка базы данных "template1": Вычисление средней статистики для оптимизатора (10 записей)
vacuumdb: обработка базы данных "demo": Вычисление стандартной (полной) статистики для оптимизатора
vacuumdb: обработка базы данных "postgres": Вычисление стандартной (полной) статистики для оптимизатора
vacuumdb: обработка базы данных "template1": Вычисление стандартной (полной) статистики для оптимизатора
```
#### 3. Удаление старого кластера
```
postgres@etcd:/pg_upgrade$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 down   postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
14  main    5433 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

/pg_upgrade/delete_old_cluster.sh
# Проверяем:
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
13  main    5432 down   <unknown> /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
14  main    5432 online postgres  /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
# Не до конца:
root@etcd:/home/mgb# pg_dropcluster 13 main
Warning: corrupted cluster: data directory does not exist
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```
#### Проверим БД demo:
```
postgres=# \l+
                                                                        Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |     Права доступа     | Размер  | Табл. пространство |                  Описание
-----------+----------+-----------+-------------+-------------+-----------------------+---------+--------------------+--------------------------------------------
 demo      | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |                       | 281 MB  | pg_default         |
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |                       | 8769 kB | pg_default         | default administrative connection database
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +| 8593 kB | pg_default         | unmodifiable empty database
           |          |           |             |             | postgres=CTc/postgres |         |                    |
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | postgres=CTc/postgres+| 8713 kB | pg_default         | default template for new databases
           |          |           |             |             | =c/postgres           |         |                    |
(4 строки)
demo=# \dn+
                             Список схем
   Имя    | Владелец |    Права доступа     |        Описание
----------+----------+----------------------+------------------------
 bookings | postgres |                      | Авиаперевозки
 public   | postgres | postgres=UC/postgres+| standard public schema
          |          | =UC/postgres         |
(2 строки)
```
#### Удаляем старый кластер:
```
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
13  main    5432 down   <unknown> /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
14  main    5432 online postgres  /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

```
#### Автозапуск при старте:
```
root@etcd:/home/mgb# systemctl enable postgresql
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /lib/systemd/system/postgresql.service.
```



















