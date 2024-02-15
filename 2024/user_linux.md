#
###
```
sudo adduser mgb
sudo passwd mgb
# Debian - добавить в группу sudo:
sudo usermod -aG sudo mgb
# Centos - добавить в группу sudo:
sudo usermod -aG wheel mgb
# Прверка:
su - mgb
```
