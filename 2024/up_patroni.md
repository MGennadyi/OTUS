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
























