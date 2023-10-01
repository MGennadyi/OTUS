



##### Просмотр SWAPON
```sudo swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/sda5 partition 975M   0B   -2
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































