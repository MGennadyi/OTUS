# Копирование БД
### Выгоняем всех пользователей из БД otus
```
psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='otus';"
```
### Блокировка целевой БД test_test:
```
psql -U postgres -c "UPDATE pg_database SET datallowconn = false WHERE datname='test_test';"
psql -U postgres -c "UPDATE pg_database SET datallowconn = true WHERE datname='test_test';"
 ```
### dumpall
```
cd /backup
 mkdir 2023_08_30
 WORKDIR=2023_08_30
 DUMPALLDIR=/backup/$WORKDIR
 time pg_dumpall --quote-all-identifiers --verbose --file $DUMPALLDIR/dumpall.dmp >> $DUMPALLDIR/dumpall.dmp.log 2>&1
 ```
### Проверка лога dumpall
```
grep -E "ERROR|ОШИБКА|DETAIL|CONTEXT|FATAL|ВАЖНО|ПАНИКА|PANIC|ПОДРОБНОСТИ|ПРЕДУПРЕЖДЕНИЕ" $DUMPALLDIR/dumpall.dmp.log
tail -n 10 $DUMPALLDIR/dumpall.dmp.log
tail -n 10 $DUMPALLDIR/dumpall.dmp
```
### dump БД-источник
```
time pg_dump -C -h localhost -U postgres 'bd_ist' > /data/copy_db/bd_ist.bac
```
### Замена инструкции в дампе: имя БД-источника на целевое имя БД-цель
```
sed -i "s/db_ist/bd_cel/g" bd_ist.bac
sed -i "s/prod_sut/test_test/g" prod_sut.dmp
```
### Создаю дамп целевой БД на целевом сервере
```
time pg_dump -U postgres test_test > /backup/dump/test_test.dmp
time pg_dump -U postgres -d test_test --create | gzip > /backup/dump/test_test.dmp.gz
```
### Скрипт pg_dump:
```
cat /backup/op286466.sh
# pg_dump
DUMPDIR=/backup/"$(date '+%Y_%m_%d')"/pg_dump
mkdir -p $DUMPDIR
# DBNAME=ИМЯ_БД (заполняется на этапе планирования)
DBNAME=name_db
time pg_dump -U postgres -v --file=$DUMPDIR/$DBNAME.dmp $DBNAME  >> $DUMPDIR/$DBNAME.dmp.log 2>&1
```
### Копирование полученного дампа на целевой сервер:
```
scp /backup/data/pg_dump/name_db.dmp mgb@destination_serer:/tmp/
```
### На целевом сервере:
### Замена инструкции в дампе: имя БД-источника на целевое имя БД-цель
```
sed -i "s/db_ist/bd_cel/g" bd_ist.bac
sed -i "s/prod_sut/test_test/g" prod_sut.dmp
```
### Загрузка НЕ ВЕРНО!!!:
```
time psql -U postgres < /backup/dump/prod_sut.dmp > /backup/dump/prod_sut.log 2>&1   - загрузится в БД postgres.
```
### Загрузка Правильно!!!:
### Сначала Удаление существующей целевой БД на целевом сервере
```
DROP DATABASE "test_test";
 ```
### Создаем пустую целевую БД test_test:
```
CREATE DATABASE test_test owner owner_test;
```
### Загрузка данных из дампа исх.БД в целевую БД:
```
time psql -U postgres -d test_test < /backup/dump/prod_sutыув.dmp >> /backup/dump/prod_sut.dmp.log 2>&1
time psql -U postgres -d test_test < /backup/dump/prod_sut_rusburm.dmp >> /backup/dump/prod_sut_rusburm.dmp.log 2>&1
 ```























