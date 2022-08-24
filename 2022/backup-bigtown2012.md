# pg_basebackup+PATRONI
###### Состав: lider=pp_pg_1 Sync Standby=pp_pg_2. Бэкап штатными средствами.
```
sudo useradd replicator -p rep-pass_321
mkdir /home/backup
sudo chown -R replicator /home/backup
```
###### 1. Создание полного бекапа:
```
# Не будем дожидаться заполнения журнала, 
pg_basebackup --checkpoint=fast -P -Xstream -z -Ft -h 192.168.5.165 -p 5432 -U replicator -D /home/backup
# Смотрим содержимое каталог архива, после выполнения:
ls -la /home/backup
# Ответ:
drwxr-xr-x 2 replicator root    4096 авг 24 17:21 .
drwxr-xr-x 4 root       root    4096 авг 24 17:13 ..
-rw------- 1 replicator root  181459 авг 24 17:21 backup_manifest
-rw------- 1 replicator root 4535002 авг 24 17:21 base.tar.gz
-rw------- 1 replicator root   17073 авг 24 17:21 pg_wal.tar.gz
```
###### 2. Восстановление из архива. Для этого:
```
# 1. На всех нодах останавливаем patroni:
systemctl stop patroni
systemctl status patroni
# 2. На всех нодах удалаяем (лучше перемещаем) содержимое директорию main:
rm -rf /var/lib/postgresql/14/main/*
# 3. Разархивируем архив base.tar.gz в каталог main:
tar -xzf base.tar.gz -C /var/lib/postgresql/14/main
# 4. Проверим, что владелец в main является postgresql:
ls -la
# 5. Разархивируем архив на лидере pg_wal в main/pg_wal:
tar -xzf pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
```
###### Удаляем patromi
```
# 1. Для васстановления postgresql это уже достаточно, но для восстановления patroni его надо сначала удалить:
patronictl -c /etc/patroni.tml remove pg-ha-cluster
# 7.Ввести название кластера patroni: pg-ha-cluster
# Ввести фразу: Yes I am aware
# Должен заработать после старта:
systemctl start patroni
# т.о восстановили лидера, переходим к slave.
```
##### На реплике
```
systemctl start patroni
```
##### На master 
```
systemctl status patroni
```
# Восстановление PATRONI на PITR Point-in-Time Recovery:
###### На всех нодах останвливаем Patroni и очищаем каталоги с БД:
```
systemctl stop patroni

```






