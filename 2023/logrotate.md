
```
root@zabbix:/home/mgb# ps -fu postgres
UID          PID    PPID  C STIME TTY          TIME CMD
postgres     583       1  0 15:58 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
```
```
vim /etc/postgresql/14/main/postgresql.conf
log_filename = 'postgresql-%u.log'
mkdir -p /log/pg_log
chown -R postgres:postgres /log/pg_log
ALTER SYSTEM SET logging_collector = 'on';
ALTER SYSTEM SET log_rotation_size = '0';
ALTER SYSTEM SET log_rotation_age = '1d';
ALTER SYSTEM SET log_directory = '/log/pg_log';
SELECT pg_reload_conf();
```
