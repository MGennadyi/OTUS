# Установка PROXMOX v.7
##### 
```
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bullseye pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
wget https://enterprise.proxmox.com/debian/proxmox-release-bullseye.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bullseye.gpg
apt update && apt full-upgrade
```
##### Install the Proxmox VE Kernel
```
apt install proxmox-ve postfix open-iscsi
systemctl reboot
```
##### Install the Proxmox VE packages

```
apt install proxmox-ve postfix open-iscsi

```














