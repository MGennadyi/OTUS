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
