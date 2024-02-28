# Установка Apache
### Уст ubuntu
```
# Проверка, что не установлен
ifconfig
ip подставляем в браузер
sudo apt install apache2 -y
ip подставляем в браузер
```
### Уст nginx
```
sudo apt install nginx -y
systemctl status nginx
systemctl start nginx
systemctl status nginx
sudo ufw status
ip подставляем в браузер
sudo ufw allow 22
sudo ufw enable
sudo ufw status
sudo ufw allow 'Nginx Full'
sudo ufw status
vim /etc/nginx/nginx.conf
```
