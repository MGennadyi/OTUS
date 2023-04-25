
```
root@zabbix:/home/mgb# ps -fu postgres
UID          PID    PPID  C STIME TTY          TIME CMD
postgres     583       1  0 15:58 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
```
```
vim /etc/postgresql/14/main/postgresql.conf
log_filename = 'postgresql-%a.log'
log_filename = 'postgresql-%u.log'
mkdir -p /log/pg_log
chown -R postgres:postgres /log/pg_log
ALTER SYSTEM SET logging_collector = 'on';
ALTER SYSTEM SET log_rotation_size = '0';
ALTER SYSTEM SET log_rotation_age = '1d';
ALTER SYSTEM SET log_directory = '/log/pg_log';
ALTER SYSTEM SET log_rotation_size = "10";
ALTER SYSTEM SET log_truncate_on_rotation = "on";

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
    copytruncate
}
```
```
vim /postgres/scripts/rotsize.sh
chmod +x /postgres/scripts/rotsize.sh
```
```
chown postgres:postgres /postgres/scripts/logrotate.conf
chown postgres:postgres /postgres/scripts/rotsize.sh
```
```
crontab -e
# Сжатие логов
0 */1 * * * /postgres/scripts/logrotate /postgres/scripts/logrotate.conf --state /postgres/scripts/logrotate-state
# Удаление логов
5 */1 * * * /postgres/scripts/rotsize.sh
```
### Импорт логов
```
COPY postgres_log FROM '/full/path/to/logfile.csv' WITH csv;
```
### Настройка лога:
```
postgres=# select name, setting from pg_settings where name like '%log%';
               name                |       setting
-----------------------------------+----------------------
 log_autovacuum_min_duration       | -1
 log_checkpoints                   | off
 log_connections                   | off
 log_destination                   | stderr
 log_directory                     | /log/pg_log
 log_disconnections                | off
 log_duration                      | off
 log_error_verbosity               | default
 log_executor_stats                | off
 log_file_mode                     | 0600
 log_filename                      | postgresql-%u-%H.log
 log_hostname                      | off
 log_line_prefix                   | %m [%p] %q%u@%d
 log_lock_waits                    | off
 log_min_duration_sample           | -1
 log_min_duration_statement        | -1
 log_min_error_statement           | error
 log_min_messages                  | warning
 log_parameter_max_length          | -1
 log_parameter_max_length_on_error | 0
 log_parser_stats                  | off
 log_planner_stats                 | off
 log_recovery_conflict_waits       | off
 log_replication_commands          | off
 log_rotation_age                  | 1440
 log_rotation_size                 | 10
 log_statement                     | all
 log_statement_sample_rate         | 1
 log_statement_stats               | off
 log_temp_files                    | -1
 log_timezone                      | Europe/Moscow
 log_transaction_sample_rate       | 0
 log_truncate_on_rotation          | on
 logging_collector                 | on
 logical_decoding_work_mem         | 65536
 max_logical_replication_workers   | 4
 syslog_facility                   | local0
 syslog_ident                      | postgres
 syslog_sequence_numbers           | on
 syslog_split_messages             | on
 wal_log_hints                     | off
(41 строка)
```
### Настройка fstab:
```
vim /etc/fstab
# Для временных файлов:
tmpfs /tempdb tmpfs size=1G,uid=postgres,gid=postgres 0 0
# Для логов:
tmpfs /log tmpfs size=1M,uid=postgres,gid=postgres 0 0
```
```
ALTER SYSTEM SET stats_temp_directory = '/tempdb';
```
```
chown -R postgres:postgres /log
```
```
logrotation_size = '1M' - переключится на другой лог
rotation size =10k сработает сжатие
ls -lhr /log/pg_log
-rw------- 1 postgres postgres 1,1M апр 24 18:31 zabbix-2023-04-24_182507.log
```
### Локали:
```
decimal_point "<U002C>" на decimal_point "<U002E>"
include "kpdl(comma) на //include "kpdl(comma)"
decimal_point   "<U002E>"
decimal_point   "<U002C>"
mon_decimal_point "<U002E>"

vim /usr/share/i18n/locales/ru_RU
sudo dpkg-reconfigure locales
```








