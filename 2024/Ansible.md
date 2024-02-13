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
=yes
```
```
# на контроллере ip=17 выполнить проброс ssh ключа командой:
ssh 192.168.0.18
```
```
sudo yum install epel-release
sudo yum update
yum -y install ansible
# Проверка версии
rpm -qa | grep ansible
ansible-2.9.27-1.el7.noarch
```

```
mkdir project
cd project/
cat > inventory
target1 ansible_host=192.168.0.18 ansible_ssh_pass=osboxes.org
# модуль=ping -i=inventory
ansible target1 -m ping -i inventory
target1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
### Клониировать ansible-target1  в ansible-target2
```
```
### SSH 1-й доступ, принимаем key. Откл. проверку key в ansible:
```
[osboxes@ansiblecontroller project]$ ssh 192.168.0.19
The authenticity of host '192.168.0.19 (192.168.0.19)' can't be established.
ECDSA key fingerprint is SHA256:1iDFrvBhm6okZBYf+uNGsEDNx4tHOOR98hRBXPGfqlY.
ECDSA key fingerprint is MD5:e7:f8:b0:07:1c:5b:4a:48:10:bc:f6:36:42:62:6c:e0.
Are you sure you want to continue connecting (yes/no)?
```
```
sudo vi /etc/ansible/ansible.cfg
/host_key - поиск
/host_key_checking = False - раскоментировать. Не рекомендуется. Лучше исп.key ssh
```
### YAML после: пробел









