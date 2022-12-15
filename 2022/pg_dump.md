# PG_DUMP
```
su postgres
mkdir /home/backups/1
```
```
# Задача теста: сравнение по времени, по потокам, загрузки CPU, загрузки hdd:
# При -Fc практически не будет сжатия; d (directory); -j (потоки); -F d (указ.формат.вывода d=директория) c (custom); -f (вывод в директорию путь обязателен);
# Простой вариант бекапа, в SQL-скрипте :
pg_dump -d otus --create > /home/backups/3.sql
# Простой со сжатием, в 2,8 раза меньше весит: 
pg_dump -d otus --create | gzip > /home/backups/otus3.gz
# Архив с оглавлением для pg_restore. Минимальная степень сжатия <10% : 
pg_dump -d otus -Fc > /home/backups/otus4.gz
# Восстановление-1:
drop database otus;
\q
psql < /home/backups/3.sql
# Восстановление-2 (drop/create/pg_restore):
echo "drop database otus;" | psql
echo "create database otus;" | psql
# Рассмотреть время выполнения в различных вариантах -j:
pg_restore -j 1 -d otus /home/backups/otus4.gz
pg_restore -j 2 -d otus /home/backups/otus4.gz
```
```
# Зальем данные:
cd ~
sudo wget --quiet https://edu.postgrespro.ru/demo_small.zip
# chown postgres /home/mgb/demo-small.zip
sudo unzip demo-small.zip
ls -la
demo-small-20170815.sql
CREATE DATABASE demo;
psql -d demo < demo-small-20170815.sql
# Ответ:
COPY 9
COPY 104
COPY 579686
COPY 262788
COPY 33121
 setval
--------
  33121

\l+
```
