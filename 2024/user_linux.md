#
### add user
```
# Заходим под root:
sudo su
sudo adduser mgb
sudo passwd mgb
sudo adduser awx
sudo adduser student
sudo passwd awx
sudo passwd student
# Debian - добавить в группу sudo:
sudo usermod -aG sudo mgb
# Centos - добавить в группу sudo:
sudo usermod -aG wheel mgb
sudo usermod -aG wheel student
# Прверка:
su - mgb
```
### nmtui - правка ip
```


```
