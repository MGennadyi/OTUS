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
Нет отслеживаемых файлов. Создаю файл просмотра show и добавляю :
#!/bin/bash
cat ./$1/*
chmode +x show
git add show
git commit -m 'first commit' - выдаст ошибку, т.к. Git не настроен
# Конфигурирую: определить владельца
git config --global user.email "ser_stiven@mail.ru"
git config --global user.name "mgb"
git log - показывает список коммитов
git log -p # Показывает, что поменялось
q # - выход из режима просмотра
git add 01 # - добавляет директорию в отслеживание.
git commit -m 'first commit'   # -первый коммит добавляет в репозиторий скрипт show
git commit -m 'recipe 01'      # -второй коммит положили первый рецепт -??? у меня ничего, у ведущего есть -??
git commit -am 'recipe 01 fix' # -добавление всех файлов с изменениями и отслеживаемые
# Перевод директорию /etc под source controle для отслеживание исходного кода конфигов серверов: 
sudo -i - стать суперюзером
cd /etc
git init
git add * # закомитить все файлы в текущей директории
git commit -m 'init'
```
### Просмотр рецептов
```
cd Projects/git
./show 01
```
### Клонироание репозитория ansible из github.com
```
cd .. # вхожу в папку Projects и клонирую
github.com/ansible/ansible.git - зеленая кнопка Code - copy url
git clone https://github.com/ansible/ansible.git # руками
cd ~/Projects/git
git log # git чекаутнет удаленную репу.
add folder to workspace -> ansible -> open integrated termonal
```
### Манипуляция историей, точки возврата через хеш коммита
```
git log # будет показан хеш
git checkout ffdcc6c # первые 6 символов хеша, возврат директории в состояние этого коммита, т.е. в прошлое
git checkout master # возвращаюсь в настоящее
бывает так, что я прыгнул в истории на коммит, но для воспроизведения бага мне нужно поправить код и сделать коммит
git commit -m 'my fix'
git checkout master
git diff
показывает разницу между последним коммитом и еще не добавленными в stage файлами

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



