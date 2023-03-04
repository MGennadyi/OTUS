# Установка postgrespro-std-13
```
root@etcd:/home/mgb# curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-13/keys/pgpro-repo-add.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 18239  100 18239    0     0  26743      0 --:--:-- --:--:-- --:--:-- 26743

root@etcd:/home/mgb# sh pgpro-repo-add.sh
--2023-03-03 19:53:48--  http://repo.postgrespro.ru/std-13/debian/dists/bullseye/main/binary-amd64/Release
Распознаётся repo.postgrespro.ru (repo.postgrespro.ru)… 213.171.56.11
Подключение к repo.postgrespro.ru (repo.postgrespro.ru)|213.171.56.11|:80... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 137 [application/octet-stream]
Сохранение в: «STDOUT»
-                                                  100%[================================================================================================================>]     137  --.-KB/s    за 0s

/2023-03-03 19:53:49 (9,95 MB/s) - записан в stdout [137/137]

Сущ:1 http://deb.debian.org/debian bullseye InRelease
Сущ:2 http://security.debian.org/debian-security bullseye-security InRelease
Сущ:3 http://deb.debian.org/debian bullseye-updates InRelease
Пол:4 http://repo.postgrespro.ru/std-13/debian bullseye InRelease [3 560 B]
Пол:5 http://repo.postgrespro.ru/std-13/debian bullseye/main amd64 Packages [11,6 kB]
Получено 15,1 kB за 2с (7 276 B/s)
Чтение списков пакетов… Готово

apt-get update
apt-get install postgrespro-std-13 -y
```
### Установка v_14
```
curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
root@etcd:/home/mgb# curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 18239  100 18239    0     0  25981      0 --:--:-- --:--:-- --:--:-- 25944


root@etcd:/home/mgb# sh pgpro-repo-add.sh
--2023-03-03 19:58:04--  http://repo.postgrespro.ru/std-14/debian/dists/bullseye/main/binary-amd64/Release
Распознаётся repo.postgrespro.ru (repo.postgrespro.ru)… 213.171.56.11
Подключение к repo.postgrespro.ru (repo.postgrespro.ru)|213.171.56.11|:80... соединение установлено.
HTTP-запрос отправлен. Ожидание ответа… 200 OK
Длина: 137 [application/octet-stream]
Сохранение в: «STDOUT»

-                                                  100%[================================================================================================================>]     137  --.-KB/s    за 0s

/2023-03-03 19:58:05 (11,1 MB/s) - записан в stdout [137/137]

Сущ:1 http://security.debian.org/debian-security bullseye-security InRelease
Сущ:2 http://deb.debian.org/debian bullseye InRelease
Сущ:3 http://deb.debian.org/debian bullseye-updates InRelease
Сущ:4 http://repo.postgrespro.ru/std-13/debian bullseye InRelease
Пол:5 http://repo.postgrespro.ru/std-14/debian bullseye InRelease [3 560 B]
Пол:6 http://repo.postgrespro.ru/std-14/debian bullseye/main amd64 Packages [12,6 kB]
Получено 16,1 kB за 2с (7 296 B/s)
Чтение списков пакетов… Готово

apt-get update
apt-get install postgrespro-std-14
# Новая версия 14 установлена.
```
```
oot@etcd:/home/mgb# systemctl status postgrespro-std-13
● postgrespro-std-13.service - Postgres Pro std 13 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-std-13.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-03-02 11:32:23 MSK; 1 day 7h ago
    Process: 474 ExecStartPre=/opt/pgpro/std-13/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 490 (postgres)
      Tasks: 8 (limit: 4662)
     Memory: 62.1M
        CPU: 13.473s
     CGroup: /system.slice/postgrespro-std-13.service
             ├─490 /opt/pgpro/std-13/bin/postgres -D /var/lib/pgpro/std-13/data
             ├─520 postgres: logger
             ├─526 postgres: checkpointer
             ├─527 postgres: background writer
             ├─528 postgres: walwriter
             ├─529 postgres: autovacuum launcher
             ├─530 postgres: stats collector
             └─531 postgres: logical replication launcher

мар 02 11:32:17 etcd systemd[1]: Starting Postgres Pro std 13 database server...
мар 02 11:32:23 etcd postgres[490]: 2023-03-02 11:32:23.737 MSK [490] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
мар 02 11:32:23 etcd postgres[490]: 2023-03-02 11:32:23.737 MSK [490] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
мар 02 11:32:23 etcd systemd[1]: Started Postgres Pro std 13 database server.
```
### Установка make
```
apt install gmake
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
E: Невозможно найти пакет gmake
root@etcd:/home/mgb# sudo apt install make
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Предлагаемые пакеты:
  make-doc
Следующие НОВЫЕ пакеты будут установлены:
  make
Обновлено 0 пакетов, установлено 1 новых пакетов, для удаления отмечено 0 пакетов, и 125 пакетов не обновлено.
Необходимо скачать 396 kB архивов.
После данной операции объём занятого дискового пространства возрастёт на 1 630 kB.
Пол:1 http://deb.debian.org/debian bullseye/main amd64 make amd64 4.3-4.1 [396 kB]
Получено 396 kB за 0с (1 135 kB/s)
Выбор ранее не выбранного пакета make.
(Чтение базы данных … на данный момент установлен 104561 файл и каталог.)
Подготовка к распаковке …/make_4.3-4.1_amd64.deb …
Распаковывается make (4.3-4.1) …
Настраивается пакет make (4.3-4.1) …
Обрабатываются триггеры для man-db (2.9.4-2) …
```
#### Установка pg_repack:
```
wget http://api.pgxn.org/dist/pg_repack/1.4.5/pg_repack-1.4.5.zip
unzip pg_repack-1.4.5.zip
root@etcd:/home/mgb/pg_repack-1.4.8# make
make[1]: вход в каталог «/home/mgb/pg_repack-1.4.8/bin»
Makefile:34: /usr/lib/postgresql/13/lib/pgxs/src/makefiles/pgxs.mk: Нет такого файла или каталога
make[1]: *** Нет правила для сборки цели «/usr/lib/postgresql/13/lib/pgxs/src/makefiles/pgxs.mk».  Останов.
make[1]: выход из каталога «/home/mgb/pg_repack-1.4.8/bin»
make: *** [Makefile:35: all] Ошибка 2

# По другому:
apt-get install pgxnclient libpq-dev -y
# PATH=/usr/lib/postgresql/14/bin:$PATH
PATH=/opt/pgpro/std-13/bin:$PATH
root@etcd:/home/mgb# pgxn install pg_repack
INFO: best version: pg_repack 1.4.8
INFO: saving /tmp/tmpufzy46wd/pg_repack-1.4.8.zip
INFO: unpacking: /tmp/tmpufzy46wd/pg_repack-1.4.8.zip
INFO: building extension
ERROR: make executable not found: gmake



```
### Остановка v_13
```
systemctl stop postgrespro-std-13
```
# Подготовка директорий:
```
mkdir -p /data/pg_data
mkdir -p /wal/pg_wal
```
# Инициалицация нового кластера v_14:
```
/opt/pgpro/std-14/bin/pg_setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data/pg_data --waldir=/wal/pg_wal

```
### pg_repack
```
CREATE EXTENSION pg_repack;
```

# Проверка перед обновлением:
```
mkdir -p /pg_upgrade
cd /pg_upgrade
/opt/pgpro/std-14/bin/pg_upgrade

```
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt install libpq-dev

vim /etc/apt/sources.list
deb http://ftp.de.debian.org/debian buster main
deb http://ftp.de.debian.org/debian bullseye main 
````























