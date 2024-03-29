# LVM
#### Есть sdb для /backup, sdc для /wal
```
[root@localhost mgb]# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                 8:0    0 10,3G  0 disk
├─sda1              8:1    0    1G  0 part /boot
└─sda2              8:2    0  9,3G  0 part
  ├─ro_redos-root 253:0    0  8,2G  0 lvm  /
  └─ro_redos-swap 253:1    0    1G  0 lvm  [SWAP]
sdb                 8:16   0 20,4G  0 disk
sdc                 8:32   0    8G  0 disk
sr0                11:0    1 1024M  0 rom
```
### Создание физического тома /dev/sdb
```
[root@localhost mgb]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
#### Смотрим, что получилось
```
[root@localhost mgb]# pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda2  ro_redos lvm2 a--   <9,26g      0
  /dev/sdb            lvm2 ---  <20,41g <20,41g
```
### Создание Volume group "vg_backup": 
```
vgcreate vg_backup /dev/sdb
Volume group "vg_backup" successfully created
```
#### Смотрим, что получилось:
```
root@localhost mgb]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ro_redos    1   2   0 wz--n-  <9,26g      0
  vg_backup   1   0   0 wz--n- <20,41g <20,41g
```
#### Инф-я о логических томах:
```
[root@localhost mgb]# lvs
  LV   VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root ro_redos -wi-ao----  8,23g
  swap ro_redos -wi-ao---- <1,03g
```
### Создание логического тома lv_backup на 10Gb
```
[root@localhost mgb]# lvcreate vg_backup -n lv_backup --size 10GB
  Logical volume "lv_backup" created.
```
#### Просмотр логических томов:
```
[root@localhost mgb]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg_backup/lv_backup
```
### Удаление логического тома /dev/vg_backup/lv_backup:
```
[root@localhost mgb]# lvremove /dev/vg_backup/lv_backup
Do you really want to remove active logical volume vg_backup/lv_backup? [y/n]: y
  Logical volume "lv_backup" successfully removed
```
### Создание еще раз логического тома lv_backup на 20Gb
```
[root@localhost mgb]# lvcreate vg_backup -n lv_backup --size 20g
  Logical volume "lv_backup" created.
```
#### Что получилось:
```
[root@localhost mgb]# lvs
  LV        VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      ro_redos  -wi-ao----  8,23g
  swap      ro_redos  -wi-ao---- <1,03g
  lv_backup vg_backup -wi-a----- 20,00g
```
#### devmapper
```
[root@localhost mgb]# ls -la /dev/mapper/vg_backup-lv_backup
lrwxrwxrwx. 1 root root 7 окт  4 10:45 /dev/mapper/vg_backup-lv_backup -> ../dm-2
```
### Создание файловой системы:
```
[root@localhost mgb]# mkfs.ext4 /dev/mapper/vg_backup-lv_backup
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 5242880 4k blocks and 1310720 inodes
```
### Монтирование /backup
```
mkdir /backup
mount /dev/mapper/vg_backup-lv_backup /backup
[root@localhost mgb]# df -h
Файловая система                Размер Использовано  Дост Использовано% Cмонтировано в
devtmpfs                          4,0M            0  4,0M            0% /dev
tmpfs                             2,0G            0  2,0G            0% /dev/shm
tmpfs                             786M         1,9M  784M            1% /run
/dev/mapper/ro_redos-root         8,1G         2,3G  5,3G           31% /
/dev/sda1                         974M          66M  841M            8% /boot
tmpfs                             393M            0  393M            0% /run/user/1000
/dev/mapper/vg_backup-lv_backup    20G          24K   19G            1% /backup
```
### RESIZE файловой системы:
```
1.
2.
pvresize
```
### Работа с  sdc для /wal
```
[root@localhost mgb]# lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                     8:0    0 10,3G  0 disk
├─sda1                  8:1    0    1G  0 part /boot
└─sda2                  8:2    0  9,3G  0 part
  ├─ro_redos-root     253:0    0  8,2G  0 lvm  /
  └─ro_redos-swap     253:1    0    1G  0 lvm  [SWAP]
sdb                     8:16   0 20,4G  0 disk
└─vg_backup-lv_backup 253:2    0   20G  0 lvm  /backup
sdc                     8:32   0    8G  0 disk
sr0                    11:0    1 1024M  0 rom
```

```
pvcreate /dev/sdc
```
```
vgcreate vg_wal /dev/sdc
```
```
lvcreate vg_wal -n lv_wal --size 8GB
```
```
mkfs.ext4 /dev/mapper/vg_wal-lv_wal
```
```
mkdir /wal
```
### FSTAB
```
vim /etc/fstab
# Для /backup
/dev/mapper/vg_backup-lv_backup                     /backup         ext4    defaults        1 1
/dev/mapper/vg_wal-lv_wal                     /wal         ext4    defaults        1 1
/dev/mapper/vg_data-lv_data                     /data         ext4    defaults        1 1
/dev/mapper/vg_log-lv_log                     /log         ext4    defaults        1 1
# Для временных файлов:
tmpfs /tempdb tmpfs size=1G,uid=postgres,gid=postgres 0 0
# Для логов:
# tmpfs /log tmpfs size=100M,uid=postgres,gid=postgres 0 0
```
### Добавление/создание нового тома /DATA на sdd: 
```
pvcreate /dev/sdd
# Что получилось:
[root@192 mgb]# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda2  ro_redos  lvm2 a--   <9,26g       0
  /dev/sdb   vg_backup lvm2 a--  <20,41g  416,00m
  /dev/sdc   vg_wal    lvm2 a--   <8,00g 1020,00m
  /dev/sdd             lvm2 ---    8,00g    8,00g
```
### Создание нового Volume group "vg_data": 
```
vgcreate vg_data /dev/sdd
Volume group "vg_data" successfully created
```
### Создание логического тома lv_data на 8Gb
```
lvcreate vg_data -n lv_data --size 7.9GB
Rounding up size to full physical extent 7,90 GiB
Logical volume "lv_data" created.
```
### Создание файловой системы:
```
[root@localhost mgb]# mkfs.ext4 /dev/mapper/vg_data-lv_data
Creating filesystem with 2071552 4k blocks and 518144 inodes
Filesystem UUID: 892063d7-314b-435b-ab7d-5420e4e8ac63
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632
Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```
### Монтирование /backup
```
mkdir /data
mount /dev/mapper/vg_data-lv_data /data
[root@localhost mgb]# df -h
Файловая система                Размер Использовано  Дост Использовано% Cмонтировано в
devtmpfs                          4,0M            0  4,0M            0% /dev
tmpfs                             2,0G            0  2,0G            0% /dev/shm
tmpfs                             786M         1,9M  784M            1% /run
/dev/mapper/ro_redos-root         8,1G         2,3G  5,3G           31% /
/dev/sda1                         974M          66M  841M            8% /boot
tmpfs                             393M            0  393M            0% /run/user/1000
/dev/mapper/vg_backup-lv_backup    20G          24K   19G            1% /backup
```
### Добавление/создание нового тома /log на sde: 
```
[root@192 mgb]# pvcreate /dev/sde
  Physical volume "/dev/sde" successfully created.
# Что получилось:
[root@192 mgb]# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda2  ro_redos  lvm2 a--   <9,26g       0
  /dev/sdb   vg_backup lvm2 a--  <20,41g  416,00m
  /dev/sdc   vg_wal    lvm2 a--   <8,00g 1020,00m
  /dev/sdd   vg_data   lvm2 a--   <8,00g   96,00m
  /dev/sde             lvm2 ---    2,50g    2,50g
```
### Создание нового Volume group "vg_log": 
```
vgcreate vg_log /dev/sde
Volume group "vg_data" successfully created
```
### Создание логического тома lv_data на 8Gb
```
[root@192 mgb]# lvcreate vg_log -n lv_log --size 2.5GB
  Logical volume "lv_log" created.
```
### Создание файловой системы:
```
[root@localhost mgb]# mkfs.ext4 /dev/mapper/vg_log-lv_log
Creating filesystem with 2071552 4k blocks and 518144 inodes
Filesystem UUID: 892063d7-314b-435b-ab7d-5420e4e8ac63
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632
Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```
### Монтирование /backup
```
mkdir /log
mount /dev/mapper/vg_log-lv_log /log
[root@localhost mgb]# df -h
Файловая система                Размер Использовано  Дост Использовано% Cмонтировано в
devtmpfs                          4,0M            0  4,0M            0% /dev
tmpfs                             2,0G            0  2,0G            0% /dev/shm
tmpfs                             786M         1,9M  784M            1% /run
/dev/mapper/ro_redos-root         8,1G         2,3G  5,3G           31% /
/dev/sda1                         974M          66M  841M            8% /boot
tmpfs                             393M            0  393M            0% /run/user/1000
/dev/mapper/vg_backup-lv_backup    20G          24K   19G            1% /backup
```
