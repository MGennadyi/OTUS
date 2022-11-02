# Ускорение с помощью конфигурации базы и оптимизации запросов
```
create table users
(
  id bigint primary key,
  login varchar(200) not null,
  first_name varchar(200) not null,
  last_name varchar(200) not null,
  create_date timestamp not null default now()
);
```
```
CREATE TABLE
insert into users
select id, random() * id, md5(sin(id)::text), md5(cos(id)::text)
from generate_series(1, 10000000) id;
```
```
analyze users;
# Ответ: ANALYZE
```
```
\timing
```
```
select count(distinct id) from users;
# Ответ:

```
































