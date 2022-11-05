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
######  интересный показатель temp_write. Он показывает, что оперативной памяти на запрос не хватило, и в какой-то момент БД создала файл на 30 страниц на диске, а потом оттуда их считывала. То есть можно увеличить work_mem, и тогда этот показатель исчезнет.
```
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































