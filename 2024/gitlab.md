# GITLAB
###
```
apt install net-tools curl -y
```
 ```
apt update
# apt install postfix
timedatectl set-timezone Europe/Moscow
apt install chrony -y
# Для автоматической синхронизации времени:
systemctl enable chrony
apt install curl openssh-server ca-certificates -y
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# Если ОС не поддерживается, то скачиваем скрипт настройки репозитория бесплатная 
wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
vi script.deb.sh
os="ubuntu"
dist="focal"
# Запускаем скрипт:
bash ./script.deb.sh
```
### Уст. пакетов GitLab:
```
apt install gitlab-ce
# выдало окно: доступно новое ядро. Список служб для перезапуска.
```
### Рекомендации после установки:
```
Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
 sudo gitlab-ctl reconfigure
```
### Конфигурируем веб-адрес
```
vi /etc/gitlab/gitlab.rb
# Меняю:
external_url 'http://gitlab.dmosk.ru'
# gitlab.dmosk.ru — в DNS или прописано в hosts.
```
# конфигурирование:
```
gitlab-ctl reconfigure
```
### Рекомендации после конфигурирования:
```
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.
```
### Вход
```
http://gitlab.dmosk.ru
# Пароль
cat /etc/gitlab/initial_root_password | grep Password:
```







  
