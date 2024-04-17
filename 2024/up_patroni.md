### UPGRADE PATRONI
```
# Проверка статуса службы:
patronictl -c /etc/patroni.yml topology
```
### Г) Настроить синхронный режим:
```
patronictl -c /etc/patroni.yml edit-config -s synchronous_mode=true synchronous_mode_strict=false

```
### Проверка версии до обновления:
```
/usr/local/bin/patroni --version # или так:
patroni --version
patroni 2.1.4
```
### ssh srv1
```
patronictl -c /etc/patroni.yml pause
patronictl -c /etc/patroni.yml list
Maintenance mode: on
```
### Удалить старую версию:
```
sudo python3 -m pip uninstall patroni[etcd]
Successful unistalled patroni - 2.1.4
Warning: result is broken permissions
```
### Установить новую версию:
```
# 1
sudo apt install patroni
# 2
python3 -m pip install --upgrade --user patroni[etcd]
sudo systemctl status patroni
sudo systemctl cat patron
[Unit]

[Service

[Install]
```
### Проверка версии после обновления:
```
/usr/bin/patroni --version
3.2.2
```
```
sudo systemctl restart patroni
```
```
patronictl -c /etc/patroni.yml version
3.2.2
```
























