# Ускорение с помощью конфигурации базы и оптимизации запросов
```
\timing
```
```
create table users
(
  id bigint primary key,
  login varchar(200) not null,
  first_name varchar(200) not null,
  last_name varchar(200) not null,
  create_date timestamp not null default now()
);
```
###### Добавим в нее 10 млн записей
```
insert into users select id, random() * id, md5(sin(id)::text), md5(cos(id)::text) from generate_series(1, 10000000) id;
```
```
analyze users;
```
```
\timing
```
```
select count(distinct id) from users;
# Ответ: Время: 48999,199 мс (00:48,999)
```
###### Хорошо ьы просмотреть hints из ролика про мониторинг
###### Повторим:
```
select count(distinct id) from users;
# Ответ: Время: 5930,707 мс (00:05,931) - разница в 10 раз
```
###### Что с планом запроса explain:
```
explain select count(distinct id) from users;
                                         QUERY PLAN
-----------------------------------------------------------------------------------------------
 Aggregate  (cost=202487.05..202487.06 rows=1 width=8)
   ->  Index Only Scan using users_pkey on users  (cost=0.43..177486.96 rows=10000035 width=8)
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
(5 строк)
Время: 57,999 мс
```
###### Что с планом запроса explain analyze:
```
explain analyze select count(distinct id) from users;
                                                                 QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=202487.05..202487.06 rows=1 width=8) (actual time=3989.718..3989.719 rows=1 loops=1)
   ->  Index Only Scan using users_pkey on users  (cost=0.43..177486.96 rows=10000035 width=8) (actual time=0.035..1045.192 rows=10000000 loops=1)
         Heap Fetches: 3904
 Planning Time: 0.071 ms
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.247 ms, Inlining 0.000 ms, Optimization 0.129 ms, Emission 1.257 ms, Total 1.633 ms
 Execution Time: 3990.045 ms
(9 строк)
Время: 3990,685 мс (00:03,991)
```
######
```
explain (analyze, buffers) select count(distinct id) from users;
                                                           QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=202487.05..202487.06 rows=1 width=8) (actual time=3836.661..3836.662 rows=1 loops=1)
   Buffers: shared hit=27395, temp read=30325 written=30382
   ->  Index Only Scan using users_pkey on users  (cost=0.43..177486.96 rows=10000035 width=8) (actual time=0.037..1063.914 rows=10000000 loops=1)
         Heap Fetches: 3904
         Buffers: shared hit=27395
 Planning Time: 0.075 ms
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.320 ms, Inlining 0.000 ms, Optimization 0.142 ms, Emission 1.386 ms, Total 1.849 ms
 Execution Time: 3837.063 ms
(11 строк)
Время: 3837,578 мс (00:03,838)
```
######  интересный показатель temp_write. Он показывает, что оперативной памяти на запрос не хватило, и в какой-то момент БД создала файл на 30 страниц на диске, а потом оттуда их считывала. То есть можно увеличить work_mem (изм.операции с сортировкой), и тогда этот показатель исчезнет.
```
postgres=# show shared_buffers;
 shared_buffers
----------------
 128MB
otus=# show work_mem;
 work_mem
----------
 4MB
# Смотрим по другому:
postgres=# SELECT current_setting('work_mem');
 current_setting
-----------------
 4MB
# Смотрим по другому:
postgres=# select sourceline, name, setting, applied from pg_file_settings where name = 'work_mem';
 sourceline |   name   | setting | applied
------------+----------+---------+---------
          7 | work_mem | 40MB    | t
 # Смотрим по другому: 
postgres=# select name, setting unit, boot_val, reset_val, source, sourcefile, sourceline, pending_restart, context from pg_settings WHERE name = 'work_mem' \gx
-[ RECORD 1 ]---+---------
name            | work_mem
unit            | 4096
boot_val        | 4096
reset_val       | 4096
source          | default
sourcefile      |
sourceline      |
pending_restart | f
context         | user
```
###### Изменим work_mem, сначала сбросим значение:
```
postgres=# reset work_mem;
otus=# alter system set work_mem = "40MB";
otus=# show work_mem;
 work_mem
----------
 40MB
 # Изменим другим способом:
 SET work_mem TO '40MB'
```
```
otus=# explain (analyze, buffers) select count(distinct id) from users;
                                                                    QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=202487.05..202487.06 rows=1 width=8) (actual time=4175.482..4175.483 rows=1 loops=1)
   Buffers: shared hit=4 read=27400, temp read=14683 written=14688
   I/O Timings: read=950.091
   ->  Index Only Scan using users_pkey on users  (cost=0.43..177486.96 rows=10000035 width=8) (actual time=0.209..2069.674 rows=10000000 loops=1)
         Heap Fetches: 3904
         Buffers: shared read=27395
         I/O Timings: read=939.904
 Planning:
   Buffers: shared hit=44 read=10
   I/O Timings: read=45.341
 Planning Time: 45.765 ms
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.381 ms, Inlining 0.000 ms, Optimization 0.224 ms, Emission 14.108 ms, Total 14.713 ms
 Execution Time: 4888.535 ms
(16 строк)
Время: 5058,112 мс (00:05,058)
```
###### Повторно:
```
otus=# explain (analyze, buffers) select count(distinct id) from users;
                                                                    QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=202487.05..202487.06 rows=1 width=8) (actual time=2983.612..2983.614 rows=1 loops=1)
   Buffers: shared hit=27395, temp read=14683 written=14688
   ->  Index Only Scan using users_pkey on users  (cost=0.43..177486.96 rows=10000035 width=8) (actual time=0.025..1035.222 rows=10000000 loops=1)
         Heap Fetches: 3904
         Buffers: shared hit=27395
 Planning Time: 0.082 ms
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 0.257 ms, Inlining 0.000 ms, Optimization 0.101 ms, Emission 1.218 ms, Total 1.576 ms
 Execution Time: 2983.958 ms
(11 строк)
Время: 2984,443 мс (00:02,984)
```
######
```
select sourceline, name, setting, applied from pg_file_settings where name = 'work_mem';

select name, setting unit, boot_val, reset_val, source, sourcefile, sourceline, pending_restart, context from pg_settings WHERE name = 'work_mem' \gx
```
#### Поиск НЕИСП ИНДЕКСОВ:
###### Самое простое — найти индексы, по которым вообще не было проходов:
```
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```
```
postgres=# SELECT
  relname                                               AS TableName,
  to_char(seq_scan, '999,999,999,999')                  AS TotalSeqScan,
  to_char(idx_scan, '999,999,999,999')                  AS TotalIndexScan,
  to_char(n_live_tup, '999,999,999,999')                AS TableRows,
  pg_size_pretty(pg_relation_size(relname :: regclass)) AS TableSize
FROM pg_stat_all_tables
WHERE schemaname = 'public'
      AND 50 * seq_scan > idx_scan -- more than 2%
      AND n_live_tup > 10000
      AND pg_relation_size(relname :: regclass) > 5000000
ORDER BY relname ASC;
 tablename | totalseqscan | totalindexscan | tablerows | tablesize
-----------+--------------+----------------+-----------+-----------
(0 строк)
--------------------------------
This checks if there are more sequence scans than index scans. If the table is small, it gets ignored, since Postgres seems to prefer sequence scans for them.
Above query does reveal missing indexes.
The next step would be to detect missing combined indexes. I guess this is not easy, but doable. Maybe analyzing the slow queries ... I heard pg_stat_statements could help.
----------------------------------
postgres=# SELECT relname, seq_scan-idx_scan AS too_much_seq, case when seq_scan-idx_scan>0 THEN 'Missing Index?' ELSE 'OK' END,  pg_relation_size(relid::regclass) AS rel_size, seq_scan, idx_scan  FROM pg_stat_all_tables WHERE schemaname='public' AND pg_relation_size(relid::regclass)>80000 ORDER BY too_much_seq DESC;
 relname | too_much_seq | case | rel_size | seq_scan | idx_scan
---------+--------------+------+----------+----------+----------
(0 строк)
```
#####  
```
# Сколько страниц прочитано
alter system set track_io_timing=on;
# Кол-во исп функций:
alter system set track_functions='all';
select pg_reload_conf();
```
```
CREATE DATABASE otus;
\q
pgbench -i otus
# Сброс статистики:
SELECT pg_stat_reset();
SELECT pg_stat_reset_shared('bgwriter');
\c otus
VACUUM pgbench_accounts;
```
```
# Смотрим статистику:
# Есть строчка для каждой таблицы. pgbench_accounts приводим к regclass, чтоб получить внутренний инлетификатор и смотрим строку, относящ. к pgbench_accounts:
otus=# SELECT * FROM pg_stat_all_tables WHERE relid='pgbench_accounts'::regclass \gx
-[ RECORD 1 ]-------+------------------------------
relid               | 16395
schemaname          | public
relname             | pgbench_accounts
seq_scan            | 2
seq_tup_read        | 100000
idx_scan            | 4492
idx_tup_fetch       | 4492
n_tup_ins           | 100000
n_tup_upd           | 2246
n_tup_del           | 0
n_tup_hot_upd       | 1003
n_live_tup          | 100000
n_dead_tup          | 0
n_mod_since_analyze | 2246
n_ins_since_vacuum  | 0
last_vacuum         | 2022-11-18 08:37:29.056348+03
last_autovacuum     | 2022-11-17 17:58:31.44941+03
last_analyze        | 2022-11-17 17:58:31.284511+03
last_autoanalyze    | 2022-11-17 17:58:31.502845+03
vacuum_count        | 2
autovacuum_count    | 1
analyze_count       | 1
autoanalyze_count   | 1

# DESC: инф по кол-ву строчек
```
```
otus=# SELECT * FROM pg_statio_all_tables WHERE relid='pgbench_accounts'::regclass \gx
-[ RECORD 1 ]---+-----------------
relid           | 16395
schemaname      | public
relname         | pgbench_accounts
heap_blks_read  | 4571
heap_blks_hit   | 20723
idx_blks_read   | 551
idx_blks_hit    | 11218
toast_blks_read |
toast_blks_hit  |
tidx_blks_read  |
tidx_blks_hit 

# DESC: инф по кол-ву страниц: heap_blks_read - чтение с диска, heap_blks_hit - чтение из буфера.
```
```
otus=# SELECT * FROM pg_stat_DATABASE WHERE datname='otus' \gx
-[ RECORD 1 ]------------+------------
datid                    | 16388
datname                  | otus
numbackends              | 1
xact_commit              | 4272
xact_rollback            | 0
blks_read                | 5337
blks_hit                 | 126402
tup_returned             | 1013324
tup_fetched              | 21851
tup_inserted             | 102357
tup_updated              | 6754
tup_deleted              | 0
conflicts                | 0
temp_files               | 4
temp_bytes               | 2023424
deadlocks                | 0
checksum_failures        |
checksum_last_failure    |
blk_read_time            | 18.839
blk_write_time           | 13.731
session_time             | 6203222.083
active_time              | 10530.677
idle_in_transaction_time | 414.18
sessions                 | 4
sessions_abandoned       | 0
sessions_fatal           | 0
sessions_killed          | 0
stats_reset  
```
```
otus=# SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 198
checkpoints_req       | 1
checkpoint_write_time | 425549
checkpoint_sync_time  | 97
buffers_checkpoint    | 3017
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 2776
buffers_backend_fsync | 0
buffers_alloc         | 996
stats_reset           | 2022-11-17 18:01:59.523438+03
```


























