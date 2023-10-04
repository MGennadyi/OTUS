# LVM
```
fdisk -l
lsblk
```
```
[root@localhost mgb]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
```
[root@localhost mgb]# pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda2  ro_redos lvm2 a--   <9,26g      0
  /dev/sdb            lvm2 ---  <20,41g <20,41g
```
```
[root@localhost mgb]# vgcreate vg_backup /dev/sdb
  Volume group "vg_backup" successfully created
```
```
[root@localhost mgb]# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda2  ro_redos  lvm2 a--   <9,26g      0
  /dev/sdb   vg_backup lvm2 a--  <20,41g <20,41g
```
```
root@localhost mgb]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ro_redos    1   2   0 wz--n-  <9,26g      0
  vg_backup   1   0   0 wz--n- <20,41g <20,41g
```
### Инф-я о логических томах:
```
[root@localhost mgb]# lvs
  LV   VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root ro_redos -wi-ao----  8,23g
  swap ro_redos -wi-ao---- <1,03g
```
### Создание логического тома
```
[root@localhost mgb]# lvcreate vg_backup -n lv_backup --size 10GB
  Logical volume "lv_backup" created.
```
### Просмотр логических томов:
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









