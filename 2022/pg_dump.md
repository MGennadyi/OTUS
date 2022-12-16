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
# Простой вариант бекапа, в SQL-скрипте :
time pg_dump -d demo --create > /home/backups/pg_dump/demo1.sql
# Ответ: real    0m1,888s 0m1,917s
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

# Архив с оглавлением для pg_restore. : 
time pg_dump -d demo -Fc > /home/backups/pg_dump/demo11.gz
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
unzip demo_small.zip
chown postgres /home/mgb/demo_small.sql
ls -la
demo_small.sql
su postgres
CREATE DATABASE demo;
psql -d demo < demo_small.sql
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












