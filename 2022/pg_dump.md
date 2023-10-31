# PG_DUMP
```
apt install time
```
```
su postgres
mkdir /home/backups/1
```
```
# Задача теста: сравнение по времени, по потокам, загрузки CPU, загрузки hdd:
# При -Fc практически не будет сжатия; d (directory); -j (потоки); -F d (указ.формат.вывода d=директория) c (custom); -f (вывод в директорию путь обязателен);
-------------------------------------
# Простой вариант бекапа, в SQL-скрипте :
pg_dump -U postgres dbname > outfile
psql -U postgres dbname < infile
-----------------------------------
# Ключ –inserts позволяет сделать бекап в формате SQL БД=otus; таблица=codes:
pg_dump -U postgres --inserts otus -t codes > codes_backup`date +%d.%m.%Y-%H.%M`.sql
------------------------------
time pg_dump -U postgres -d demo --create > /backups/pg_dump/demo1.sql
# Ответ: real    0m1,888s 0m1,917s
------------------------------
# Бекап только одной таблицы codes из БД otus
pg_dump -U postgres otus -t codes > codes_backup`date +%d.%m.%Y-%H.%M`.sql


# Простой со сжатием, в 2,8 раза меньше весит:
time pg_dump -d demo --create | gzip > /home/backups/demo1.gz
# real    0m5,829s 0m5,685s 0m5,682s
ls -lh /home/backups/pg_dump/
 22M demo1.gz
100M demo1.sql
time pg_dump -j 2 -Fd demo -f /home/backups/pg_dump/111
rm /home/backups/pg_dump/111/*
real    0m3,420s
du -sh /home/backups/pg_dump/111 22M
# -—format=формат
-—format=p — формирует текстовый SQL-скрипт;
-—format=c — формирует резервную копию в архивном формате;
-—format=d — формирует копию в directory-формате;
-—format=t — формирует копию в формате tar.
# В виде архива -Ft:
pg_dump -Ft zabbix > /backup/zabbix.tar
----------------------------
# Восстановление данных БД mbillcz5054 из сжатого бекапа. Создание исключенной таблицы.
gunzip -c mbillcz5054_backup.sql.gz | psql -U postgres mbillcz5054
# Архив с оглавлением для pg_restore (-Fc в виде бинарного файла): 
time pg_dump -d demo -Fc > /home/backups/pg_dump/demo11.gz
# Восстановление-1:
drop database otus;
\q
psql < /home/backups/3.sql
# Восстановление-2 (drop/create/pg_restore):
echo "drop database otus;" | psql
echo "create database otus;" | psql
-----------------------------
# БД testdb уже существует:
pg_restore -U postgres -Ft -d testdb < testdb.tar
# БД testdb нет:
pg_restore -U postgres -Ft -C -d testdb < testdb.tar
# Восстановление из файла резервной копии back_it.sql:
psql -f /backup/back_it.sql -U postgres 

# Рассмотреть время выполнения в различных вариантах -j:
pg_restore -j 1 -d otus /home/backups/otus4.gz
pg_restore -j 2 -d otus /home/backups/otus4.gz
```
### Загрузка демо-базы авиаперевозки:
```
# Зальем данные:
cd ~
sudo wget --quiet https://edu.postgrespro.ru/demo_small.zip
unzip demo_small.zip
root@pg:/home/mgb# ls -la
drwxr-xr-x 14 mgb  mgb       4096 фев 23 16:35  .
drwxr-xr-x  3 root root      4096 авг 21  2022  ..
-rw-rw-r--  1 root root 103857328 ноя 11  2016  demo_small.sql
-rw-r--r--  1 root root  22183920 авг 12  2019  demo_small.zip
# Меняем права на postgres для demo_small.sql:
chown postgres /home/mgb/demo_small.sql
ls -la - другое дело!
sudo -i -u postgres
psql -c 'CREATE DATABASE demo';
psql -d demo < /home/mgb/demo_small.sql
# Ответ:
COPY 9
COPY 104
COPY 579686
COPY 262788
COPY 33121
 setval
--------
  33121
postgres@pg:~$ psql -c '\l+';
                                                                        Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |     Права доступа     | Размер  | Табл. пространство |                  Описание
-----------+----------+-----------+-------------+-------------+-----------------------+---------+--------------------+--------------------------------------------
 demo      | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |                       | 280 MB  | pg_default         |
 postgres  | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 |                       | 7901 kB | pg_default         | default administrative connection database
 template0 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +| 7753 kB | pg_default         | unmodifiable empty database
           |          |           |             |             | postgres=CTc/postgres |         |                    |
 template1 | postgres | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =c/postgres          +| 7901 kB | pg_default         | default template for new databases
           |          |           |             |             | postgres=CTc/postgres |         |                    |
(4 строки)
```
#### Сравнение времени создания бекапов:
```
SQL-скрипт
real    0m1,888s 0m1,917s
gzip > /home/backups/demo1.gz
# real    0m5,829s 0m5,685s 0m5,682s

pg_probackup
real    0m16,929s


# WAL_G
real    0m5,125s
real    0m1,232s
```
```
pg_ctl -D /data/pg_data/ start
```
### PG_DUMPALL
```
pg_dumpall --quote-all-identifiers --verbose --file=/backup/dump/dumpall.dmp 2> /backup/dump/dumpall.dmp.log от Лосева работает
real    0m17,066s
# Создание:
pg_dump -U postgres --file=/backup/2023_08_11.sql имя_бд >> /backup/2023_08_11/имя_бд.log 2>&1
pg_dump -U postgres --file=/backup/2023_08_11.sql otus >> /backup/2023_08_11/otus.log 2>&1
grep -E "ERROR|ошибка|DETAIL|CONTEXT|FATAL" /backup/2023_08_11/имя_бд.log

# Восстановление:
psql -f /backup/dump/dump_all.dmp >> /backup/dump/import.log
psql -f /backup/dump/dumpall.dmp >> /backup/dump/dumpall_restore.dmp.log 2>&1 - от Лосева
# Просмотр лога
grep -E "ERROR|ОШИБКА|DETAIL|CONTEXT" /backup/dump/import.log
```
### PG_DUMP
```
DBNAME=otus
FNAME=`date +%F`_$DBNAME
DIRDUMP=/backup/dump
echo ----------------------------- pg_dump $FNAME
date
pg_dump -U postgres -v --file=$DIRDUMP/$FNAME.dmp $DBNAME  >> $DIRDUMP/$FNAME.dmp.log 2>&1
grep -E "ERROR|ОШИБКА|DETAIL|CONTEXT|FATAL" $DIRDUMP/$FNAME.dmp.log
tail -n 10 $DIRDUMP/$FNAME.dmp.log
tail -n 10 $DIRDUMP/$FNAME.dmp
ls -lah $DIRDUMP
date
```


