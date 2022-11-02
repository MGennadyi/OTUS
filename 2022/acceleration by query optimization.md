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


































