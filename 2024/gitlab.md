# GITLAB
  ```
apt update
# apt install postfix
timedatectl set-timezone Europe/Moscow
apt install chrony
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
### Уст Gitlab
```
apt install gitlab-ce
```
### Конфигурируем веб-адрес
```
vi /etc/gitlab/gitlab.rb
# Меняю:
external_url 'http://gitlab.dmosk.ru'
# gitlab.dmosk.ru — в DNS или прописано в hosts.
```







  
