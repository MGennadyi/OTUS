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
# Смотрим, что установилось, первые 2 не показывают:
dpkg-query -l
dpkg -l | grep ansible
ansible --version
```
```
vim ~/ansible/.hosts.txt

[servers]
ansible-master ansible_host=192.168.5.170 ansible_user=root ansible_ssh_pass=12345 ansible_ssh_port=22
# Не слабо:
etcd1 ansible_ssh_host=192.168.5.162 ansible_ssh_user=mgb ansible_ssh_pass=12345
etcd2 ansible_ssh_host=192.168.5.163 ansible_ssh_user=mgb ansible_ssh_pass=12345
etcd3 ansible_ssh_host=192.168.5.164 ansible_ssh_user=mgb ansible_ssh_pass=12345
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
# С ansible раскладываем ключ по другим ВМ из под mgb:
ssh-copy-id etcd1
ssh-copy-id etcd2
ssh-copy-id etcd3
ssh-copy-id -i ~/.ssh/id_rsa.pub mgb@etcd1
ssh-copy-id -i ~/.ssh/id_rsa.pub mgb@etcd2
ssh-copy-id -i ~/.ssh/id_rsa.pub mgb@etcd3
yes
passwd
# В результате на удаленной машине появится autirized_keys
sudo systemctl restart sshd
```
```
vim hosts.txt

[staging_server]
etcd2 ansible_host=192.168.5.163 ansible_user=mgb ansible_privat_key_file=/home/mgb/.ssh/autirized_keys
etcd2 ansible_host=192.168.5.163 ansible_user=mgb ansible_privat_key_file=/home/mgb/.ssh/autirized_keys
```
```
mgb@etcd1:~/ansible$ ansible -i hosts.txt all -m ping
mgb@etcd1:~/ansible$ ansible -i hosts.txt all -m ping
etcd2 | SUCCESS => {mc
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
##### Отконфигурируем ansible /home/mgb/ansible/ansible.cfg:
```
# В папке с ansible.cfg будет использован не глобальный, а локальный cfg
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

###### ansible-playbook -v install_apache_playbook.yml -kK
```
# ansible-playbook -v install_apache_playbook.yml --extra-vars "ansible_sudo_pass=12345"
# -k, --ask-pass: ask for connection password
# -K, --ask-become-pass: ask for privilege escalation password
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
        
    - name: APACHE2 LISTEN ON PORT 8081
      lineinfile: dest=/etc/apache2/ports.conf regexp="^Listen 80" Line="listen 8081"
        
        
  handlers:
    - name: RESTART APACHE2
      servise: name=apache2 state=restarted    
```
###### ansible-playbook -v install_apache.yml
```
---
- hosts: servers
  tasks:
    - name: Installs apache web server
      apt: pkg=apache2 state=installed update_cache=true
```
###### Просмотр свободной памяти
```
# Вывод всех параметров:
ansible -i hosts.txt etcd1 -m setup
# Вывод доступной памяти ОЗУ:
ansible -m shell -a 'free -m' etcd1
ansible -i hosts.txt etcd1 -a 'filter=ansible_memtotal_mb' -m setup
```
###### ansible-playbook
```
[servers]
etcd1 ansible_host=192.168.5.162 ansible_user=mgb ansible_ssh_private_key_file=/home/mgb/.ssh/id_rsa
etcd2 ansible_host=192.168.5.163 ansible_user=mgb ansible_ssh_private_key_file=/home/mgb/.ssh/id_rsa
etcd3 ansible_host=192.168.5.164 ansible_user=mgb ansible_ssh_private_key_file=/home/mgb/.ssh/id_rsa
```
###### ansible-playbook -v install_postgres.yml
```
---
- name: Populate PostgreSQL apt template on target host
  template:
    src: templates/pgdg.list.j2
    dest: /etc/apt/sources.list.d/pgdg.list
    owner: root
    group: root
    mode: '0644'
```
# AWX
```
apt install git python3-pip docker docker-compose -y

pip3 install ansible
mkdir /downloads
cd /downloads/
# Клонируем репозиторий:
git clone https://github.com/ansible/awx
cd /downloads/
ansible-playbook.yml -i inventory

```























