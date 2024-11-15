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
```
### список доступных репозиториев YUM
```
sudo yum repolist
идентификатор репозитория                                                                       имя репозитория
base                                                                                            RedOS - Base
kernels                                                                                         Kernels updates for RED OS 7.3
updates                                                                                         RedOS - Updates
```
### Изменения ip-адресов и имена хостов для клонирование ВМ
```
sudo nmtui # предустановлена.
sudo shutdown -r now
```
```
sudo -i
dnf install epel-release -y
dnf search ansible
ansible.noarch : Curated set of Ansible collections included in addition to ansible-core
```
### Уст. ansible с нужными пакетами:
```
dnf install ansible.noarch -y
# проверка:
rpm -q ansible
ansible-6.7.0-1.el7.noarch
ansible --version
```
### Првка hosts
```
vim /etc/hosts
192.168.0.61 c9-server01
192.168.0.62 c9-server02
192.168.0.63 c9-client01
ping -c1 c9-server01 # теперь проходит по имени
for HOST in 1 2; do ping -c2 192.168.0.$HOST; done
for HOST in c9-server0{1,2}; do host $HOST; done
```
#### .1 Проверка, что ssh в домашнем каталоге нет:
```
ls -d ~/.ssh
ls: невозможно получить доступ к '/home/student/.ssh': No such file or directory
```
#### .2 Генарация ключа без passphrase:
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
id_rsa  id_rsa.pub
```
#### .5 Подкл. под student к c9-server01:
```
ssh student@c9-server01
ssh student@c9-server02
yes
.6 exit
```
#### .7 Убедитесь, что отпечаток системы c9-server01 добавлен в файл known_hosts:
```
cat ~/.ssh/known_hosts
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
#### Скопировать открытый ключ RSA в профиль пользователей student (пароль: Pa$$w0rd) и root (пароль: Pa$$w0rd) на системы c9-server01, c9-server02:
```
for HOST in {student,root}@{c9-server0{1,2}}; \ > do ssh-copy-id $HOST; done -не верно -bash: синтаксическая ошибка рядом с неожиданным маркером «\ »
for HOST in student@c9-server0{1,2}; do ssh-copy-id $HOST; done
```
### DEBIAN 8Gb-ssd
```
apt update
mgb@ansible:~$ python3 --version
Python 3.9.2
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
$ sudo apt update && sudo apt install ansible
При обработке следующих пакетов произошли ошибки:
 /tmp/apt-dpkg-install-HjDFe8/1-ansible-core_2.12.10-1ppa~focal_all.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)
apt install ansible
Уже установлен пакет ansible самой новой версии (5.10.0-1ppa~focal)
apt --fix-broken install
```
#### Проверка версии
```
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







