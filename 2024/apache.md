# Установка Apache
###
```
apt install net-tools
netstat -tlpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      498/sshd: /usr/sbin
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      521/postgres
tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      1002/sshd: mgb@pts/
tcp        0      0 0.0.0.0:6432            0.0.0.0:*               LISTEN      478/pgbouncer
tcp6       0      0 :::22                   :::*                    LISTEN      498/sshd: /usr/sbin
tcp6       0      0 ::1:5432                :::*                    LISTEN      521/postgres
tcp6       0      0 ::1:6010                :::*                    LISTEN      1002/sshd: mgb@pts/
tcp6       0      0 :::6432                 :::*                    LISTEN      478/pgbouncer
```
```
root@wal-g2:/home/mgb# curl localhost
curl: (7) Failed to connect to localhost port 80: В соединении отказано
```
### Уст ubuntu
```
# Проверка, что не установлен
ifconfig
ip подставляем в браузер
sudo apt install apache2 -y
systemctl status apache2
curl localhost
ip подставляем в браузер = стартовая страница apache2
```
### Уст nginx
```
sudo apt install nginx -y
systemctl status nginx
systemctl start nginx
systemctl status nginx
sudo ufw status
ip подставляем в браузер
sudo ufw allow 22
sudo ufw enable
sudo ufw status
sudo ufw allow 'Nginx Full'
sudo ufw status
vim /etc/nginx/nginx.conf
```
