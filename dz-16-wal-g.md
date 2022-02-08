# WAL-G

###### 1. Установка Postgresql-13
###### 2. Установка WAL-G
```
wget https://github.com/wal-g/wal-g/releases/download/v1.1.2-rc/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -zxvf /home/mgb/wal-g-pg-ubuntu-20.04-amd64.tar.gz
mkdir /usr/local/bin/wal-g
chown -R postgres /usr/local/bin/wal-g
rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod -R 777 /home/backups
su postgres
mkdir /var/lib/postgresql/14/main/log
vim ~/.walg.json
{
    "WALG_FILE_PREFIX": "/home/backups",

    "WALG_COMPRESSION_METHOD": "brotli",

    "WALG_DELTA_MAX_STEPS": "5",

    "PGDATA": "/var/lib/postgresql/13/main",

    "PGHOST": "/var/run/postgresql/.s.PGSQL.5432"
}
```

###### DESC: сжатие=brotli; delta=5;




