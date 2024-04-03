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
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
https://gist.github.com/dmccuk/93db22e9b30d1963b8fca0de96fc82f0
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
id awx - было
uid=1001(awx) gid=1001(awx) группы=1001(awx) - нет групп у awx -> исправляю:
usermod -aG docker awx
usermod -aG sudo awx
id awx - стало
uid=1001(awx) gid=1001(awx) группы=1001(awx),27(sudo),137(docker)
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=6g   -долго
awx@gitlab:~$ kubectl get po -A
NAMESPACE       NAME                                        READY   STATUS      RESTARTS      AGE
ingress-nginx   ingress-nginx-admission-create-xnd45        0/1     Completed   0             2m6s
ingress-nginx   ingress-nginx-admission-patch-v7hhd         0/1     Completed   1             2m6s
ingress-nginx   ingress-nginx-controller-7c6974c4d8-nslmq   1/1     Running     0             2m6s
kube-system     coredns-5dd5756b68-vv4dt                    1/1     Running     0             2m6s
kube-system     etcd-minikube                               1/1     Running     0             2m18s
kube-system     kube-apiserver-minikube                     1/1     Running     0             2m18s
kube-system     kube-controller-manager-minikube            1/1     Running     0             2m18s
kube-system     kube-proxy-65r4v                            1/1     Running     0             2m6s
kube-system     kube-scheduler-minikube                     1/1     Running     0             2m18s
kube-system     storage-provisioner                         1/1     Running     1 (94s ago)   2m15s
```
###
```
https://github.com/ansible/awx-operator
# определяем последнюю версию 2.14.0. На видео= 0.12.0  - подставляю:
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.12.0/deploy/awx-operator.yaml
```
###
```
awx@gitlab:~$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
awx-operator-78fb784cb7-klbq9   1/1     Running   0          7m57s
```
###
```
vim awx-demo.yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.example.com
```
```
kubectl apply -f awx-demo.yaml
awx.awx.ansible.com/awx-demo created
```




