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
###### 2. Восстановление из архива. Для этого:
```
# 1. На всех нодах останавливаем patroni:
systemctl stop patroni
systemctl status patroni
# 2. На всех нодах удалаяем (лучше перемещаем) директорию main:
rm -rf /var/lib/postgresql/14/main
# 3. азархивируем base.tar.gz в каталог main:
tar -xzf base.tar.gz -C /var/lib/postgresql/14/main
# 4. Проверим, что владелец в main является postgresql:
ls -la
# 5. На лидере распаковка pg_wal:
tar -xzf pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
# 6. Удаление кластера patroni:
patronictl -c /etc/patroni.tml remove pg-ha-cluster
# 7.Ввести: Yes I am aware
systemctl start patroni
```
```
# На реплике
systemctl start patroni
```









