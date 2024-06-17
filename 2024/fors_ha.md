#### pg1
```
pg_lsclusters # первонач.состян=красн
sudo systemctl status postgres
sudo systemctl start postgres  # loaded; enabled; active
pg_lsclusters # зеленый
```
```
postgres=# select pg_is_in_recovery();
pg_is_in_recovery
-------------------
t
(1 row)
```
#### pg4
```
pg_isready # 5432-noresponce
sudo systemctl status postgresql@12-main.service # loaded; disabled; inactive
sudo vi /etc/postgres/12/main/start.conf
disabled > auto
sudo systemctl daemon-reload
sudo systemctl stop postgre
sudo systemctl status postgresql@12-main.service # loaded; enabled; active
pg_lsclusters # зеленый
```
```
postgres=# select pg_is_in_recovery();
pg_is_in_recovery
-------------------
f
(1 row)
```
