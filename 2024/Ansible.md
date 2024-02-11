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
sudo yum update
yum -y install ansible
# Проверка версии
rpm -qa | grep ansible
```

```
mkdir project
cd project/
cat > inventory
target1 ansible_host=192.168.0.18 ansible_ssh_pass=osboxes.org
# модуль=ping -i=inventory
ansible target1 -m ping -i inventory
```











