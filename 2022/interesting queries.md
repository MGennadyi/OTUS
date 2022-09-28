## pg_stat_activity
###### Инфа по выполняющимся запросам в данное время
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
###### Инфа по раннее выполненным запросам
```
# Предварительно активировать:
vim /var/lib/postgresql/14/main/postgresql.conf
# Для patroni: add в секции 'postgresql':
vim /etc/patroni.yml
shared_preload_libraries = 'pg_stat_statements'
systemctl stop patroni
systemctl start patroni
create extension pg_stat_statements;
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
### Размер базы данных:
```
SELECT pg_size_pretty( pg_database_size( 'otus' ) );
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
ORDER BY pg_relation_size (i.indexrelid) / nullif (idx_scan, 0) DESC NULLS FIRST pg_relation_size (i.indexrelid) DESC;
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





