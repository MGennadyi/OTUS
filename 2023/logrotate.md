
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

```
root@zabbix:/home/mgb# logrotate --version
logrotate 3.18.0

    Default mail command:       /usr/bin/mail
    Default compress command:   /bin/gzip
    Default uncompress command: /bin/gunzip
    Default compress extension: .gz
    Default state file path:    /var/lib/logrotate/status
    ACL support:                yes
    SELinux support:            yes
```
### Запуск от postgres:
```
cp /usr/sbin/logrotate /postgres/scripts/
chown postgres:postgres /postgres/scripts/logrotate
```
```
vim /postgres/scripts/logrotate.conf
/log/pg_log/*.log
{
    rotate 99
    size 10k
    missingok
    compress
    notifempty
    maxage 30
}
```
```
crontab -e
# compresslog
0 */1 * * * /postgres/scripts/logrotate /postgres/scripts/logrotate.conf --state /postgres/scripts/logrotate-state

```


