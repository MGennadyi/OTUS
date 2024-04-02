# Удаление postgresql
```
dpkg --get-selections | grep -v deinstall | grep postgres
postgresql-14                                   install
postgresql-client-14                            install
postgresql-client-common                        install
postgresql-common                               install
```
```
apt purge postgresql-14 -y
apt purge postgresql-client-14 -y
apt purge postgresql-client-common -y
apt purge postgresql-common -y
```






