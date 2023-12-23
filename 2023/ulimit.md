# ulimit
### Настройка лимитов для пользователя
```
vim /etc/security/limits.conf
```
 ### Изменение лимита навсегда:
```
vim /etc/sysctl.conf
# Однако, в конфигах нет явно указанных параметров
```

```
ulimit -f unlimited
mgb@astra:~$ ulimit -a postgres
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) 25000000
pending signals                 (-i) 7548
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 2048
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 1000
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```








