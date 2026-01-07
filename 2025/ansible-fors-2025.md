### ANSIBLE-2025 AstraLinux VM: al-1 al-2. По видео от Лосева
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
### Установка VSCode через скачивание:
```
https://code.visualstudio.com/Download
cd /загрузки
sudo apt install ./code......
add folder to workspace
explorer
```
### Установкка Git
```
sudo apt install git -y
вставить образ/удалить образ
по git Open integrated terminal
git init - инициализация нового репозитория /home/user/project/git/.git
```
# Создание структуры файлов:
```
ctrl+s по всем файлам
git status
Нет отслеживаемых файлов. Создаю файл просмотра:
#!/bin/bash
cat ./$1/*
chmode +x chow
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
### Добавление переменных:
```
[server]
host2 ansible_host=al-2

[all:vars]
ansible_user=user
ansible_ssh_password=11111111
ansible_ssh_args=' -o StrictHostKeyChecking=no'
```
### Создание рабочей директории
```
# Ansible в учебной ВМ уже установлен.
ansible --version  [core 2.18.3]
mkdir -p ~/ansible/
```



