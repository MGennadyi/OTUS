# Установка postgrespro-std-13
```
curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-13/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
apt-get update
apt-get install postgrespro-std-14
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





















