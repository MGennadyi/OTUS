# PGBENCH+PATRONI тест производительности

###### Изначально postgres имеет параметр max_connections = 100
```
# На лосальном хосте:
sudo - u postgres psql - h localhost
# Через haproxy+keepalived:
psql -p 5432 -d otus -h 192.168.5.180 -U postgres
show max_connections;
# Ответ одинаков: 100
```
###### Смотр параметров на PG1:
```
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



