# ANSIBLE
### ADD USER
```
# Заходим под root:
sudo su
sudo adduser mgb
sudo passwd mgb
sudo adduser awx
sudo adduser student
sudo passwd awx
sudo passwd student
# Debian - добавить в группу sudo:
sudo usermod -aG sudo mgb
# Centos - добавить в группу sudo:
sudo usermod -aG wheel mgb
sudo usermod -aG wheel student
# Прверка:
su - mgb
```
### Обновление пакетов
```
sudo dnf update -y
sudo dnf upgrade -y
# В минимальном сервере yum нет:
dnf install yum -y
```
### Изменения ip-адресов и имена хостов для клонирование ВМ:
```
sudo dnf install NetworkManager-tui -y
sudo nmtui # предустановлена.
sudo shutdown -r now
```
### Првка hosts:
```
mcedit /etc/hosts
vim /etc/hosts
192.168.0.61 c9-server01
192.168.0.62 c9-server02
192.168.0.63 c9-client01
ping -c1 c9-server01 # теперь проходит по имени
for HOST in 1 2; do ping -c2 192.168.0.$HOST; done
for HOST in c9-server0{1,2}; do host $HOST; done
```
### Хост c9-server01=ip adress 192.168.0.61
```
dnf install dnsutils -y
[root@c9-client01 ~]# nslookup c9-server01
Name:   c9-server01
Address: 192.168.0.61
```
### Список доступных репозиториев YUM
```
sudo yum repolist
base                                                                                                    RedOS - Base
updates                                                                                                 RedOS - Updates
```
### Загрузите конфигурационный файл репозитория Ansible
```
sudo wget -P /etc/yum.repos.d/ http://rpm.ll-100.local/repo/ansible29.repo # -По материалам занятия
sudo wget -P /etc/yum.repos.d/ https://dl.rockylinux.org/pub/rocky/8/Devel/aarch64/os/Packages/c/centos-release-ansible-29-1-2.el8.noarch.rpm
sudo yum update -y
sudo yum upgrade -y
root@c9-client01 ~]# sudo yum install ansible.noarch -y
Последняя проверка окончания срока действия метаданных: 3:51:29 назад, Ср 08 янв 2025 10:46:07.
Пакет ansible-9.2.0-1.red80.noarch уже установлен.
```
### Redos Уст. ansible с нужными пакетами на client01: на Redos не ставится:
```
sudo -i # 2 команды уст ansible:
dnf install epel-release -y
dnf install ansible.noarch -y
# проверка:
rpm -q ansible
ansible-6.7.0-1.el7.noarch
ansible --version
===============
sudo dnf install pip
sudo dnf install pip3
dnf search ansible
pip3 install ansible
dnf install python3.12-pip
# ansible из пакетов
```
#### Проверка версии ansible
```
rpm -qa | grep ansible
# Ответ:
ansible-core-2.16.3-1.red80.noarch
ansible-9.2.0-1.red80.noarch
[root@c9-client01 ~]# ansible --version
ansible [core 2.16.3]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.11/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /bin/ansible
  python version = 3.11.10 (main, Sep  9 2024, 00:00:00) [GCC 12.4.1 20240730 (RED SOFT 12.4.0-1)] (/usr/bin/python3)
  jinja version = 3.1.3
  libyaml = True
```
```
root@c9-client01 ~]# dnf info ansible-core
Последняя проверка окончания срока действия метаданных: 4:25:43 назад, Ср 08 янв 2025 10:46:07.
Установленные пакеты
Имя          : ansible-core
Версия       : 2.16.3
Выпуск       : 1.red80
```
#### .1 Проверка, что в домашнем каталоге student ssh нет :
```
sudo -i -u student
ls -d ~/.ssh
ls: невозможно получить доступ к '/home/student/.ssh': No such file or directory
```
#### .2 Генарация ключа (без passphrase):
```
ssh-keygen
sudo dnf install sshpass
```
#### .3 Проверка, что ssh в домашнем каталоге есть:
```
ls -d ~/.ssh
```
#### .4 Ключи появились
```
ls ~/.ssh/
# Ответ:
id_ed25519  id_ed25519.pub
```
#### .5 Подкл. под student к c9-server01:
```
ssh student@c9-server01
ssh student@c9-server02
yes + ввод пароля !!!!
.6 exit
```
#### .7 Убедитесь, что отпечаток системы c9-server01 добавлен в файл known_hosts:
```
cat ~/.ssh/known_hosts
c9-server01 ecdsa-sha2-nistp256
c9-server02 ssh-ed25519 
```
#### .8 Выполнить команду hostname на c9-server01 и c9-server02, подключившись к ним по протоколу SSH от имени пользователя student (пароль: 12345).
```
for HOST in c9-server01 c9-server02; do ssh student@$HOST hostname; done
student@c9-server01's password:
c9-server01
student@c9-server02's password:
c9-server02
```
#### .9 Убедитесь, что отпечатки систем c9-server01 и c9-server02 добавлены в файл known_hosts
```
tail -n2 ~/.ssh/known_hosts
```
#### 10. Скопировать открытый ключ RSA в профиль пользователей student (пароль: 12345) и root (пароль: 12345) на c9-server01, c9-server02:
```
for HOST in {student,root}@{c9-server0{1,2}}; \ > do ssh-copy-id $HOST; done -не верно -bash: синтаксическая ошибка рядом с неожиданным маркером «\ »
# Для одного пользователя student:
for HOST in student@c9-server0{1,2}; do ssh-copy-id $HOST; done
# Для нескольких пользователей сразу: student, root:
for HOST in {student,root}@c9-server0{1,2}; do ssh-copy-id $HOST; done
```
### 11. Проверка подключения к c9-server01 c9-server02 от student и root с ключа RSA, т.е. без пароля:
```
for HOST in {student,root}@c9-server0{1,2}; do ssh $HOST 'echo "$(whoami) on $(uname -n)"'; done
# Ответ:
student on c9-server01
student on c9-server02
root on c9-server01
root on c9-server02
```
### 3.1 Убедитесь, что пользователь student состоит в группе wheel на системах c8-server01, c8-server02 и c7-server01.
```
for HOST in c9-server01 c9-server02; do ssh student@$HOST hostname; done
```
### Убедитесь что sudo требует ввода пароля группы wheel:
```
for HOST in root@{c9-server0{1,2},c9-client01}; do ssh $HOST 'echo "$(hostname):"; grep ^.wheel /etc/sudoers'; done
```
### DEBIAN 8Gb-ssd
```
apt update
mgb@ansible:~$ python3 --version
Python 3.11.10
apt install git python3-pip docker docker-compose   - Romnero
apt install python3-pip  - Romnero
pip3 install ansible     - Romnero
ansible [core 2.15.9] - не последняя, при этом config file = None
ps -aux | grep ansible
apt install ansible  - dmosk.ru ansible [core 2.15.9]
# https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian
 UBUNTU_CODENAME=jammy  #Debian 12=jammy 11=focal 10=bionic
UBUNTU_CODENAME=focal
$ wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
$ echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
$ sudo apt update && sudo apt upgrade
apt install ansible
Уже установлен пакет ansible самой новой версии (5.10.0-1ppa~focal)
apt --fix-broken install
```
### Конфиг по умолчанию ??
```
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
dnf install pip -y
yum install dnf # Уже был.
sudo dnf install git-all -y
Complete!
sudo dnf install ansible.noarch -y
# Ответ: Установлен:  ansible-9.2.0-1.red80.noarch        ansible-core-2.16.3-1.red80.noarch
========================
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
### Конфиг
```
vi ansible.cfg
[default]
inventory=./hosts
```
### Просмотр конфига:
```
ansible-config view -v # Параметр -v покажет путь к конфигу
```






