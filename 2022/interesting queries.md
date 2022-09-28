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



```
