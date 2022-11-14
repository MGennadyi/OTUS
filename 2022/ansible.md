# Postgres

```
# создаем ssh ключ (без фразы)  для связывания ВМ между собой:
sudo apt install ssh
mkdir /home/mgb/.ssh
sudo usermod -aG sudo mgb
sudo chown -R mgb /home/mgb/.ssh
su mgb
sudo -u mgb ssh-keygen -t rsa -b 4096 -q -f /home/mgb/.ssh/id_rsa -N ''
```
```
[staging_server]
etcd1 ansible_host=192.168.5.162 ansible_user=mgb ansible_pass=12345 
```


























