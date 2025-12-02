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
### Первый учебный инвентори:
```
host2 ansible_host=al-2 ansible_user=user ansible_ssh_password=11111111
```
#### Запуск инвентор
```
user@al-1:~$ ansible -i inventory all -m ping
host2 | FAILED! => {
    "msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"
}
sudo apt install sshpass
```
#### Запустился, но есть лишние сообщения. Устраняю:
```
vim ~/ansible.cfg
[defaults]
interpreter_python=auto_silent
```
### !!!


### Создание рабочей директории
```
# Ansible в учебной ВМ уже установлен.
ansible --version  [core 2.18.3]
mkdir -p ~/ansible/
```



