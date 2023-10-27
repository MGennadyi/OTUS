# Копирование БД
### Выгоняем всех пользователей из БД otus
```
psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='otus';"
```

```
cd /backup
 mkdir 2023_08_30
 WORKDIR=2023_08_30
 DUMPALLDIR=/backup/$WORKDIR
 time pg_dumpall --quote-all-identifiers --verbose --file $DUMPALLDIR/dumpall.dmp >> $DUMPALLDIR/dumpall.dmp.log 2>&1
 ```
### Проверка
```
grep -E "ERROR|ОШИБКА|DETAIL|CONTEXT|FATAL|ВАЖНО|ПАНИКА|PANIC|ПОДРОБНОСТИ|ПРЕДУПРЕЖДЕНИЕ" $DUMPALLDIR/dumpall.dmp.log
tail -n 10 $DUMPALLDIR/dumpall.dmp.log
tail -n 10 $DUMPALLDIR/dumpall.dmp
```
### Делаем дамп БД-источник
```
/opt/pgpro/ent-11/bin/pg_dump -C -h localhost -U postgres 'bd_ist' > /data/copy_db/bd_ist.bac

Заменяем инструкции в дампе, меняя имя БД-источника на целевое имя БД-цель
sed -i "s/db_ist/bd_cel/g" bd_ist.bac

sed -i "s/prod_sut/test_test/g" prod_sut.dmp
```
```
Создаю дамп целевой БД на целевом сервере

time pg_dump -U postgres test_test > /backup/dump/test_test.dmp

time pg_dump -U postgres -d test_test --create | gzip > /backup/dump/test_test.dmp.gz
Создаем дамп.

 

cat /backup/op286466.sh
# pg_dump
DUMPDIR=/backup/"$(date '+%Y_%m_%d')"/pg_dump
mkdir -p $DUMPDIR
# DBNAME=ИМЯ_БД (заполняется на этапе планирования)
DBNAME=name_db
time pg_dump -U postgres -v --file=$DUMPDIR/$DBNAME.dmp $DBNAME  >> $DUMPDIR/$DBNAME.dmp.log 2>&1
 

scp /backup/data/pg_dump/name_db.dmp mgb@destination_serer:/tmp/
 

Заливаем в postgres – по ошибке!!!:

time psql -U postgres < /backup/dump/prod_sut.dmp > /backup/dump/prod_sut.log 2>&1
 

Правильно!!!:

CREATE DATABASE test_test owner owner_name_user;
На целевом сервере:

time psql -U postgres -d test_test < /backup/dump/prod_sut_.dmp >> /backup/dump/prod_sut.dmp.log 2>&1
 

 

Блокировка целевой БД test_test:

psql -U postgres -c "UPDATE pg_database SET datallowconn = true WHERE datname='test_test';"
 

Удаление целевой БД на целевом сервере

DROP DATABASE "test_test";
 

Создаем целевую БД test_test, уже без данных с owner_целевого сервера:

CREATE DATABASE test_test owner owner_test;
 

Копирование данных  из дампа исх.БД в целевую БД

time psql -U postgres -d test_test < /backup/dump/prod_sut_rusburm.dmp >> /backup/dump/prod_sut_rusburm.dmp.log 2>&1
 

```























