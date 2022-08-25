# pg_basebackup+PATRONI
###### Состав: lider=pp_pg_1 Sync Standby=pp_pg_2. Бэкап штатными средствами.
```
sudo useradd replicator -p rep-pass_321
mkdir /home/backup
sudo chown -R replicator /home/backup
```
###### 1. Создание полного бекапа:
```
# Не будем дожидаться заполнения журнала:
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
###### 2. Восстановление:
```
# 2.1. На всех нодах останновить patroni:
systemctl status patroni
systemctl stop patroni
systemctl status patroni
# 2.2 На всех нодах удалаяем (лучше перемещаем) содержимое директорию main:
rm -rf /var/lib/postgresql/14/main/*
# 3.3. Разархивируем архив base.tar.gz в каталог main:
tar -xzf base.tar.gz -C /var/lib/postgresql/14/main
#  Проверим, что владелец в main является postgresql:
ls -la
# 3.4 Разархивируем архив на лидере pg_wal в main/pg_wal:
tar -xzf pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
```
###### Удаляем patromi
```
# 1. Для васстановления postgresql это уже достаточно, но для восстановления patroni его надо сначала удалить:
patronictl -c /etc/patroni.yml remove patroni
# Ответ:
Please confirm the cluster name to remove:
patroni
# Ответ:
You are about to remove all information in DCS for patroni, please type: "Yes I am aware":
Yes I am aware
# Ответ: ничего
# Стартуем:
systemctl start patroni
sudo patronictl -c /etc/patroni.yml list
# Ответ:
+--------+---------------+--------+---------+----+-----------+
| Member | Host          | Role   | State   | TL | Lag in MB |
+ Cluster: patroni (7135394571389649652) ---+----+-----------+
| pg1    | 192.168.5.165 | Leader | running |  2 |           |
+--------+---------------+--------+---------+----+-----------+
# т.о восстановили лидера. 
# Переходим к slave pg2:
systemctl start patroni
# На pg1
sudo patronictl -c /etc/patroni.yml list
+--------+---------------+---------+---------+----+-----------+
| Member | Host          | Role    | State   | TL | Lag in MB |
+ Cluster: patroni (7135394571389649652) ----+----+-----------+
| pg1    | 192.168.5.165 | Leader  | running |  2 |           |
| pg2    | 192.168.5.166 | Replica | stopped |    |   unknown |
+--------+---------------+---------+---------+----+-----------+
# На pg3:
systemctl start patroni
# На pg1:
sudo patronictl -c /etc/patroni.yml list
+--------+---------------+---------+---------+----+-----------+
| Member | Host          | Role    | State   | TL | Lag in MB |
+ Cluster: patroni (7135394571389649652) ----+----+-----------+
| pg1    | 192.168.5.165 | Leader  | running |  2 |           |
| pg2    | 192.168.5.166 | Replica | running |  2 |         0 |
| pg3    | 192.168.5.167 | Replica | stopped |    |   unknown |
+--------+---------------+---------+---------+----+-----------+
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






