# PGBENCH+PATRONI тест производительности
```
# Уст. времени выполнения команд:
apt install time -y
# Пример:
time apt update
```

###### Изначально postgres имеет параметр max_connections = 100
```
# На лосальном хосте:
sudo -u postgres psql -h localhost
# Через haproxy+keepalived:
psql -p 5432 -d otus -h 192.168.5.180 -U postgres
show max_connections;
# Ответ одинаков: 100
```
###### Смотр и правка параметров конфига на PG1:
```
# 
sudo patronictl -c /etc/patroni.yml edit-config
# Ответ:
loop_wait: 10
maximum_lag_on_failover: 1048576
postgresql:
  parameters: null
  use_pg_rewind: true
retry_timeout: 10
ttl: 30
```
###### Аристов: pinding restart
```
# Не поможет, если:
sudo -u postgres psql -h localhost
alter system set max_connections=25;
````
###### Может помочь:
```
sudo patronictl -c /etc/patroni.yml edit-config
# Убираем null и Добавляем ниже:
  parameters:
    max_connections: 25
```
```
sudo -u postgres psql -h localhost
show max_connections;
```
```
# Рестатуем PATRONI:
sudo patronictl -c /etc/patroni.yml list
sudo patronictl -c /etc/patroni.yml restart patroni
# Ответ:
When should the restart take place (e.g. 2022-09-15T10:03)  [now]: enter - прямо сейчас
Are you sure you want to restart members pg1, pg2, pg3? [y/N]: y - конечно!
Restart if the PostgreSQL version is less than provided (e.g. 9.5.2)  []: enter - согоашаемся
# Ответ:
Success: restart on member pg3
Success: restart on member pg2
Success: restart on member pg1
```
###### Для анализа производительности под нагрузкой подключим pg_stat_statements :
```
# так:
sudo patronictl -c /etc/patroni.yml edit-config
shared_preload_libraries: pg_stat_statements
sudo vim /etc/patroni.yml
shared_preload_libraries: 'pg_stat_statements'
sudo patronictl -c /etc/patroni.yml restart patroni
sudo -u postgres psql -h localhost
 \timing
CREATE EXTENSION pg_stat_statements;
````
```
# Плохо читается:
SELECT * FROM pg_stat_statements ORDER BY total_exec_time DESC;
# Смотрим поля в таблице и видим total_time заменено на total_exec_time:
\d pg_stat_statements

# запрос, показывающий долгих по времени выполнения:
SELECT substring (query, 1, 50) AS short_query,
              round (total_exec_time::numeric, 2) AS total_exec_time,
              calls,
              round (total_exec_time::numeric, 2) AS mean,
              round ((100 * total_exec_time / sum(total_exec_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM  pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```
###### 4. Предв.подготовка: создание и заполнение таблиц для теста: pgbench с ключом -i (иниц-я), -s (маштаб-е) -F (фактор заполнения)-U (user); БД testpgbench:
```
# На лидере patroni:
sudo -u postgres psql -h localhost
create database testpgbench;
pgbench -h 192.168.5.165 -p 5432 -U postgres -i -s 100 -F 80 testpgbench
pgbench -h 10.128.0.64 -p 5432 -U postgres -i -s 100 -F 80 testpgbench
pgbench -i -s 100 -F 80 -U postgres testpgbench
```
###### Запуск теста: -c (подключенных коиентов)

```
###### Тест на разницу лидер/keepalived
time pgbench -c 25 -j 2 -P 10 -T 300 -U postgres otus
# Ответ:
done in 1237.82 s (drop tables 0.00 s, create tables 0.13 s, client-side generate 263.41 s, vacuum 908.19 s, primary keys 66.08 s).
pgbench -c 25 -j 2 -P 10 -T 300 -p 5432 -h 192.168.5.180 -U postgres testpgbench
###### Тест на разницу при 125 клиантах с ограничением 25  лидер/keepalived
pgbench -c 125 -j 2 -P 10 -T 300 -U postgres otus
pgbench -c 125 -j 2 -P 10 -T 300 -p 5432 -h 192.168.5.180 -U postgres otus
# Запуск теста
pgbench -c 1 -j  -P 10 -T 300 -p 5432 -h 192.168.5.180 -U postgres otus
```

###### Тест host=pgdump Режим synchronous_commit = on
```
# 
postgres=# SHOW synchronous_commit;
 synchronous_commit
--------------------
 on
(1 строка)
# Инициализация:
time pgbench -i -s 100 -F 80 -U postgres otus
done in 185.96 s (drop tables 0.42 s, create tables 0.09 s, client-side generate 57.41 s, vacuum 75.85 s, primary keys 52.18 s). real    3m6,037s

# Запуск теста на 100 соединений:
time pgbench -c 100 -j 1 -P 10 -T 100 -U postgres otus
# Ответ:
latency average = 741.818 ms
latency stddev = 514.101 ms
initial connection time = 432.039 ms
# Отевет: tps = 132.733466 (without initial connection time) real    1m42,602s
```
###### Тест host=pgdump Режим synchronous_commit = off
```
ALTER SYSTEM SET synchronous_commit = off;
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
postgres=# SHOW synchronous_commit;
 synchronous_commit
--------------------
 off
# Результат: tps = 139.934274 (without initial connection time) real    1m40,879s
# Изменим режим:
postgres=# SHOW synchronous_commit;
 synchronous_commit
--------------------
 on
time pgbench -c 100 -j 1 -P 10 -T 100 -U postgres otus
# Ответ: tps = 121.485529 (without initial connection time)
time pgbench -c 10 -j 1 -P 10 -T 100 -U postgres otus
# Ответ: tps = 94.506382 (without initial connection time)
time pgbench -c 1 -j 1 -P 10 -T 100 -U postgres otus
# Ответ: tps = 49.019043 (without initial connection time)
time pgbench -c 1 -j 4 -P 10 -T 100 -U postgres otus
tps = 39.520682 (without initial connection time)





```












