# Установка AWX 18 и выше в kubernetis по умолчанию. 
### 0.repos for Kubernetes and Cubectl - получилось:
```
sudo apt update && sudo apt -y upgrade
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo cp kubectl /usr/local/bin && rm kubectl
kubectl version
# Ответ:
Client Version: v1.29.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
The connection to the server localhost:8080 was refused - did you specify the right host or port?
==============================
sudo cp minikube /usr/local/bin && rm minikube
minikube version: v1.32.0
```
### Уст.kubectl - не получилось
```
# Уст. доп.пакеты:
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl # E: Невозможно найти пакет kubectl
```
### 1. Уст. kubectl с помощью стороннего пакетного менеджера SNAP - получилось:
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
### 2. Уст. docker
```
apt install docker.io -y
docker version  # Ответ:
Client:
 Version:           24.0.5
 API version:       1.43
 Go version:        go1.20.3
 Git commit:        24.0.5-0ubuntu1~22.04.1
 Built:             Mon Aug 21 19:50:14 2023
```
### 3. Уст миникуб:
```
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.21.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
# Предпочтительнее:
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
# Раздельные команды:
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
cp minikube /usr/local/bin
rm minikube
minikube version  # v1.32.0
```
### Включение автодополнения ввода kubectl
```
echo 'source <(kubectl completion bash)' >>~/.bashrc
kubectl completion bash >/etc/bash_completion.d/kubectl
```
### 4. Подготовка minikube к start:
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
```
### 5. Рестарт сервер
### 6. Старт minikube
```
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
### 7. Уст. awx operator
```
https://github.com/ansible/awx-operator # определяем последнюю версию 2.14.0. На видео= 0.12.0  - подставляю:
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.13.0/deploy/awx-operator.yaml  # работает
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.12.0/deploy/awx-operator.yaml
# Ответ: clusterrolebinding.rbac.authorization.k8s.io/awx-operator created
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.14.0/deploy/awx-operator.yaml - не работает
```
### Проверка запуска контейнера:
```
awx@gitlab:~$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
awx-operator-78fb784cb7-l8xf5   1/1     Running   0          3m45s
```
### 8. Create the deployment file:
```
vim awx-demo.yml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.example.com
```
### 9. Применение awx-demo.yml
```
kubectl apply -f awx-demo.yml
awx.awx.ansible.com/awx-demo created
awx@gitlab:~$ kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
# Сразу ответ: No resources found in default namespace. Спустя 2 мин будет:
NAME                  READY   STATUS    RESTARTS   AGE
awx-demo-postgres-0   1/1     Running   0          2m3s
```
### Ждем пару минут
```
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
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
| default       | awx-demo-service                   | http/80      | http://192.168.49.2:30505 |
| default       | awx-operator-metrics               | No node port |                           |
| default       | kubernetes                         | No node port |                           |
| ingress-nginx | ingress-nginx-controller           | http/80      | http://192.168.49.2:31595 |
|               |                                    | https/443    | http://192.168.49.2:30798 |
| ingress-nginx | ingress-nginx-controller-admission | No node port |                           |
| kube-system   | kube-dns                           | No node port |                           |
|---------------|------------------------------------|--------------|---------------------------|
```
### 10. Get the Admin user password:
```
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
KV6EPV4rGjFgusdzyrj8BM3nAtaXMY4z
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
### Документация kubectl -командная строка для kubernetis:
```
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
https://gist.github.com/dmccuk/93db22e9b30d1963b8fca0de96fc82f0
https://timeweb.cloud/tutorials/docker/kak-ustanovit-docker-na-ubuntu-22-04
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
```






















