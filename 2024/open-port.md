###
###
```

```
### netstat
```
yum install net-tools
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
### nmap
```
yum install nmap
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
### lsof
```
[root@masterr mgb]# lsof -i
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
NetworkMa  903   root   24u  IPv6  23761      0t0  UDP masterr:dhcpv6-client
NetworkMa  903   root   29u  IPv4  23826      0t0  UDP masterr:bootpc->_gateway:bootps
chronyd    919 chrony    5u  IPv4  19915      0t0  UDP localhost:323
chronyd    919 chrony    6u  IPv6  19916      0t0  UDP localhost:323
sshd       969   root    3u  IPv4  22334      0t0  TCP *:ssh (LISTEN)
sshd       969   root    4u  IPv6  22340      0t0  TCP *:ssh (LISTEN)
zabbix_ag  977 zabbix    8u  IPv6  20363      0t0  TCP *:zabbix-agent (LISTEN)
zabbix_ag  977 zabbix   10u  IPv4  59048      0t0  TCP masterr:38541->zabbix:zabbix-trapper (SYN_SENT)
master    1247   root   13u  IPv4  23654      0t0  TCP localhost:smtp (LISTEN)
sshd      1331   root    4u  IPv4  23170      0t0  TCP masterr:ssh->192.168.0.12:50994 (ESTABLISHED)
sshd      1335   root    4u  IPv4  23218      0t0  TCP masterr:ssh->192.168.0.12:50998 (ESTABLISHED)
sshd      1337    mgb    4u  IPv4  23170      0t0  TCP masterr:ssh->192.168.0.12:50994 (ESTABLISHED)
sshd      1348    mgb    4u  IPv4  23218      0t0  TCP masterr:ssh->192.168.0.12:50998 (ESTABLISHED)
```
### ss
```
[root@masterr mgb]# ss -lntu
Netid            State             Recv-Q             Send-Q                                             Local Address:Port                          Peer Address:Port            Process
udp              UNCONN            0                  0                                                      127.0.0.1:323                                0.0.0.0:*
udp              UNCONN            0                  0                                                          [::1]:323                                   [::]:*
udp              UNCONN            0                  0                              [fe80::a00:27ff:fedd:697a]%enp0s3:546                                   [::]:*
tcp              LISTEN            0                  128                                                      0.0.0.0:22                                 0.0.0.0:*
tcp              LISTEN            0                  100                                                    127.0.0.1:25                                 0.0.0.0:*
tcp              LISTEN            0                  4096                                                           *:10050                                    *:*
tcp              LISTEN            0                  128                                                         [::]:22                                    [::]:*
```
