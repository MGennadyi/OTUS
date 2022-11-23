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
# Уст.framework:
pip3 install flask
```
```
vim sample_app.py

from flask import Flask
from flask import request
from flask import render_template

sample = Flask(_name_)

@sample.route("/")
def main():
    return render_template("index.html")
    return "You are calling me from " + request.remote_addr + "\n"

if __name__== "__main__":
 sample.run(host = "0.0.0.0", port=8080)
```
```
python3 sample_app.py
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8080
 * Running on http://192.168.5.163:8080
Press CTRL+C to quit
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
```
###### vim sample_app.sh
```
mkdir temdir
mkdir temdir/templates
mkdir temdir/static

cp sample_app.py tempdir/.
cp -r templates/* tempdir/templates/.
cp -r static /* tempdir/static/.

echo "FROM python" > temdir/Dockerfile
echo "RUN pip install flask >> temdir/Dockerfile

echo "COPY ./static /home/myapp/static" >> tempdir/Dockerfile
echo "COPY ./template /home/myapp/templates/" >> tempdir/Dockerfile
echo "COPY sample_app.py /home/myapp/" >> tempdir/Dockerfile

echo "EXPOSE 8080" >> tempdir/Dockerfile

echo "CMD python3 /home/myapp/sample_app.py" >> tempdir/Dockerfile

cd tempdir
docker build-t sampleapp .

docker run -t -d -p 8080:8080 --name samplerunning sampleapp

docker ps -a
```
# ----------------------------------------
```
# Извлекаем последний публичный образ postgres  Docker Hub:
docker pull postgres
# Или определенную версию:
docker pull postgres:14.2
# запуск Docker-конт. исп.образ postgres:latest: переменные POSTGRES_USER, POSTGRES_PASSWORD, -p 5432:5432 для уст.user/pass/port: 
docker run -itd -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=12345 -p 5432:5432 -v /data:/var/lib/postgresql/data --name postgresql postgres

PGPASSWORD=12345 psql -U postgres
PGPASSWORD=12345 psql -U postgres -c '\l'
```











