### UPGRADE PATRONI
```
# Проверка статуса службы:
patronictl -c /etc/patroni.yml topology
```
### Г) Настроить синхронный режим:
```
synchronous_mode: true
synchronous_mode_strict: false
```
###
```
/usr/local/bin/patroni --version
patroni 2.1.4
```
###
```
patronictl -c /etc/patroni.yml pause
patronictl -c /etc/patroni.yml list
Maintenance mode: on
```
























