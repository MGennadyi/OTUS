
```
root@zabbix:/home/mgb# ps -fu postgres
UID          PID    PPID  C STIME TTY          TIME CMD
postgres     583       1  0 15:58 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
```
### 1. Подготовка директорий
```
mkdir -p /log/pg_log
mkdir -p /log/llog
chown -R postgres:postgres /log
mkdir -p /postgres/scripts/logrotate
chown -R postgres:postgres /postgres
```
### 2. Создание logrotate+logrotate.conf
```
cp /usr/sbin/logrotate /postgres/scripts/logrotate/
vim /postgres/scripts/logrotate.conf
/log/pg_log/*.log -rt|head -n -1|sed 's/^/\/log\/pg_log\//'
{
    rotate 99
    size 500M
    missingok
    compress
    ifempty
    maxage 1
    postgrorate
                /postgres/scripts/logrotate/logclean.sh
    endscript
}
```
### 3. Правка конфигов postgresql.auto
```
ALTER SYSTEM SET log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
ALTER SYSTEM SET log_rotation_size = '500MB'; # вкл ротацию логов по размеру.
ALTER SYSTEM SET logging_collector = 'on'; # логирование в лог СУБД, рестарт СУБД! 
# При нулевом значении смена файлов по времени не производится:
ALTER SYSTEM SET log_rotation_age = '0'; # откл ротацию логов по времени
ALTER SYSTEM SET log_directory = '/log/pg_log';
ALTER SYSTEM SET log_truncate_on_rotation = "on";
ALTER SYSTEM SET log_statement = 'all';  - что пишем в журнал.

SELECT pg_reload_conf();
```
### 4. Создание logclean.sh
```
vim /postgres/scripts/logrotate/logclean.sh
chmod +x /postgres/scripts/logrotate/logclean.sh
```
# 5. Планировщик
```
*/5 * * * * /postgres/scripts/logrotate/logrotate /postgres/scripts/logrotate/logrotate.conf --state /postgres/logrotate/scripts/logrotate-state
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
### Исходный конфиг:
```
postgres@zabbix:/home/mgb$ cat /postgres/scripts/logrotate.conf
/log/pg_log/*.log
{
        rotate 99
        size 500M
        missingok
        compress
        delaycompress
        notifempty
        maxage 1
}
```




