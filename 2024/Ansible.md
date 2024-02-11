### osboxes.org/centos
### pass
```
Username: osboxes
Password: osboxes.org
Root Account Password: osboxes.org
```
### ip
```
ifconfig
192.168.122.1
# Не моя сеть!!!
```
### Активация сет.адаптер в автозагрузке
```
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```
yum install epel-release
```

```
mkdir project
cd project/
cat > inventory
target1 ansible_host=192.168.018 ansible_ssh_pass=osboxes.org
```











