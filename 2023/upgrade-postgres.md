# Мажерное обновление POSTGRESQL
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
##### 1. Остановка потоковой архивации (если есть):
```
# Из-под перс.УЗ
sudo systemctl stop  pg_receivewal.core-s-pgaidb01.service
systemctl status pg_receivewal.core-s-pgaidb01.service
```
#### 2. Архивация собранных WAL 
```
sudo -i -u postgres
2 скрипта
```
#### 3. Отключаем задания в планировщике:
```
sudo -i -u postgres
# Закоментрировать:
crontab -e
```
#### 4. Отключаем пользователей:
```
# Проверка клиентских подключений:
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
# Замена предварительно откоректрированного pg_hba.conf
mv /backup/12.02.2023/ph_hba.conf-user-off /etc/postgresql/13/main/pg_hba.conf
psql -c "SELECT pg_reload_conf()"
# Отключить клиентские соединения:
postgres@zabbix:/home/mgb$ psql -c "select pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname <> 'postgres'"
 pg_terminate_backend
----------------------
 t
(28 строк)

```
#### 5. Остановка СУБД
###### Рекомендации: pg_bouncer на паузу; checkpoint
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

```

#### Создание РО в СКМ
```
192.168.0.19:8080/zabbix.php
обслуживание/создать период (без сбора данных)
```
#### Установка пакетов новой версии
```
apt install postgresql-14
apt ibstall postgresql-15
```
#### Перенос данных старого кластера
```
# Из-под УЗ postgres:
sudo su -i -u postgres
mkdir /log/pg_log_11
mv /data/pg_data/ /data/pg_data_11
chmod 700 /data/pg_data_11
rm /data/pg_data_11/pg_wal
mv /wal/pg_wal
mv /log/pg_log
```
#### Инициализация нового кластера
```





```

```
alter extations pg_stat_stations update

```























