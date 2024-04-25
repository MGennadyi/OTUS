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
### 1. Уст.kubectl - RomNero -получилось
```
# Уст. доп.пакеты:
apt install curl software-properties-common ca-certificates apt-transport-https -y
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg 
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update
apt install snap
apt install kubectl # Не верно!!! command including --classic.
snap install kubectl --classic  # Верно!!!
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
```
awx@awx:~$ ls -lh /usr/local/bin
итого 137M
-rwxr-xr-x 1 root root 48M апр  6 09:31 kubectl
-rwxr-xr-x 1 root root 90M апр  6 09:48 minikube

```
### 2.1 Уст. docker
```
apt install docker.io -y
docker version  # Ответ:
Client:
 Version:           24.0.5
 API version:       1.43
 Go version:        go1.20.3
 Git commit:        24.0.5-0ubuntu1~22.04.1
 Built:             Mon Aug 21 19:50:14 2023
root@gitlab:/home/mgb# docker -v
Docker version 24.0.5, build 24.0.5-0ubuntu1~22.04.1
```
### 2.1 docker-compose
```
wget https://github.com/docker/compose/releases/download/1.29.0/docker-compose-Linux-x86_64
mv docker-compose-Linux-x86_64 /usr/bin/docker-compose
chmod +x /usr/bin/docker-compose
root@gitlab:/home/mgb# docker-compose version
docker-compose version 1.29.0, build 07737305
docker-py version: 5.0.0
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```
###
```
wget https://github.com/ansible/awx/archive/17.1.0.zip
unzip 17.1.0.zip
cd awx-17.1.0/installer/
pwgen -N 1 -s 30
iDtTi7HMNWNAmx8HTEjk6TaXeWrWMm
admin_user=admin
admin_password=mivo
secret_key=iDtTi7HMNWNAmx8HTEjk6TaXeWrWMm
ansible-playbook -i inventory install.yml
localhost                  : ok=15   changed=7    unreachable=0 # Ответ
root@gitlab:/home/mgb/awx-17.1.0/installer# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       8         5d0da3dc9764   2 years ago   231MB

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
### 4. Подготовка minikube к start, уходим от root:
###### Создание user=awx и add group docker: от RomNero
```
useradd -m -s /bin/bash awx
id awx - было
uid=1001(awx) gid=1001(awx) группы=1001(awx) - нет групп у awx -> исправляю:
# Включаем в группу docker sudo
sudo usermod -aG sudo awx
sudo usermod -aG docker awx
# Заводим пароль:
sudo passwd awx
id awx - стало
uid=1001(awx) gid=1001(awx) группы=1001(awx),27(sudo),137(docker)
```
### 5. Рестарт сервер
### 6. Старт minikube
```
su - awx # Далее все от awx
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=6g   #-долго, 4g-не запустится
#Ответ: Готово! kubectl настроен для использования кластера "minikube" и "default" пространства имён по умолчанию
```
```
awx@awx:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
 ### Прверка, какие kubernetis server уст. и используются:
```
awx@gitlab:~$ minikube kubectl -- get nodes
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  47.56 MiB / 47.56 MiB [-------------] 100.00% 4.89 MiB p/s 9.9s
NAME       STATUS   ROLES           AGE    VERSION
minikube   Ready    control-plane   3m1s   v1.28.3
```
```
awx@gitlab:/home/mgb$ minikube kubectl -- get nodes
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  49.07 MiB / 49.07 MiB [------------] 100.00% 10.91 MiB p/s 4.7s
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   3h57m   v1.30.0
```
### Проверка pod:
```
awx@gitlab:~$ kubectl get pods -A # -A отображает все компоненты
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
### Уст Dashboard: не получилось
```
apt install snap
sudo snap install helm --classic
```
```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```
### 7. Уст. awx operator: last=2.14.0 - не работает??
```
https://github.com/ansible/awx-operator #  На видео- RomNero= 0.12.0  - подставляю:
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.12.0/deploy/awx-operator.yaml
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.13.0/deploy/awx-operator.yaml  # работает
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.12.0/deploy/awx-operator.yaml  # работает
# Ответ: clusterrolebinding.rbac.authorization.k8s.io/awx-operator created
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.14.0/deploy/awx-operator.yaml - не работает
# Ответ: clusterrolebinding.rbac.authorization.k8s.io/awx-operator created
```
### Проверка запуска контейнера:
```
awx@awx:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
awx-demo-postgres-0            1/1     Running   5 (63m ago)   16d
awx-operator-dbcdf6499-jptzc   1/1     Running   7 (63m ago)   16d
```
```
kubectl describe pod awx-operator-dbcdf6499-jptzc
```
```
awx@awx:~$ kubectl describe pod awx-operator-dbcdf6499-jptzc
Name:             awx-operator-dbcdf6499-jptzc
Namespace:        default
Priority:         0
Service Account:  awx-operator
Node:             minikube/192.168.49.2
Start Time:       Sat, 06 Apr 2024 10:22:00 +0300
Labels:           name=awx-operator
                  pod-template-hash=dbcdf6499
Annotations:      <none>
Status:           Running
IP:               10.244.0.55
IPs:
  IP:           10.244.0.55
Controlled By:  ReplicaSet/awx-operator-dbcdf6499
Containers:
  awx-operator:
    Container ID:   docker://28ee58921525230eb21b484f7a380c354a492397c47208f82a719e96b7c0d397
    Image:          quay.io/ansible/awx-operator:0.13.0
    Image ID:       docker-pullable://quay.io/ansible/awx-operator@sha256:80c3297ec966ad4064c1b9477ad7331281f67d6043c3c5efe8e74cac941aaa0b
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 22 Apr 2024 15:15:39 +0300
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Tue, 16 Apr 2024 09:12:52 +0300
      Finished:     Mon, 22 Apr 2024 15:14:54 +0300
    Ready:          True
    Restart Count:  7
    Liveness:       http-get http://:6789/healthz delay=15s timeout=1s period=20s #success=1 #failure=3
    Environment:
      WATCH_NAMESPACE:
      POD_NAME:            awx-operator-dbcdf6499-jptzc (v1:metadata.name)
      OPERATOR_NAME:       awx-operator
      ANSIBLE_GATHERING:   explicit
      OPERATOR_VERSION:    0.13.0
      ANSIBLE_DEBUG_LOGS:  false
    Mounts:
      /tmp/ansible-operator/runner from runner (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tv278 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  runner:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-tv278:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
awx@awx:~$
```
```
awx@awx:~$ kubectl get pods --show-labels
NAME                           READY   STATUS    RESTARTS      AGE   LABELS
awx-demo-postgres-0            1/1     Running   5 (96m ago)   16d   app.kubernetes.io/component=database,app.kubernetes.io/instance=postgres-awx-demo,app.kubernetes.io/managed-by=awx-operator,app.kubernetes.io/name=postgres,app.kubernetes.io/part-of=awx-demo,apps.kubernetes.io/pod-index=0,controller-revision-hash=awx-demo-postgres-79f76985cb,statefulset.kubernetes.io/pod-name=awx-demo-postgres-0
awx-operator-dbcdf6499-jptzc   1/1     Running   7 (96m ago)   16d   name=awx-operator,pod-template-hash=dbcdf6499
```
### 7.1 set the current namespace for kubectl
```
kubectl config set-context --current --namespace=awx
```
```
awx@awx:~$ kubectl get pods -n awx
NAME                                               READY   STATUS      RESTARTS        AGE
awx-operator-controller-manager-6458cd4798-84v7w   2/2     Running     6 (178m ago)    15d
awx-server-migration-24.1.0-zv55f                  0/1     Completed   0               15d
awx-server-postgres-0                              1/1     Running     3 (178m ago)    15d
awx-server-postgres-15-0                           1/1     Running     3 (178m ago)    15d
awx-server-task-8676c66657-djtrl                   4/4     Running     12 (178m ago)   15d
awx-server-web-9d8c7499f-nw82s                     3/3     Running     9 (178m ago)    15d
```
### 8. Create the deployment file: конфликтует с другой установкой:
```
vim awx-demo.yml
---
apiVersion: awx.ansible.com/v_1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.com
```
### 9. Создание awx инстанса в моем кластере awx-demo.yml:
```
kubectl apply -f awx-demo.yml
# Ответ: error: resource mapping not found for name: "awx-demo" namespace: "" from "awx-demo.yml": no matches for kind "AWX" in version "awx.ansible.com/v1beta"
ensure CRDs are installed first
# Удаление:
kubectl delete -f awx-demo.yml

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
### 10.1 Просмотр сервисов:
```
awx@gitlab:~$ minikube service list
|---------------|------------------------------------|--------------|---------------------------|
|   NAMESPACE   |                NAME                | TARGET PORT  |            URL            |
|---------------|------------------------------------|--------------|---------------------------|
| default       | awx-demo-postgres                  | No node port |                           |
| default       | awx-demo-service                   | http/80      | http://192.168.49.2:31502 |
| default       | awx-operator-metrics               | No node port |                           |
| default       | kubernetes                         | No node port |                           |
| ingress-nginx | ingress-nginx-controller           | http/80      | http://192.168.49.2:32759 |
|               |                                    | https/443    | http://192.168.49.2:30324 |
| ingress-nginx | ingress-nginx-controller-admission | No node port |                           |
| kube-system   | kube-dns                           | No node port |                           |
|---------------|------------------------------------|--------------|---------------------------|
```
### 10.2 Создание доп.сервиса  LoadBalancer - не работает!
```
kubectl expose deployment awx-demo --type=LoadBalancer --port=8080 # На видео работает Romnero
# Засада
Error from server (NotFound): deployments.apps "awx-demo" not found
```
```
awx@gitlab:~$ kubectl get svc # На видео появился LoadBalancer=pending
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
awx-demo-postgres      ClusterIP   None             <none>        5432/TCP            130m
hello-minikube1        NodePort    10.100.238.34    <none>        8080:31389/TCP   3s
awx-demo-service       NodePort    10.108.249.137   <none>        80:31502/TCP        130m
awx-operator-metrics   ClusterIP   10.108.57.167    <none>        8383/TCP,8686/TCP   133m
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP             139m
# awx-demo-service - главный сервис
```
### minikube tunnel
```
awx@gitlab:~$  minikube tunnel
[sudo] пароль для awx:
Status:
        machine: minikube
        pid: 306031
        route: 10.96.0.0/12 -> 192.168.49.2
        minikube: Running
        services: []
    errors:
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```
```
awx@gitlab:~$ minikube ip
192.168.49.2
docker stats
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT   MEM %     NET I/O          BLOCK I/O     PIDS
395d8bbe9cba   minikube   21.31%    765MiB / 8GiB       9.34%     438MB / 14.8MB   0B / 4.23MB   449
kubectl port-forward service/awx-demo --address 0.0.0.0 31502:80  # 
```
```
awx@awx:~$ kubectl get pods --show-labels
NAME                           READY   STATUS    RESTARTS      AGE   LABELS
awx-demo-postgres-0            1/1     Running   5 (96m ago)   16d   app.kubernetes.io/component=database,app.kubernetes.io/instance=postgres-awx-demo,app.kubernetes.io/managed-by=awx-operator,app.kubernetes.io/name=postgres,app.kubernetes.io/part-of=awx-demo,apps.kubernetes.io/pod-index=0,controller-revision-hash=awx-demo-postgres-79f76985cb,statefulset.kubernetes.io/pod-name=awx-demo-postgres-0
awx-operator-dbcdf6499-jptzc   1/1     Running   7 (96m ago)   16d   name=awx-operator,pod-template-hash=dbcdf6499
```
### 8.0 Уст. operator через kustomization.yaml - от  NetDevops -работает!!!
```
# Подставить последнюю № версии: github.com/ansible/awx-operator/releases или https://quay.io/repository/ansible/awx-operator?tab=tags&tag=latest
vim kustomization.yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/ansible/awx-operator/config/default?ref=2.14.0
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.14.0
namespace: awx
```
```
kubectl apply -k .
# Ответ: namespase/awx created
kubectl config set-context --current --namespace=awx
```

### 8.1 Create the deployment file awx-server.yaml:
```
vim awx-server.yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-server
spec:
  service_type: nodeport
```
```
kubectl apply -k .
kustomize build . | kubectl -f -
```
### 8.2 Create the deployment file awx-demo.yaml: от NetDevops
```
vim awx-demo.yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
```
### 8.3 Добавляем awx-demo.yaml в kustomization.yaml 
```
vim kustomization.yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/ansible/awx-operator/config/default?ref=2.14.0
  - awx-demo.yaml
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.14.0
namespace: awx
```
```
kubectl apply -k .  # от NetDevops
# kustomize build . | kubectl -f -
```
```
kubectl config set-context --current --namespace=awx
# Ответ: Context "minikube" modified.
```
### От Calvin 2 сент 2022. ADD nodeport_port: 30080
```
vim awx.yml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  nodeport_port: 30080
```
### Применяем От Calvin:
```
kustomize build . | kubectl -f -
```
```
awx@gitlab:~$ kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-6458cd4798-84v7w   2/2     Running   0          3m42s
awx@awx:~$ kubectl get nodes -n awx -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   25h   v1.28.3   192.168.49.2   <none>        Ubuntu 22.04.3 LTS   6.5.0-26-generic   docker://24.0.7
awx@awx:~$ kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None             <none>        5432/TCP       25h
awx-demo-service    NodePort    10.108.249.137   <none>        80:31502/TCP   25h
==============================
INTERNAL-IP:31502 - от NetDevops
 awx                  | awx-server-service                              | http/80      | http://192.168.49.2:31148  - открылось на ВМ!!!!
```
###
```
awx@gitlab:~$ kubectl get svc
NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
awx-operator-controller-manager-metrics-service   ClusterIP   10.105.149.176   <none>        8443/TCP       81m
awx-server-postgres                               ClusterIP   None             <none>        5432/TCP       18m
awx-server-postgres-15                            ClusterIP   None             <none>        5432/TCP       32m
awx-server-service                                NodePort    10.101.157.19    <none>        80:31148/TCP   32m  - нужный сервис
```
### port-forward service
```
awx@gitlab:~$ kubectl port-forward service/awx-server-service --address 0.0.0.0 30080:80  -на видео
kubectl port-forward service/awx-server-service --address 0.0.0.0 31148:80
Error from server (NotFound): services "awx-demo-service" not found
error: timed out waiting for the condition
```
### 12. Get the Admin user password:
```
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
OaDtpysnI9asUvIsQDESxfBggr47UcBx
awx@gitlab:~$
```



### Документация kubectl -командная строка для kubernetis:
```
https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
https://gist.github.com/dmccuk/93db22e9b30d1963b8fca0de96fc82f0
https://timeweb.cloud/tutorials/docker/kak-ustanovit-docker-na-ubuntu-22-04
https://docs.ansible.com/automation-controller/4.0.0/html/administration/operator.html
https://github.com/ansible/awx-operator - изменена, ссылка на
https://ansible.readthedocs.io/projects/awx-operator/en/latest/installation/basic-install.html
https://github.com/ansible/awx-operator/blob/devel/docs/installation/basic-install.md - от NetDevops
```






















