# Установка AWX 18 и выше в kubernetis по умолчанию.
```
sudo apt update && sudo apt -y upgrade
```
### Уст миникуб:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \  && chmod +x minikube
sudo cp minikube /usr/local/bin && rm minikube
root@gitlab:/home/mgb# minikube version
minikube version: v1.32.0
minikube start - выдаст ошибку, т.к. нет места хранения (контейнеров-old) подов->
apt install docker.io -y
```
### Документация kubectl -командная строка для kubernetis:
```
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
https://gist.github.com/dmccuk/93db22e9b30d1963b8fca0de96fc82f0
https://timeweb.cloud/tutorials/docker/kak-ustanovit-docker-na-ubuntu-22-04
```
### Уст. послед версию - не получилось:
```
# Уст. доп.пакеты:
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```
### Уст. kubectl с помощью стороннего пакетного менеджера SNAP - получилось:
```
sudo apt install snapd
sudo snap install core
sudo snap install kubectl --classic
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
```
### Подготовка minikube к start:
###### Создание user=awx и add group docker:
```
minikube start -> выдаст ошибку: "docker" driver should not be used with root privileges -> уходим от root:
useradd -m -s /bin/bash awx
id awx - было
uid=1001(awx) gid=1001(awx) группы=1001(awx) - нет групп у awx -> исправляю:
usermod -aG docker awx
usermod -aG sudo awx
id awx - стало
uid=1001(awx) gid=1001(awx) группы=1001(awx),27(sudo),137(docker)
su - awx # Далее все от awx
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=8g   #-долго, 4g-не запустится
#Ответ: Готово! kubectl настроен для использования кластера "minikube" и "default" пространства имён по умолчанию
```
### Прверка, какие kubernetis server уст. и используются:
```
awx@gitlab:~$ minikube kubectl -- get nodes
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  47.56 MiB / 47.56 MiB [-------------] 100.00% 4.89 MiB p/s 9.9s
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   3m1s   v1.28.3
```
### Проверка pod:
```
awx@gitlab:~$ kubectl get po -A
NAMESPACE       NAME                                        READY   STATUS      RESTARTS      AGE
ingress-nginx   ingress-nginx-admission-create-qqrfc        0/1     Completed   0             10m
ingress-nginx   ingress-nginx-admission-patch-dk88h         0/1     Completed   0             10m
ingress-nginx   ingress-nginx-controller-7c6974c4d8-7wsdl   1/1     Running     0             10m
kube-system     coredns-5dd5756b68-5pvgf                    1/1     Running     0             10m
kube-system     etcd-minikube                               1/1     Running     0             10m
kube-system     kube-apiserver-minikube                     1/1     Running     0             10m
kube-system     kube-controller-manager-minikube            1/1     Running     0             10m
kube-system     kube-proxy-n5f8r                            1/1     Running     0             10m
kube-system     kube-scheduler-minikube                     1/1     Running     0             10m
kube-system     storage-provisioner                         1/1     Running     1 (10m ago)   10m
```
### Уст оператора:
```
# Смотрим RELEASES, справа: определяем последнюю версию 2.14.0. На видео= 0.12.0  - подставляю:
https://github.com/ansible/awx-operator
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/2.14.0/deploy/awx-operator.yaml
```
###
```
awx@gitlab:~$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
awx-operator-78fb784cb7-klbq9   1/1     Running   0          7m57s
```
### Create the deployment file:
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
awx@gitlab:~$ kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
NAME                  READY   STATUS    RESTARTS   AGE
awx-demo-postgres-0   1/1     Running   0          14m
```
```
watch kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
Каждые2,0с:        kubectl get pods -l app.kubernetes.io/managed-by=awx-operator     gitlab: Wed Apr  3 18:33:06 2024
NAME                  READY   STATUS    RESTARTS   AGE
awx-demo-postgres-0   1/1     Running   0          36m
```
```
awx@gitlab:~$ minikube service list
|---------------|------------------------------------|--------------|---------------------------|
|   NAMESPACE   |                NAME                | TARGET PORT  |            URL            |
|---------------|------------------------------------|--------------|---------------------------|
| default       | awx-demo-postgres                  | No node port |                           |
| default       | awx-demo-service                   | http/80      | http://192.168.49.2:32705 |
| default       | awx-operator-metrics               | No node port |                           |
| default       | kubernetes                         | No node port |                           |
| ingress-nginx | ingress-nginx-controller           | http/80      | http://192.168.49.2:32530 |
|               |                                    | https/443    | http://192.168.49.2:30719 |
| ingress-nginx | ingress-nginx-controller-admission | No node port |                           |
| kube-system   | kube-dns                           | No node port |                           |
|---------------|------------------------------------|--------------|---------------------------|
```
### Get the Admin user password:
```
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
Eu02jpgI7k6x4QJb3kPOWaOjMIALYKOW
awx@gitlab
```
### Создание доп.сервиса  LoadBalancer
```
kubectl expose deployment awx-demo --type=LoadBalancer --port=8080
# Засада
Error from server (NotFound): deployments.apps "awx-demo" not found
```
### Просмотр сервисов:
```
awx@gitlab:~$ kubectl get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
awx-demo-postgres      ClusterIP   None             <none>        5432/TCP            53m
awx-demo-service       NodePort    10.102.86.56     <none>        80:32705/TCP        53m
awx-operator-metrics   ClusterIP   10.105.137.112   <none>        8383/TCP,8686/TCP   119m
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP             136m
```
# 1. HELM
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh
helm version # version.BuildInfo{Version:"v3.14.3",
```
# 2. Install the AWX chart
```
helm repo add awx-operator https://ansible.github.io/awx-operator/
# Ответ: "awx-operator" has been added to your repositories
helm repo update
```
### Install awx-operator via chart:
```
helm install ansible-awx-operator awx-operator/awx-operator -n awx --create-namespace
```
