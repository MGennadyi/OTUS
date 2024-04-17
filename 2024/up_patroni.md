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
### Проверяю:
```
sudo systemctl status patroni
Active: active (runing) - во дела, работает!
```
### Останавливаю patroni:
```
sudo systemctl stop patroni
sudo systemctl start patroni
sudo systemctl status patroni
Active: filed - то, что нужно!
```
### Установить новую версию:
```
sudo apt install patroni -что верно-??? -на srv1
python3 -m pip install --upgrade --user patroni[etcd] -на srv2
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
























