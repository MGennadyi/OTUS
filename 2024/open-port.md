###
###
```
yum install net-tools
```
```
[root@masterr mgb]# netstat -ltupan
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      969/sshd: /usr/sbin
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1247/master
tcp        0      0 192.168.0.18:22         192.168.0.12:50998      ESTABLISHED 1335/sshd: mgb [pri
tcp        0    144 192.168.0.18:22         192.168.0.12:50994      ESTABLISHED 1331/sshd: mgb [pri
tcp6       0      0 :::10050                :::*                    LISTEN      977/zabbix_agent2
tcp6       0      0 :::22                   :::*                    LISTEN      969/sshd: /usr/sbin
udp        0      0 192.168.0.18:68         192.168.0.1:67          ESTABLISHED 903/NetworkManager
udp        0      0 127.0.0.1:323           0.0.0.0:*                           919/chronyd
udp6       0      0 ::1:323                 :::*                                919/chronyd
udp6       0      0 fe80::a00:27ff:fedd:546 :::*                                903/NetworkManager
```
```
yum install nmap
```
```
[root@masterr mgb]# nmap -n -PN -sT -sU -p- 192.168.0.18
Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-06 17:42 MSK
Nmap scan report for 192.168.0.18
Host is up (0.00021s latency).
Not shown: 131068 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
10050/tcp open  zabbix-agent

Nmap done: 1 IP address (1 host up) scanned in 4.55 seconds
```
