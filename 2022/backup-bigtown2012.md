# pg_basebackup+PATRONI
###### Состав: lider=pp_pg_1 Sync Standby=pp_pg_2 
```


```
###### Создание полного бекапа:
```
# Не будем дожидаться заполнения журнала, 
pg_basebackup --checkpoint=fast -P -Xstream -z -Ft -h 10.128.0.51 -p15432 -U repl -D /mnt/fastceph/full
```
###### Восстановление из архива:
```
# Останавливаем patroni:
systemctl stop patroni
systemctl status patroni
# На всех нодах удалаяем (лучше перемещаем) main:
rm -rf /var/lib/postgresql/14/main

```









