# Postgres
```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
vim /etc/apt/sources.list
deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main
sudo apt update
# Лучше устанавливать Ansible НЕ из репозитория Debian, а использовать Pip или Pip3:
apt install python3 python3-pip
pip install ansible
```
```
Installing collected packages: pycparser, pyparsing, MarkupSafe, cffi, resolvelib, PyYAML, packaging, jinja2, cryptography, ansible-core, ansible
Successfully installed MarkupSafe-2.1.1 PyYAML-6.0 ansible-6.6.0 ansible-core-2.13.6 cffi-1.15.1 cryptography-38.0.3 jinja2-3.1.2 packaging-21.3 pycparser-2.21 pyparsing-3.0.9 resolvelib-0.8.1
root@ansible:/home/mgb#
```
```
# Смотрим, что установилось:
dpkg-query -l
dpkg -l | grep ansible
ansible --version
```

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
##### Отконфигурируем ansible:
```
vim ansible.cfg
[defaults]
host_key_checking = false
inventory = ./hosts.txt
```
```
# Теперь команда будет короче:
ansible all -m ping
```


























