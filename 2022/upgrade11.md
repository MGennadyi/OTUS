##### Установка Postgres Pro 11
```
sudo apt-get install -y wget gnupg2 || sudo apt-get install -y gnupg
wget -O - http://repo.postgrespro.ru/keys/GPG-KEY-POSTGRESPRO | sudo apt-key add -
sudo apt-add-repository "deb http://repo.postgrespro.ru//pgpro-archive/pgpro-12.1.1/ubuntu bionic main"
sudo apt-add-repository "deb http://repo.postgrespro.ru//pgpro-archive/pgpro-12.1.1/ubuntu/dists/bionic/main/"

```
pg_upgrade в режиме -check с ключем -link путь:  -d исполн.файлов старой версии -D путь исп.файлов новой версии.

sudu su
echo "tmpfs /tempdb tmpfc size=1G, uid=postgres, gid=postgres 0 0" >> /etc/fstab
mount /tempdb
df -h /tempdb





