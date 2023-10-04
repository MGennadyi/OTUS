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













