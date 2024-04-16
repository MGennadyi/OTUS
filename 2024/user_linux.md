#
###
```
# Заходим под root:
sudo su
sudo adduser mgb
sudo passwd mgb
sudo adduser awx
sudo passwd awx
# Debian - добавить в группу sudo:
sudo usermod -aG sudo mgb
# Centos - добавить в группу sudo:
sudo usermod -aG wheel mgb
# Прверка:
su - mgb
```
