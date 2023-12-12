###### Просмотр полей в представлении; total_exec_time:
```
\d pg_stat_statements
\d pg_stat_activity
```
## pg_stat_activity
###### Просмотр активных сессий:
```
select pid as process_id, usename as username, datname as database_name, client_addr as client_address, application_name, backend_start, state, state_change from pg_stat_activity;
```
###### Инфа по выполняющимся в данное время запросам :
```
# активные (state='active') запросы длительностью более 5 секунд:
SELECT now() - query_start as "runtime", usename, datname, wait_event_type, state, query
FROM pg_stat_activity WHERE now() - query_start > '5 seconds'::interval and state='active'
ORDER BY runtime DESC;
```

```
# State = ‘idle’ Но хуже всего – idle in transaction!
# для active
SELECT pg_cancel_backend(procpid); 
```
```
# для idle State = ‘idle’
SELECT pg_terminate_backend(procpid); для idle
```
```
# Повисшие транзакции - зло:
SELECT pid, xact_start, now() - xact_start AS duration
FROM pg_stat_activity
WHERE state LIKE '%transaction%'
ORDER BY duration DESC;
```
## pg_stat_statements
###### Инфа по раннее выполненным, т.е. завершенным запросам: 
```
# Предварительно активировать:
show config_file;
 -----------------------------------------
 /etc/postgresql/14/main/postgresql.conf
# Не верно:
sudo vim /var/lib/postgresql/14/main/postgresql.conf
# Верно:
sudo vim /etc/postgresql/14/main/postgresql.conf

# Для patroni: add в секции 'postgresql':
vim /etc/patroni.yml
shared_preload_libraries = 'pg_stat_statements'
systemctl stop patroni
systemctl start patroni
postgres=# show shared_preload_libraries;
 shared_preload_libraries
--------------------------
 pg_stat_statements
(1 строка)

create extension pg_stat_statements;
# Необходимо перечитать конфигурацию
psql -c "SELECT pg_reload_conf();"
=======================================
vim /etc/postgresql/13/main/postgresql.conf
#-------------------
#CLIENT CONNECTION DEFAULTS
#--------------------
shared_preload_libraries = 'pg_stat_statements'


```
## pg_repack
```
postgres=# ALTER SYSTEM SET shared_preload_libraries = 'pg_repack'; -отключит другие расширения, не применять
ALTER SYSTEM
SELECT pg_reload_conf(); - не поможет. Только стоп/старт
# Важно: 'pg_stat_statements' -удалиться. По этому правим 
vim /etc/postgresql/13/main/postgresql.conf
\c demo
Вы подключены к базе данных "demo" как пользователь "postgres".
demo=# create extension pg_repack; - уст на конкретную БД, к примеру demo.
CREATE EXTENSION
drop extension pg_repack;
```
```
# ТОП по загрузке CPU:
SELECT substring(query, 1, 50) AS short_query, round(total_exec_time::numeric, 2) AS total_time, calls,
rows, round(total_exec_time::numeric / calls, 2) AS avg_time, round((100 * total_exec_time /
sum(total_exec_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 20;

                                  short_query                                  | total_time | calls |   rows   | avg_time  | percentage_cpu
-------------------------------------------------------------------------------+------------+-------+----------+-----------+----------------
 vacuum analyze pgbench_accounts                                               |  905079.02 |     1 |        0 | 905079.02 |          45.12
 SELECT count(*)                                                              +|  396347.46 | 38454 |    38454 |     10.31 |          19.76
                                 FROM pg_catalog.pg_stat_all_ta                |            |       |          |           |
 copy pgbench_accounts from stdin                                              |  261989.00 |     1 | 10000000 | 261989.00 |          13.06
 SELECT row_to_json (T)                                                       +|   90442.45 |  7689 |     7689 |     11.76 |           4.51
     FROM (                                                                   +|            |       |          |           |
           SELECT                                                              |            |       |          |           |
 SELECT json_object_agg(coalesce (datname,$1), row_                            |   82928.18 |  7692 |     7692 |     10.78 |           4.13
 SELECT row_to_json (T)                                                       +|   82373.48 |  7690 |     7690 |     10.71 |           4.11
     FROM  (                                                                  +|            |       |          |           |
       SELECT                                                                 +|            |       |          |           |
                                                                               |            |       |          |           |
 SELECT row_to_json(T)                                                        +|   82354.48 |  7689 |     7689 |     10.71 |           4.11
                                                         FROM (               +|            |       |          |           |
                                                                         SELEC |            |       |          |           |
 alter table pgbench_accounts add primary key (aid)                            |   65289.45 |     1 |        0 |  65289.45 |           3.25
 SELECT row_to_json(T)                                                        +|   10041.71 |  7689 |     7689 |      1.31 |           0.50
                                                         FROM (               +|            |       |          |           |
                                                                 WITH v        |            |       |          |           |
 SELECT pg_database_size(datname::text)                                       +|    8005.40 |  7693 |     7675 |      1.04 |           0.40
                 FROM pg_c                                                     |            |       |          |           |
 SELECT pg_catalog.pg_postmaster_start_time(), CASE                            |    5447.05 | 48512 |    48512 |      0.11 |           0.27
 WITH T AS                                                                    +|    2671.44 |  7692 |     7692 |      0.35 |           0.13
         (SELECT db.datname dbname,                                           +|            |       |          |           |
                         lower(rep                                             |            |       |          |           |
 vacuum analyze pgbench_branches                                               |    2198.60 |     1 |        0 |   2198.60 |           0.11
 WITH T AS (                                                                  +|    2078.59 |  7690 |     7690 |      0.27 |           0.10
                 SELECT                                                       +|            |       |          |           |
                         db.datname,                                          +|            |       |          |           |
                         coalesce(T.                                           |            |       |          |           |
 SELECT CASE WHEN pg_catalog.pg_is_in_recovery() TH                            |    1156.64 | 46154 |    46154 |      0.03 |           0.06
 SELECT row_to_json(T)                                                        +|     956.02 |  7690 |     7690 |      0.12 |           0.05
         FROM (                                                               +|            |       |          |           |
                 SELECT                                                       +|            |       |          |           |
                         sum(CASE                                              |            |       |          |           |
 SELECT json_object_agg(application_name, row_to_js                            |     837.65 |  7689 |     7689 |      0.11 |           0.04
 SELECT COUNT(DISTINCT client_addr) + COALESCE(SUM(                            |     764.01 |  7692 |     7692 |      0.10 |           0.04
 vacuum analyze pgbench_tellers                                                |     677.55 |     1 |        0 |    677.55 |           0.03
 CHECKPOINT                                                                    |     617.57 |     5 |        0 |    123.51 |           0.03
(20 строк)


# ТОП по времени выполнения: 
SELECT substring(query, 1, 100) AS short_query, round(total_exec_time::numeric, 2) AS total_time, calls,
rows, round(total_exec_time::numeric / calls, 2) AS avg_time, round((100 * total_exec_time /
sum(total_exec_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM pg_stat_statements ORDER BY avg_time DESC LIMIT 20;


                                                 short_query                                                 | total_time | calls |   rows   | avg_time  | percentage_cpu
-------------------------------------------------------------------------------------------------------------+------------+-------+----------+-----------+----------------
 vacuum analyze pgbench_accounts                                                                             |  905079.02 |     1 |        0 | 905079.02 |          45.09
 copy pgbench_accounts from stdin                                                                            |  261989.00 |     1 | 10000000 | 261989.00 |          13.05
 alter table pgbench_accounts add primary key (aid)                                                          |   65289.45 |     1 |        0 |  65289.45 |           3.25
 vacuum analyze pgbench_branches                                                                             |    2198.60 |     1 |        0 |   2198.60 |           0.11
 vacuum analyze pgbench_tellers                                                                              |     677.55 |     1 |        0 |    677.55 |           0.03
 CREATE EXTENSION pg_stat_statements                                                                         |     343.62 |     1 |        0 |    343.62 |           0.02
 alter table pgbench_branches add primary key (bid)                                                          |     308.86 |     1 |        0 |    308.86 |           0.02
 SELECT pg_catalog.pg_create_physical_replication_slot($1, $2) WHERE NOT EXISTS (SELECT $3 FROM pg_ca        |     429.15 |     2 |        2 |    214.57 |           0.02
 alter table pgbench_tellers add primary key (tid)                                                           |     188.34 |     1 |        0 |    188.34 |           0.01
 vacuum analyze pgbench_history                                                                              |     172.47 |     1 |        0 |    172.47 |           0.01
 CHECKPOINT                                                                                                  |     617.57 |     5 |        0 |    123.51 |           0.03
 SELECT pg_catalog.pg_drop_replication_slot($1) WHERE EXISTS (SELECT $2 FROM pg_catalog.pg_replicatio        |     135.25 |     4 |        4 |     33.81 |           0.01
 SELECT row_to_json (T)                                                                                     +|   90586.81 |  7703 |     7703 |     11.76 |           4.51
     FROM (                                                                                                 +|            |       |          |           |
           SELECT                                                                                           +|            |       |          |           |
               checkpoints_timed                                                                            +|            |       |          |           |
             , che                                                                                           |            |       |          |           |
 SELECT json_object_agg(coalesce (datname,$1), row_to_json(T))                                              +|   83073.50 |  7706 |     7706 |     10.78 |           4.14
     FROM  (                                                                                                +|            |       |          |           |
       SELECT                                                                                               +|            |       |          |           |
         datna                                                                                               |            |       |          |           |
 SELECT row_to_json(T)                                                                                      +|   82488.40 |  7702 |     7702 |     10.71 |           4.11
                                                         FROM (                                             +|            |       |          |           |
                                                                         SELECT archived_count, failed_count+|            |       |          |           |
                                                                           FROM                              |            |       |          |           |
 SELECT row_to_json (T)                                                                                     +|   82518.53 |  7704 |     7704 |     10.71 |           4.11
     FROM  (                                                                                                +|            |       |          |           |
       SELECT                                                                                               +|            |       |          |           |
         sum(numbackends) as numbackends                                                                    +|            |       |          |           |
       , sum(                                                                                                |            |       |          |           |
 SELECT count(*)                                                                                            +|  396911.90 | 38519 |    38519 |     10.30 |          19.77
                                 FROM pg_catalog.pg_stat_all_tables                                         +|            |       |          |           |
                            WHERE (n_dead_tup/(n_live_tup+n_dead                                             |            |       |          |           |
 create table pgbench_history(tid int,bid int,aid    int,delta int,mtime timestamp,filler char(22))          |       3.39 |     1 |        0 |      3.39 |           0.00
 SELECT name, setting, unit, vartype, context, sourcefile FROM pg_catalog.pg_settings  WHERE pg_catal        |      18.67 |     9 |      117 |      2.07 |           0.00
 SELECT row_to_json(T)                                                                                      +|   10055.22 |  7702 |     7702 |      1.31 |           0.50
                                                         FROM (                                             +|            |       |          |           |
                                                                 WITH values AS (                           +|            |       |          |           |
                                                                         SELECT                             +|            |       |          |           |
                                                                                 $1/(ceil(pg_s               |            |       |          |           |
(20 строк)
```
#### pg_stat_user_tables
```
# Выборка таблиц в которых больше всего операций последовательного чтения:
SELECT schemaname, relname, seq_scan, seq_tup_read, seq_tup_read / seq_scan AS avg, idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC
LIMIT 25;
 schemaname | relname | seq_scan | seq_tup_read | avg | idx_scan
------------+---------+----------+--------------+-----+----------
(0 строк)
```
#### pg_statio_user_tables
```
# кэширование таблиц, вставить поля heap_blks и idx_blks
```
### Размер одной базы:
```
SELECT pg_size_pretty(pg_database_size('otus'));
```
### Размер всех баз:
```
select datname, pg_size_pretty(pg_database_size(datname)) from pg_database where not datname in ('postgres','template0','template1') order by pg_size_pretty desc;
```
### Обнаружение не использ-х индексов
```
# wiki.postgresql.org/wiki/index_maintenance
# https://wiki.postgresql.org/wiki/Index_Maintenance  - !!!

select 
schemaname || '.' || relname AS table, 
indexrelname AS index,
pg_size_pretty (pg_relation_size (i.indexrelid)) AS index_size, 
idx_scan as index_scans 
from pg_stat_user_indexes ui
JOIN pg_index i ON ui.indexrelid = i.indexrelid
WHERE NOT indisunique AND idx_scan < 50 AND pg_relation_size (relid) > 5 * 8192
ORDER BY pg_relation_size (i.indexrelid) / nullif (idx_scan, 0) DESC;
```
```
SELECT column_name, column_default, data_type 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE table_name = 'pg_stat_user_indexes';
```
### Поиск неиспользуемых индексов
```
 Если idx.scan=0, значит индекс не используется 
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
SELECT * FROM pg_stat_all_indexes WHERE relname='t';


SELECT *
FROM
  pg_stat_statements
ORDER BY
  total_exec_time DESC;
```
### Отсутствующие индексы
```
SELECT
  relname,
  seq_scan - idx_scan AS too_much_seq,
  CASE
    WHEN
      seq_scan - coalesce(idx_scan, 0) > 0
    THEN
      'Missing Index?'
    ELSE
      'OK'
  END,
  pg_relation_size(relname::regclass) AS rel_size, seq_scan, idx_scan
FROM
  pg_stat_all_tables
WHERE
  schemaname = 'public'
  AND pg_relation_size(relname::regclass) > 80000
ORDER BY
  too_much_seq DESC;
```
### Неиспользуемые индексы
```
SELECT
  indexrelid::regclass as index,
  relid::regclass as table,
  'DROP INDEX ' || indexrelid::regclass || ';' as drop_statement
FROM
  pg_stat_user_indexes
  JOIN
    pg_index USING (indexrelid)
WHERE
  idx_scan = 0
  AND indisunique is false;
```

```
postgres=# SELECT column_name, column_default, data_type
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'pg_stat_user_indexes';
  column_name  | column_default | data_type
---------------+----------------+-----------
 relid         |                | oid
 indexrelid    |                | oid
 idx_scan      |                | bigint
 idx_tup_read  |                | bigint
 idx_tup_fetch |                | bigint
 schemaname    |                | name
 relname       |                | name
 indexrelname  |                | name
(8 строк)
```
```
vim /etc/postgresql/14/main/postgresql.conf
```
```
\d pg_stat_history
```
##### Видит ли ZABBIX больше 1 инстанса на ноде
```
time sudo pg_createcluster 14 main2
time sudo pg_ctlcluster 14 main2 stop
time sudo pg_dropcluster 14 main2
time sudo pg_ctlcluster 14 main3 stop
time sudo pg_dropcluster 14 main3
```
##### Перекинуть таблицу из БД otus в otus2:
```
pg_dump -d otus --table=test | psql -d otus2
```
##### Стоимость чтения с диска HDD и SSD;
```
select type, database, user_name, address, auth_method FROM pg_hba_file_rules;
select random_page_cost FROM postgresql_file_rules;
psql -c 'show seq_page_cost'
# Ответ: 1
psql -c 'alter system set seq_page_cost = 0.1'
systemctl restart postgresql
# Ответ: 0.1
psql -c 'show random_page_cost'
# Ответ: 4
```
##### Вкл.статист ввода-вывода
```
psql -c 'ALTER SYSTEM SET track_io_timing=on;'
psql -c 'ALTER SYSTEM SET track_functions="all";'
SELECT pg_reload_conf();
# Сброс статистики:
SELECT pg_stat_reset();
```
###### Загруженность оборудования
```
# Если показатель приближается к 100%, то нужно подумать об увеличении памяти.
iostat –dx
```
###### Активеые запросы больше 5 сек:
```
SELECT now() - query_start as "runtime", usename, datname, wait_event, state, query FROM pg_stat_activity WHERE now() - query_start > '5 seconds'::interval and state='active' ORDER BY runtime DESC;
 runtime | usename | datname | wait_event | state | query
---------+---------+---------+------------+-------+-------
(0 строк)
```
###### Зависшие транзакции:
```
sudo -u postgres psql -h localhost
SELECT pid, xact_start, now() - xact_start AS duration FROM pg_stat_activity WHERE state LIKE '%transaction%' ORDER BY 3 DESC;
select pg_cancel_backend(pid) from pg_stat_activity where state = 'idle in transactions' and datname = 'otus';
select pg_cancel_backend(pid) from pg_stat_activity where state = 'idle in transactions';
select * from pg_stat_activity\gx
select datname, usename, wait_event from pg_stat_activity where state = 'idle';
select count(*) from pg_stat_activity where state = 'idle';
```
###### random_page_cost
```
show random_page_cost;
 random_page_cost
------------------
 4
alter system set random_page_cost = 10;
sudo patronictl -c /etc/patroni.yml restart patroni
show random_page_cost;
 random_page_cost
------------------
 10
 show max_connections;
 alter system set max_connections = 150;
 systemctl restart postgresql
pgbench -i -s 100 -F 80 -U postgres otus
```
##### Мониторинг процессов автовакуума
```
# Для 1С:
# Увелич.autovacuum_max_workers в 2 раза, > autovacuum_vacuum_cost_limit увелич. в два раза;
autovacuum_analyze_scale_factor = 0.5
# Мониторинг процессов автовакуума
select count(1) from pg_stat_progress_vacuum;
```
##### REINDEX INDEX CONCURRENTLY
```
CREATE INDEX CONCURRENTLY new_index ON …;
DROP INDEX CONCURRENTLY old_index;
ALTER INDEX new_index RENAME TO old_index;
CREATE EXTENSION pg_repack;
```
###### Включение логирование
```
otus=# ALTER SYSTEM SET log_min_duration_statement = 0;
otus=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 строка)
# Смотрим, что параметр применился:
otus=# show log_min_duration_statement;
 log_min_duration_statement
----------------------------
 0
(1 строка)
```
###### Уст pgbadger
```
sudo DEBIAM_FRONTEND=nointeractive apt install pgbadger -y
tail /var/log/postgresql/postgresql-14-main.log
pgbadger -f /var/log/postgresql/postgresql-14-main.log


ALTER SYSTEM SET log_line_prefix='(pid=%p) ';
SELECT pg_reload_conf();
FATAL: unable to detect log file format from /var/log/postgresql/postgresql-14-main.log, please use -f option.
    - Error at line 17757
```
###### PERF TOP
```
# Не проходит
sudo apt install linux-tools-common
# Не верный путь:
root@wal-g2:/home/mgb# uname -r
5.10.0-12-amd64
sudo apt-get install linux-tools-5.10.0-12

# Правильный путь!:
sudo apt install linux-perf
perf -v
perf version 5.10.149
# Запуск
perf top
```
###### AUDIT
```
sudo apt-get install auditd
auditctl -v
auditctl version 3.0

root@wal-g2:/home/mgb# auditctl -e 0
enabled 0
failure 1
pid 595353
rate_limit 0
backlog_limit 8192
lost 0
backlog 0
backlog_wait_time 60000
backlog_wait_time_actual 0
root@wal-g2:/home/mgb#
```
### VACUUM
```
\c бД
VACUUM имя_табл;
SELECT * FROM pg_stat_progress_vacuum \gx
```


