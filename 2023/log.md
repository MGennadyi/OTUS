```
mkdir -p /log/pg_log
chown -R postgres /log/pg_log
chown -R postgres:postgres /log/pg_log
ALTER SYSTEM SET log_directory = '/log/pg_log';
ALTER SYSTEM SET log_filename = 'postgresql-%u.log';
ALTER SYSTEM SET log_filename = 'postgresql.log';
SELECT pg_reload_conf();
```
##### Просмотр лог на ошибки:
```
grep -E "ERROR|ОШИБКА|DETAIL|FATAL" /var/log/postgresql/postgresql-13-main.log

```
# Просмотр директории с данными:
```
postgres=# show data_directory;
       data_directory
-----------------------------
 /var/lib/postgresql/13/main
(1 строка)
```





















