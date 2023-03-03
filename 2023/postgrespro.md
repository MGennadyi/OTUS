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
apt-get install postgrespro-std-13
curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
root@etcd:/home/mgb# curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 18239  100 18239    0     0  25981      0 --:--:-- --:--:-- --:--:-- 25944
root@etcd:/home/mgb# mc
MoTTY X11 proxy: Unsupported authorisation protocol

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
### 
```
systemctl stop postgrespro-std-13
```

```
/opt/pgpro/std-14/bin/pg_setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data/pg_data --waldir=/wal/pg_wal

```



















