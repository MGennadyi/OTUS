# Postgres

```
# создаем ssh ключ (без фразы)  для связывания ВМ между собой:
sudo usermod -aG sudo mgb
sudo apt install ssh
su mgb
mkdir /home/mgb/.ssh
# Если не из-под mgb, то
sudo chown -R mgb /home/mgb/.ssh
sudo -u mgb ssh-keygen -t rsa -b 4096 -q -f /home/mgb/.ssh/id_rsa -N ''

# Раскоментировать на всех нодах:
sudo vim /etc/ssh/sshd_config
PasswordAuthentication yes
sudo systemctl restart sshd
# С etcd1 раскладываем ключ по другим ВМ из под mgb:
ssh-copy-id etcd2
yes
passwd
ssh-copy-id etcd3
yes
passwd
# В результате появится autirized_keys
```
```
mkdir hosts.txt

[staging_server]
etcd2 ansible_host=192.168.5.163 ansible_user=mgb ansible_privat_key_file=/home/mgb/.ssh/autirized_keys
etcd2 ansible_host=192.168.5.163 ansible_user=mgb ansible_privat_key_file=/home/mgb/.ssh/autirized_keys
```
```
mgb@etcd1:~/ansible$ ansible -i hosts.txt all -m ping
etcd2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
etcd3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```




























