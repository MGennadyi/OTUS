#
### Уст через миникуб
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \  && chmod +x minikube
sudo cp minikube /usr/local/bin && rm minikube
root@gitlab:/home/mgb# minikube version
minikube version: v1.32.0
minikube start - выдаст ошибку с выбором хранения->
apt install docker.io -y

```
### Документация kubectl
```
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
```
### Уст. послед версию - не получилось
```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```
### Уст. kubectl с помощью стороннего пакетного менеджера SNAP - получилось
```
apt install snap
snap install kubectl --classic
kubectl 1.29.3 от Canonical✓ установлен
kubectl version
Client Version: v1.29.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
### Включение автодополнения ввода kubectl
```
echo 'source <(kubectl completion bash)' >>~/.bashrc
kubectl completion bash >/etc/bash_completion.d/kubectl
minikube start
"docker" driver should not be used with root privileges
useradd -m -s /bin/bash awx
id awx
uid=1001(awx) gid=1001(awx) группы=1001(awx) - нет групп у awx
```





