#### Размер базы данных
```
# Не удобный вывод:
SELECT pg_database_size(current_database());
# Удобный, т.е. сокращенный по OTUS:
SELECT pg_size_pretty(pg_database_size(current_database()));
# Ответ:  pg_size_pretty 4275 MB
```
#### Вывод всех таблиц текущей БД
```
SELECT table_name FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema','pg_catalog');
```

```


```








































