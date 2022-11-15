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
# Смотрим, что установилось, парвые 2 не показывают:
dpkg-query -l
dpkg -l | grep ansible
ansible --version
```
```
vim ~/ansible/.hosts.txt

[servers]
ansible-master ansible_host=192.168.5.170 ansible_user=root ansible_ssh_pass=12345 ansible_ssh_port=22
# Не слабо:
192.168.5.170 ansible_ssh_user=mgb ansible_ssh_pass=12345
192.168.5.162 ansible_ssh_user=mgb ansible_ssh_pass=12345
192.168.5.163 ansible_ssh_user=mgb ansible_ssh_pass=12345
192.168.5.164 ansible_ssh_user=mgb ansible_ssh_pass=12345
```
```
# создаем ssh ключ (без фразы)  для связывания ВМ между собой:
sudo usermod -aG sudo mgb
sudo apt install opensss-server
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
vim hosts.txt

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
vim /home/mgb/ansible/ansible.cfg
[defaults]
host_key_checking = false
inventory = ./hosts.txt
retry_files_enabled = false
```
```
# Теперь команда будет короче. ansible.cfg примениться, находясь в одной с ним папке:
# Затребовал: apt install sshpass
cd /home/mgb/ansible
root@ansible:/home/mgb/ansible# ansible all -m ping
192.168.5.170 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
```
ansible all -m command -a "/bin/echo Hello World"
```
```
---
- hosts: servers
  tasks:
    - name: run echo command
      command: /bin/echo Hello World
```
```
###### Install_apache_playbook.yml
---
- hosts: servers
  become: yes
  tasks:
    - name: INSTALL APACHE2
      apt: name=apache2 update_cache=yes state=latest
      
    - name: ENABLE MOD_REWRITE      
      apache2_module: name=rewrite state=present
      notify:
        - RESTART APACH2
  handlers:
    - name: RESTART APACHE@
      servise: name=apache2 state=restarted    
        
        
```

























