# Установка postgrespro-std-13
#### Просмотр доступных пакетов ДО:
```
apt list | grep -E "postgresql|postgrespro"
yum list | grep -E "postgresql|postgrespro"
apt info postgrespro-std-14-server
```
# Установка репозитория postgrespro-std-13
```
root@etcd:/home/mgb# curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-13/keys/pgpro-repo-add.sh
apt-get update
apt-get install postgrespro-std-13 -y
```
#### Просмотр доступных пакетов после установки репозитория:
```
apt list | grep -E "postgrespro"
```
### Установка v_14
```
curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
root@etcd:/home/mgb# curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-14/keys/pgpro-repo-add.sh
apt-get update
apt-get install postgrespro-std-14
systemctl status postgrespro-std-14
# Новая версия 14 установлена.
```
### Установка V_15
```
wget http://repo.postgrespro.ru/std-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
sudo apt update
sudo apt install postgrespro-std-15
# Проверка
systemctl status postgrespro-std-15.service
● postgrespro-std-15.service - Postgres Pro std 15 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-std-15.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-10-13 09:59:54 MSK; 6min ago
    Process: 2839 ExecStartPre=/opt/pgpro/std-15/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 2841 (postgres)
      Tasks: 7 (limit: 4662)
     Memory: 48.5M
        CPU: 376ms
     CGroup: /system.slice/postgrespro-std-15.service
             ├─2841 /opt/pgpro/std-15/bin/postgres -D /var/lib/pgpro/std-15/data
```
```
# БД уст. в /var/lib/pgpro/std-15/data
# Добавит установленные программы в путь поиска PATH:
/opt/pgpro/std-15/bin/pg-wrapper links updatemc
echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

```
##### Установка make
```
apt install gmake
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
E: Невозможно найти пакет gmake
root@etcd:/home/mgb# sudo apt install make
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Предлагаемые пакеты:
  make-doc
Следующие НОВЫЕ пакеты будут установлены:
  make
Обновлено 0 пакетов, установлено 1 новых пакетов, для удаления отмечено 0 пакетов, и 125 пакетов не обновлено.
Необходимо скачать 396 kB архивов.
После данной операции объём занятого дискового пространства возрастёт на 1 630 kB.
Пол:1 http://deb.debian.org/debian bullseye/main amd64 make amd64 4.3-4.1 [396 kB]
Получено 396 kB за 0с (1 135 kB/s)
Выбор ранее не выбранного пакета make.
(Чтение базы данных … на данный момент установлен 104561 файл и каталог.)
Подготовка к распаковке …/make_4.3-4.1_amd64.deb …
Распаковывается make (4.3-4.1) …
Настраивается пакет make (4.3-4.1) …
Обрабатываются триггеры для man-db (2.9.4-2) …
```
##### Подготовка к установке pg_repack:
```
apt -y install build-essential -y
sudo apt-get install postgresql-server-dev-all -y
sudo apt-get install postgresql-common -y
apt-get install pgxnclient libpq-dev -y
sudo apt-get install libreadline8
sudo apt-get install libreadline-dev
sudo apt-get install libssl-dev
sudo apt install libgl1-mesa-dev
apt install liblz4-dev
apt-get install zlib1g-dev
```
#### Установка pg_repack:
```
wget http://api.pgxn.org/dist/pg_repack/1.4.5/pg_repack-1.4.5.zip
unzip pg_repack-1.4.5.zip
cd pg_repack-1.4.5
root@etcd:/home/mgb/pg_repack-1.4.8# make
make[1]: вход в каталог «/home/mgb/pg_repack-1.4.8/bin»
Makefile:34: /usr/lib/postgresql/13/lib/pgxs/src/makefiles/pgxs.mk: Нет такого файла или каталога
make[1]: *** Нет правила для сборки цели «/usr/lib/postgresql/13/lib/pgxs/src/makefiles/pgxs.mk».  Останов.
make[1]: выход из каталога «/home/mgb/pg_repack-1.4.8/bin»
make: *** [Makefile:35: all] Ошибка 2

# По другому:
# Before building, you might need to install the PostgreSQL development packages (postgresql-devel, etc.):
libpq-dev

add the directory containing pg_config to your $PATH
and add the directory containing pg_config to your $PATH:
# PATH=/usr/lib/postgresql/14/bin:$PATH
export PATH=/opt/pgpro/std-13/bin:$PATH
export PATH="$PATH:/usr/lib/postgresql/13/bin"
echo $PATH
root@etcd:/home/mgb# pgxn install pg_repack
INFO: best version: pg_repack 1.4.8
INFO: saving /tmp/tmpufzy46wd/pg_repack-1.4.8.zip
INFO: unpacking: /tmp/tmpufzy46wd/pg_repack-1.4.8.zip
INFO: building extension
ERROR: make executable not found: gmake
# Then you can run:
cd pg_repack
$ make
$ sudo make install
CREATE EXTENSION pg_repack;
```
##### Остановка postgrespro-std
```
systemctl stop postgrespro-std-13
systemctl stop postgrespro-std-14
systemctl stop postgrespro-std-15
```
### Список установленных пакетов
```
apt list installed | grep postgres
postgrespro-std-14-client/неизвестно,now 14.8.2-1.bullseye amd64 [установлен, автоматически]
postgrespro-std-14-contrib/неизвестно,now 14.8.2-1.bullseye amd64 [установлен, автоматически]
postgrespro-std-14-libs/неизвестно,now 14.8.2-1.bullseye amd64 [установлен, автоматически]
postgrespro-std-14-server/неизвестно,now 14.8.2-1.bullseye amd64 [установлен, автоматически]
postgrespro-std-14/неизвестно,now 14.8.2-1.bullseye amd64 [установлен]
# Без лишней информации:
dpkg --get-selections | grep -v deinstall | grep postgres
postgrespro-std-14                              install
postgrespro-std-14-client                       install
postgrespro-std-14-contrib                      install
postgrespro-std-14-libs:amd64                   install
postgrespro-std-14-server                       install
```
### Удаление пакетов postgrespro-std:
```
apt purge postgrespro-std-14
# Удалит зависимые пакеты, что бы не удалять все в ручную:
apt autoremove
dpkg --get-selections | grep -v deinstall | grep postgres
```
#### Подготовка директорий:
```
mkdir -p /data/pg_data
mkdir -p /wal/pg_wal
```
#### Пропускаем - Инициалицация нового кластера v_14:
```
/opt/pgpro/std-14/bin/pg-setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data/pg_data --waldir=/wal/pg_wal  # pg-setup через дефис
/opt/pgpro/std-15/bin/pg-setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data/pg_data --waldir=/wal/pg_wal
If you want to setup second postgres instance in /data/pg_data use /opt/pgpro/std-14/bin/initdb directly and configure service startup manually. USE:
/opt/pgpro/std-15/bin/initdb directly
```
# Проверка перед обновлением:
```
mkdir -p /pg_upgrade
cd /pg_upgrade
/opt/pgpro/std-14/bin/pg_upgrade
```
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt install libpq-dev

vim /etc/apt/sources.list
deb http://ftp.de.debian.org/debian buster main
deb http://ftp.de.debian.org/debian bullseye main 
````
```
root@etcd:/home/mgb# pg_config
BINDIR = /usr/lib/postgresql/13/bin
DOCDIR = /usr/share/doc/postgresql-doc-13
HTMLDIR = /usr/share/doc/postgresql-doc-13
INCLUDEDIR = /usr/include/postgresql
PKGINCLUDEDIR = /usr/include/postgresql
INCLUDEDIR-SERVER = /usr/include/postgresql/13/server
LIBDIR = /usr/lib/x86_64-linux-gnu
PKGLIBDIR = /usr/lib/postgresql/13/lib
LOCALEDIR = /usr/share/locale
MANDIR = /usr/share/postgresql/13/man
SHAREDIR = /usr/share/postgresql/13
SYSCONFDIR = /etc/postgresql-common
PGXS = /usr/lib/postgresql/13/lib/pgxs/src/makefiles/pgxs.mk
```
```
make PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config
```
```
cd pg_repack
PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config make
PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config make install
```
### Установка v_15
```
wget https://repo.postgrespro.ru/std-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
apt-get update
apt-get install postgrespro-std-15
```
```
root@etcd:/home/mgb# systemctl status postgrespro-std-15
● postgrespro-std-15.service - Postgres Pro std 15 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-std-15.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-06-09 15:56:41 MSK; 1min 54s ago
    Process: 2904 ExecStartPre=/opt/pgpro/std-15/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 2906 (postgres)
      Tasks: 7 (limit: 4662)
     Memory: 47.4M
        CPU: 263ms
     CGroup: /system.slice/postgrespro-std-15.service
             ├─2906 /opt/pgpro/std-15/bin/postgres -D /var/lib/pgpro/std-15/data
             ├─2907 postgres: logger
             ├─2908 postgres: checkpointer
             ├─2909 postgres: background writer
             ├─2911 postgres: walwriter
             ├─2912 postgres: autovacuum launcher
             └─2913 postgres: logical replication launcher

июн 09 15:56:41 etcd systemd[1]: Starting Postgres Pro std 15 database server...
июн 09 15:56:41 etcd postgres[2906]: 2023-06-09 15:56:41.258 MSK [2906] СООБЩЕНИЕ:  передача вывода в протокол процессу сбора протоколов
июн 09 15:56:41 etcd postgres[2906]: 2023-06-09 15:56:41.258 MSK [2906] ПОДСКАЗКА:  В дальнейшем протоколы будут выводиться в каталог "log".
июн 09 15:56:41 etcd systemd[1]: Started Postgres Pro std 15 database server.
```




















