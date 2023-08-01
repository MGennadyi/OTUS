# Роли, права.
#### 1. Заведение нового пользователя с правом входа:
```
CREATE USER gbsemkin WITH LOGIN password '12345';  user-с правом входа
```
#### 2. Создание роли в СУБД (если нет роли):
```
# Назначаем роль на БД demo:
bash /postgres/scripts/create_role/create_readonly_group.sh --list demo
Grant privileges from readonly_role:
List databases: demo
Granted role "readonly_role_demo" for database "demo" successfully.
Role creation comleted.
```
#### 3. Присвоение/назначение пользователю роль: 
```
grant readonly_role to gbsemkin;
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


































