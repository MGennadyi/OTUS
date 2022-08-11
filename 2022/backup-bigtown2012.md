# pg_basebackup+PATRONI
###### Состав: lider=pp_pg_1 Sync Standby=pp_pg_2. Бэкап штатными средствами.
```


```
###### 1. Создание полного бекапа:
```
# Не будем дожидаться заполнения журнала, 
pg_basebackup --checkpoint=fast -P -Xstream -z -Ft -h 10.128.0.51 -p15432 -U repl -D /mnt/fastceph/full
# Смотрим содержимое каталог архива, после выполнения:
ls -la /mnt/fastceph/full
# Видим 2 файла:
base.tar.gz
pg_wal.tar.gz
```
###### Восстановление из архива:
```
# На всех нодах останавливаем patroni:
systemctl stop patroni
systemctl status patroni
# На всех нодах удалаяем (лучше перемещаем) main:
rm -rf /var/lib/postgresql/14/main
tar -xzf base.tar.gz -C /var/lib/postgresql/14/main
# На лидере распаковка pg_wal:
tar -xzf pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
# Удаление кластера patroni:
patronictl -c /etc/patroni.tml remove pg-ha-cluster
# Ввести: Yes I am aware
systemctl start patroni
```
```
# На реплике
systemctl start patroni
```









