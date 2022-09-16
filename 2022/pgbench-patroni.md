# PGBENCH+PATRONI тест производительности

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
alter system set max_connections=20;
````
###### Может помочь:
```
# Убираем null
  parameters:
# Добавляем ниже 
max_connections: 15
```
```
sudo -u postgres psql -h localhost
show max_connections;
# Ответ: 100 - не помогло.
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
```
# так:
sudo patronictl -c /etc/patroni.yml edit-config
shared_preload_libraries: pg_stat_statements
sudo vim /etc/patroni.yml
shared_preload_libraries: 'pg_stat_statements'
sudo patronictl -c /etc/patroni.yml restart patroni
sudo -u postgres psql -h localhost
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





















