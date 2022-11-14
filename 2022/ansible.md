# Postgres

```
# создаем ssh ключ (без фразы)  для связывания ВМ между собой:
sudo apt install ssh
mkdir /home/mgb/.ssh
sudo usermod -aG sudo mgb
sudo chown -R mgb /home/mgb/.ssh
su mgb
sudo -u mgb ssh-keygen -t rsa -b 4096 -q -f /home/mgb/.ssh/id_rsa -N ''

# Раскоментировать 
sudo vim /etc/ssh/sshd_config
PasswordAuthentication yes
sudo systemctl restart sshd
# С etcd1 раскладываем ключ по другим ВМ из под gpadmin:
ssh-copy-id etcd2
ssh-copy-id etcd3
```
```
[staging_server]
etcd1 ansible_host=192.168.5.162 ansible_user=mgb ansible_privat_key_file=/home/mgb/.ssh/autir
```


























