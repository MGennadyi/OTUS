# Ускорение с помощью конфигурации базы и оптимизации запросов
```
\timing
```
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
###### Добавим в нее 10 млн записей
```
insert into users select id, random() * id, md5(sin(id)::text), md5(cos(id)::text) from generate_series(1, 10000000) id;
```
```
analyze users;
```
```
\timing
```
```
select count(distinct id) from users;
# Ответ: Время: 48999,199 мс (00:48,999)
```
###### Хорошо ьы просмотреть hints из ролика про мониторинг
###### Повторим
```
select count(distinct id) from users;
# Ответ: Время: 5930,707 мс (00:05,931) - разница в 10 раз
```































