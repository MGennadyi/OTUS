# Установка и настройка Docker
```
sudo apt install docker.io -y
# Проверка версии
docker -v
# Ответ:
Docker version 20.10.5+dfsg1, build 55c4c88
```
```
# Docker определит, что нет лок.образа с таким именем и загруз. его с Docker Hub,запустит образ в виде контейнера. После вывода приветствия и заверш.работу.
sudo docker run hello-world
```
```
# Просмотр загруженных образов:
sudo docker images
#  Искать образы для загрузки:
sudo docker search debian
# Загр. образ без запуска:
sudo docker pull debian

sudo docker run -it -p 8080:80 yeasy/simple-web
 CTRL + C
```

```
vim user-input

#!/bin/bash

echo -n "enter Your Name:"
read userName
echo "Your name is $userName"

bash user-input
./user-input
# Отказ доступа: дадим права на запуск:
chmod +x user-input
```

```
sudo apt-cache search python3-pip
sudo apt install python3-pip
pip3 install flask


```















