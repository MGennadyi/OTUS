
```
# Что под капотом:
lsb_release -a
pg_lsclusters
root@etcd:/home/mgb# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```
```
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
##### 1. Остановка потоковой архивации
```

```











