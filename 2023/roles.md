# Роли, права.
#### 1. Заведение нового пользователя с правом входа:
```
CREATE USER gbsemkin WITH LOGIN password '12345';  user-с правом входа, команда WITH LOGIN - лишняя.
CREATE USER gbsemkin password '12345';
ALTER USER "gren-d-DALysko" WITH LOGIN;  -задать право на логин
ALTER USER "gren-d-DALysko" WITH PASSWORD 'new_password'; -смена пароля
create USER iibutikov password '****' valid until '2023-09-15 23:59:00+03';
```
#### 2. Создание роли в СУБД (если нет роли):
```
# Назначаем роль на БД demo:
bash /postgres/scripts/create_role/create_readonly_group.sh --list demo
Grant privileges from readonly_role:
List databases: demo
Granted role "readonly_role_demo" for database "demo" successfully.
Role creation comleted.
-----------------------------------
postgres@backup-restore:/home/mgb$ bash /postgres/scripts/create_role/create_readonly_group.sh --list otus
Grant privileges from readonly_role:
List databases: otus
Role readonly_role_otus created successfully.
Granted role "readonly_role_otus" for database "otus" successfully.
Role creation comleted.

```
#### Роль на конкретную базу - создается роль_имя_базы:
```
demo=# \du+
                                                          Список ролей
      Имя роли      |                                Атрибуты                                 |      Член ролей      | Описание
--------------------+-------------------------------------------------------------------------+----------------------+----------
 arwd_role_demo     | Вход запрещён                                                           | {}                   |
 avesenin           |                                                                         | {readonly_role_demo} |
 evsemkin           |                                                                         | {arwd_role_demo}     |
 expert             |                                                                         | {}                   |
 gbsemkin           |                                                                         | {}                   |
 postgres           | Суперпользователь, Создаёт роли, Создаёт БД, Репликация, Пропускать RLS | {}                   |
 readonly_role_demo | Вход запрещён  
```
#### 3. Присвоение/назначение пользователю роль: 
```
GRANT readonly_role_demo to gbsemkin;
postgres=# GRANT readonly_role_demo to gbsemkin;
ЗАМЕЧАНИЕ:  роль "gbsemkin" уже включена в роль "readonly_role_demo"
GRANT ROLE
postgres=# GRANT readonly_role_otus to gbsemkin;
GRANT ROLE
```
#### Схема:
```
CREATE SCHEMA oil;
```
#### Создание групповой роли create_arwd_group.sh на БД demo;
```
# Правим строку пути к бинарниками в конфиге:
vim /postgres/scripts/create_group.config
bin_path=/usr/lib/postgresql/14/bin
# Вешаем роль на бд demo:
bash create_arwd_group.sh --list demo
postgres=# \l+ demo
                                                      Список баз данных
 Имя  | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |      Права доступа      | Размер  | Табл. пространство | Описание
------+----------+-----------+-------------+-------------+-------------------------+---------+--------------------+----------
 demo | expert   | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =Tc/expert             +| 8577 kB | pg_default         |
      |          |           |             |             | expert=CTc/expert      +|         |                    |
      |          |           |             |             | arwd_role_demo=c/expert |         |                    |
postgres=# grant readonly_role_demo to avesenin;
GRANT ROLE
postgres=# grant arwd_role_demo to evsemkin;
GRANT ROLE
```
#### Создание групповой роли create_readonly_group.sh на БД demo;
```
# Правим строку пути к бинарниками в конфиге:
vim /postgres/scripts/create_group.config
bin_path=/usr/lib/postgresql/14/bin
# Вешаем роль на бд demo:
bash create_readonly_group.sh --list demo
postgres=# \l+ demo
                                                      Список баз данных
 Имя  | Владелец | Кодировка | LC_COLLATE  |  LC_CTYPE   |      Права доступа      | Размер  | Табл. пространство | Описание
------+----------+-----------+-------------+-------------+-------------------------+---------+--------------------+----------
 demo | expert   | UTF8      | ru_RU.UTF-8 | ru_RU.UTF-8 | =Tc/expert             +| 8577 kB | pg_default         |
      |          |           |             |             | expert=CTc/expert      +|         |                    |
      |          |           |             |             | arwd_role_demo=c/expert |         |                    |
postgres=# grant readonly_role_demo to avesenin;
GRANT ROLE
postgres=# grant arwd_role_demo to evsemkin;
GRANT ROLE
```
### Исправляю ошибку:
```
create database "db1" with owner "owner_test";
```
```
CREATE USER owner_test password '12345';
ALTER USER owner_test WITH SUPERUSER;
ALTER TABLE tab_test OWNER TO owner_test;
ALTER TABLE employee_information OWNER TO owner_test;
create table test_table (column_name varchar(50)) OWNER TO owner_test;
```
```
select * from pg_tables where tableowner = 'owner_test';
 schemaname | tablename  | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity
------------+------------+------------+------------+------------+----------+-------------+-------------
 public     | test_table | owner_test |            | f          | f        | f           | f
```
```
DROP TABLE IF EXISTS test_table;
```
```
SELECT 'DROP TABLE ' || test_table || ' ' CASCADE;' \ FROM pg_catalog WHERE tableowner = 'owner_test';
> /tmp/droptables


```





























