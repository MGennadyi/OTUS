# Установка Postgresql v_11
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt update 
apt install postgresql-11 -y
```
```
# Проверка что под капотом:
```
lsb_release -a
pg_lsclusters
11  main    5432 online postgres /var/lib/postgresql/11/main /var/log/postgresql/postgresql-11-main.log
dpkg -l | grep postgresql
ii  postgresql-11                         11.18-1.pgdg110+1               amd64        The World's Most Advanced Open Source Relational Database
ii  postgresql-client-11                  11.18-1.pgdg110+1               amd64        front-end programs for PostgreSQL 11
ii  postgresql-client-common              246.pgdg110+1                   all          manager for multiple PostgreSQL client versions
ii  postgresql-common                     246.pgdg110+1                   all          PostgreSQL database-cluster manager
```



