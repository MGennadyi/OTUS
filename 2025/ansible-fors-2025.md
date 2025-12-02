### ANSIBLE-2025
### ADD USER
```
# Заходим под root:
sudo su
adduser user
passwd user # в AstraLinux сразу запрашивает пароль!
# Debian - добавить в группу sudo:
usermod -aG sudo user
# Centos - добавить в группу sudo:
sudo usermod -aG wheel mgb
sudo usermod -aG wheel user
# Прверка:
su - mgb
user@al-1:~$
```
### Создание рабочей директории
```
# Ansible в учебной ВМ уже установлен.
mkdir -p ~/ansible/
```
```
host2 ansible_host=al-2 ansible_user=user ansible_ssh_password=11111111
```
```
ansible -i inventory all -m ping
```
### !!!



