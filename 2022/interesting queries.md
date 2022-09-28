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
SELECT substring(query, 1, 50) AS short_query, round(total_time::numeric, 2) AS total_time, calls,
rows, round(total_time::numeric / calls, 2) AS avg_time, round((100 * total_time /
sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM pg_stat_statements ORDER BY total_time DESC LIMIT 20;

# ТОП по времени выполнения: 
SELECT substring(query, 1, 100) AS short_query, round(total_time::numeric, 2) AS total_time, calls,
rows, round(total_time::numeric / calls, 2) AS avg_time, round((100 * total_time /
sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM pg_stat_statements ORDER BY avg_time DESC LIMIT 20;


```


