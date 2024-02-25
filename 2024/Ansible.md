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
[osboxes@ansiblecontroller ansible-demo1]$ ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/osboxes/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Nov 14 2023, 16:14:06) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```
### Конфиг
```
# По умолчанию:
/etc/ansible/ansible.cfg
```
### Рабочие папки:
```
mkdir -p ~/"ansible-apache"
~/ansible.cfg - в приоритете
```
### Рабочие папки:
```
mkdir project
cd ansible-apache/
cd project/
cat > inventory
target1 ansible_host=192.168.0.18 ansible_ssh_pass=osboxes.org
target1 ansible_host=192.168.0.18 ansible_ssh_pass=11111111
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
### Подкл: 
```
.ssh/config/ -удалить все наследуемые разрешения -ок
дополниетльно/добавить/принципал.дополнительно/найти/mgb/ок/полный доступ-галочки/ок
# Connect to host/add new/ ввод команд ssh:
ssh mgb@192.168.0.17 - выбрать место хранен.конфигур (верхн.строчку)
ssh osboxes@192.168.0.17
в правом нижнем: подключиться/продолжить/ввод пароля
```
```
sudo vi /etc/ansible/ansible.cfg
/host_key - поиск
/host_key_checking = False - раскоментировать. Не рекомендуется. Лучше исп.key ssh
```
### YAML после: пробел
###
```
mkdir ansible-demo1
cd ansible-demo1/
в atom ansible-demo1/
new folder exer1
```
```
cp ../project/inventory .
cat inventory
[linux]
target1 ansible_host=192.168.0.18 ansible_ssh_pass=osboxes.org
target2 ansible_host=192.168.0.19 ansible_ssh_pass=osboxes.org
```
### CentOS
```
yum install dnf
sudo dnf install git-all
Complete!
https://about.gitlab.com/install/#centos-7
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ce
external_url 'http://gitlab.dmosk.ru'
vi /etc/gitlab/gitlab.rb
gitlab-ctl reconfigure
gitlab Reconfigured!
vim /etc/gitlab/initial_root_password
192.168.0.17 root/4QXYKrEq3vKoPb7p9ujyWAgL8z08Loo+4pkB288lmJo=  - ansible-kontroller
```
### Настройка Gitlab
```
Admin area/users: mgb/q12345678
pass должен соответствовать уровню сложности
```
### Windows
```
https://git-scm.com/download/win
https://git-scm.com/download/win
64-bit Git for Windows Setup.
```







