# GITLAB
### UBUNTU ip=61
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
### Конфигурируем веб-адрес для входа на gitlab:
```
vi /etc/gitlab/gitlab.rb
# Меняю в строке адрес:
external_url 'http://gitlab.dmosk.ru' на http://gitlab.local
vim /etc/hosts
192.168.0.61 gitlab.local - не работает по имени, только по ip.
# Пароль для первого входа берем здесь:
cat /etc/gitlab/initial_root_password | grep Password:
```
# конфигурирование:
```
# не быстро:
gitlab-ctl reconfigure
# gitlab Reconfigured!
```
### Рекомендации после конфигурирования:
```
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.
```
### Проверка занимаемых портов:
```
netstat -tulnp | grep gitlab
```







  
