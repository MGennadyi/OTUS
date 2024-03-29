# Установка postgrespro-std
### Проверка локали:
```
localectl status
System Locale: LANG=ru_RU.UTF-8
       VC Keymap: n/a
      X11 Layout: us,ru
       X11 Model: pc105
     X11 Variant: ,
     X11 Options: grp:alt_shift_toggle,grp_led:scroll
# или
locale -a
C
C.UTF-8
POSIX
ru_RU.utf8
# Изменение локали:
localectl set-locale LANG=en_US.utf-8
localectl set-locale LANG=en_US.utf-8
root@etcd:/home/mgb# localectl status
   System Locale: LANG=en_US.utf-8
       VC Keymap: n/a
      X11 Layout: us,ru
       X11 Model: pc105
     X11 Variant: ,
     X11 Options: grp:alt_shift_toggle,grp_led:scroll
После перезагрузки:
# locale
LANG=en_US.utf-8
LANGUAGE=
LC_CTYPE="en_US.utf-8"
LC_NUMERIC="en_US.utf-8"
LC_TIME="en_US.utf-8"
LC_COLLATE="en_US.utf-8"
LC_MONETARY="en_US.utf-8"
LC_MESSAGES="en_US.utf-8"
LC_PAPER="en_US.utf-8"
LC_NAME="en_US.utf-8"
LC_ADDRESS="en_US.utf-8"
LC_TELEPHONE="en_US.utf-8"
LC_MEASUREMENT="en_US.utf-8"
LC_IDENTIFICATION="en_US.utf-8"
LC_ALL=
```
### Установка postgrespro-std-13
```
curl -o pgpro-repo-add.sh https://repo.postgrespro.ru/pgpro-13/keys/pgpro-repo-add.sh
apt-get update
apt-get install postgrespro-std-13 -y
```
#### Просмотр доступных пакетов после установки репозитория:
```
apt list | grep -E "postgresql|postgrespro"
yum list | grep -E "postgresql|postgrespro"
apt info postgrespro-std-14-server
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
### Установка V_15 Вариант по умолчанию:
```
wget http://repo.postgrespro.ru/std-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
sudo yum update
# Вариант установки № 1 - # Установка по умолчанию, в том числе и инициализация:
yum install postgrespro-std-15 
apt install postgrespro-std-15         
dpkg --get-selections | grep -v deinstall | grep postgres   # Проверка установленных пакетов
postgrespro-std-15                              install
postgrespro-std-15-client                       install
postgrespro-std-15-contrib                      install
postgrespro-std-15-libs:amd64                   install
postgrespro-std-15-server                       install
# Проверка установки. Пути дефолтные:
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
### Установка V_15 Вариант по заданными параметрами через отдельную инициализацию БД:
```
# Вариант установки № 2 - # Установка с заданными параметрами через отдельную инициализацию БД:
yum install postgrespro-std-15-server
apt install postgrespro-std-15-server  
dpkg --get-selections | grep -v deinstall | grep postgres   # Проверка установленных пакетов
postgrespro-std-15-client                       install
postgrespro-std-15-contrib                      install
postgrespro-std-15-libs:amd64                   install
postgrespro-std-15-server                       install
```
### Проверка запуска службы:
```
systemctl status postgrespro-std-15.service
● postgrespro-std-15.service - Postgres Pro std 15 database server
     Loaded: loaded (/lib/systemd/system/postgrespro-std-15.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-10-15 18:33:42 MSK; 1s ago
    Process: 1005 ExecStartPre=/opt/pgpro/std-15/bin/check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 1007 (postgres)
      Tasks: 7 (limit: 4662)
     Memory: 45.4M
        CPU: 93ms
     CGroup: /system.slice/postgrespro-std-15.service
             ├─1007 /opt/pgpro/std-15/bin/postgres -D /data/pg_data
             ├─1008 postgres: logger
             ├─1009 postgres: checkpointer
             ├─1010 postgres: background writer
             ├─1012 postgres: walwriter
             ├─1013 postgres: autovacuum launcher
             └─1014 postgres: logical replication launcher
# Автозапуск/ручной запуск службы:
systemctl enable postgrespro-std-15.service  # Loaded: enabled;
systemctl disable postgrespro-std-15.service # Loaded: disabled;
```
### Запуск не получится:
```
postgres@etcd:~$ psql
-bash: psql: command not found
```
### !!! Добавить установленные пакеты postgres-pro в путь поиска PATH pg-wrapper:  обязательно!!! -из-под root
```
# Добавит установленные программы в путь поиска PATH:
[root@192 wal]# /opt/pgpro/std-15/bin/pg-wrapper links update
Updating /etc/man_db.conf

echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```
### Список установленных пакетов DEBIAN
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
------------------
postgrespro-std-15-client                       install
postgrespro-std-15-contrib                      install
postgrespro-std-15-libs:amd64                   install
postgrespro-std-15-server                       install
```
### Список установленных пакетов RedOS
```
[root@192 mgb]# yum list installed | grep postgres
postgrespro-std-15-client.x86_64         15.4.2-1.el7                   @postgrespro-std-15
postgrespro-std-15-libs.x86_64           15.4.2-1.el7                   @postgrespro-std-15
postgrespro-std-15-server.x86_64         15.4.2-1.el7                   @postgrespro-std-15
postgresql.x86_64                        12.12-2.el7                    @updates
```

### Удаление пакетов postgrespro-std -RedOS:
```
yum remove postgrespro-std-15-server
yum remove postgrespro-std-15-client
yum remove postgrespro-std-15-libs
yum remove postgresql
rm /etc/dafault/postgrespro-std-15
rm /initdb.data.log
----------------
apt remove postgrespro-std-15
apt purge postgrespro-std-15
# Удалит зависимые пакеты, что бы не удалять все в ручную:
apt autoremove
dpkg --get-selections | grep -v deinstall | grep postgres
```
#### Подготовка директорий перед инициализацией:
```
mkdir -p /data/pg_data
chown -R postgres:postgres /data
mkdir -p /wal/pg_wal
chown -R postgres:postgres /wal
mkdir -p /backup/wal_arc_archive
chown -R postgres:postgres /backup
mkdir -p /log/pg_log
chown -R postgres:postgres /log/
mkdir -p /postgres/scripts
chown -R postgres:postgres /postgres
```
### Инициалицация нового кластера v_14 v_15 из-под root:
```
/opt/pgpro/std-15/bin/pg-setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data/pg_data --waldir=/wal/pg_wal
[root@192 wal]# /opt/pgpro/std-15/bin/pg-setup initdb --data-checksums --locale=en_US.utf8 --pgdata=/data --waldir=/wal
Initalizing database...
OK
# Просмотр лога:
[postgres@192 ~]$ cat /initdb.data.log
Файлы, относящиеся к этой СУБД, будут принадлежать пользователю "postgres".
От его имени также будет запускаться процесс сервера.

Кластер баз данных будет инициализирован со следующими параметрами локали:
  провайдер:   icu
  локаль ICU:  en-US
  LC_COLLATE:  en_US.utf8
  LC_CTYPE:    en_US.utf8
  LC_MESSAGES: en_US.utf8
  LC_MONETARY: en_US.utf8
  LC_NUMERIC:  en_US.utf8
  LC_TIME:     en_US.utf8
В качестве кодировки БД по умолчанию установлена "UTF8".
Выбрана конфигурация текстового поиска по умолчанию "english".

Контроль целостности страниц данных включён.

исправление прав для существующего каталога /data... ок
исправление прав для существующего каталога /wal... ок
создание подкаталогов... ок
выбирается реализация динамической разделяемой памяти... posix
выбирается значение max_connections по умолчанию... 100
выбирается значение shared_buffers по умолчанию... 128MB
выбирается часовой пояс по умолчанию... Europe/Moscow
создание конфигурационных файлов... ок
выполняется подготовительный скрипт... ок
выполняется заключительная инициализация... ок
сохранение данных на диске... ок
Готово. Теперь вы можете запустить сервер баз данных:
/opt/pgpro/std-15/bin/pg_ctl -D /data -l файл_журнала start

```
```
ls -lhr /data/pg_data/
total 132K
-rw------- 1 postgres postgres   79 Oct 15 18:33 postmaster.pid
-rw------- 1 postgres postgres   52 Oct 15 18:33 postmaster.opts
-rw------- 1 postgres postgres  30K Oct 15 18:33 postgresql.conf
-rw------- 1 postgres postgres   88 Oct 15 18:33 postgresql.auto.conf
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_xact
lrwxrwxrwx 1 postgres postgres   11 Oct 15 18:33 pg_wal -> /wal/pg_wal
-rw------- 1 postgres postgres    3 Oct 15 18:33 PG_VERSION
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_twophase
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_tblspc
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_subtrans
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_stat_tmp
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_stat
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_snapshots
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_serial
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_replslot
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_notify
drwx------ 4 postgres postgres 4.0K Oct 15 18:33 pg_multixact
drwx------ 4 postgres postgres 4.0K Oct 15 18:38 pg_logical
-rw------- 1 postgres postgres 1.6K Oct 15 18:33 pg_ident.conf
-rw------- 1 postgres postgres 4.5K Oct 15 18:33 pg_hba.conf
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_dynshmem
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 pg_commit_ts
drwx------ 2 postgres postgres 4.0K Oct 15 18:33 log
drwx------ 2 postgres postgres 4.0K Oct 15 18:36 global
-rw------- 1 postgres postgres   44 Oct 15 18:33 current_logfiles
drwx------ 5 postgres postgres 4.0K Oct 15 18:33 base
```
Требуется настройка логов:
```
mcedit /data/pg_data/postgresql.auto.conf
log_directory = '/log/pg_log'
log_line_prefix = '%m [%p] %u@%d/%a'
/opt/pgpro/std-15/bin/pg_ctl -D /data/pg_data start
# Просмотр:
ls -lh /log/pg_log
ls -lh /wal/pg_wal
```
### /tempdb
```
ALTER SYSTEM SET stats_temp_directory = '/tempdb';
postgres=# select pg_reload_conf();
```







































```
cd pg_repack
PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config make
PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config make install
```
```
make PG_CONFIG=/usr/lib/postgresql/13/bin/pg_config
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



















