# PATRONI+pg_basebackup
```
# Что бы patroni умел сохранять WAL- файлы необходимо изменить конфиг patroni.yml.demo
parametrs:
  archive_mode: on
  archive_command: /usr/local/bin/copy_wal.sh %p %f
  archive_timeout: 600s
  unix_socket_directories: '/var/run/postgresql/'
  port: 5432

```
###### Правка copy_wal.sh -d (декомпрессия)
```



```
###### 2 способа потоковой архивации:
```
# 1. вкл параметр и через скрипт copy_wal.sh
# 2. вкл потоковая репликация, работает по умолчанию в patroni.





```












