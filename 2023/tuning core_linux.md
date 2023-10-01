



##### Просмотр SWAPON
```
sudo swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/sda5 partition 975M   0B   -2
```
```
cat /proc/sys/vm/swappiness
60 - дефолтное значение
sudo sysctl vm.swappiness=10
cat /proc/sys/vm/swappiness
10
cat /proc/sys/vm/overcommit_ratio
50
cat /proc/sys/vm/overcommit_memory
0
```
```
OOM-Killer можно не только включать и выключать. Мы уже говорили, что Linux может зарезервировать для процессов больше памяти, чем есть, но не выделять ее по факту, и этим поведением управляет параметр ядра Linux. За это отвечает переменная vm.overcommit_memory.

Для нее можно указывать следующие значения:

0: ядро само решает, стоит ли резервировать слишком много памяти. Это значение по умолчанию в большинстве версий Linux.
1: ядро всегда будет резервировать лишнюю память. Это рискованно, ведь память может закончиться, потому что, скорее всего, однажды процессы затребуют положенное.
2: ядро не будет резервировать больше памяти, чем указано в параметре overcommit_ratio.
```
```
root@backup-restore:/backup# echo 1 > /proc/sys/vm/overcommit_memory  -vim не проходит
root@backup-restore:/backup# cat /proc/sys/vm/overcommit_memory
1
```





























