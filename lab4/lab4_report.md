University: [ITMO University](https://itmo.ru/ru/)   
Faculty: [FICT](https://fict.itmo.ru)   
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)   
Year: 2024/2025   
Group: K4111с   
Author: Shabashov Denis Igorevich   
Lab: Laboratory Work No. 4 "Communication Networks in Minikube, CNI, and CoreDNS"
Date of create: 13.01.2025  
Date of finished: 13.01.2025

### Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

## Описание
Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают underlay и overlay сети, а управление может быть организованно различными CNI.

## Цель работы
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

## Ход работы

Запустим minikube с calico и Multi-Node Clusters `minikube start -p lab4 --network-plugin=cni --cni=calico --nodes 2`, результат:
Можно проверить количество нод командой `kubectl get nodes`:
```bash
    NAME       STATUS   ROLES           AGE   VERSION
    lab4       Ready    control-plane   45s   v1.31.0
    lab4-m02   Ready    <none>          12s   v1.31.0
```
Эта команда запускает Minikube с двумя рабочими узлами и устанавливает Calico в качестве плагина CNI.


А так же проверить, что calico установлен и работает `kubectl get pods -n kube-system`, результат:
```bash
    NAME                                       READY   STATUS     RESTARTS      AGE
    calico-kube-controllers-7fbd86d5c5-z8t4g   1/1     Running    0             55s
    calico-node-45j57                          0/1     Init:2/3   0             30s
    calico-node-5n6k5                          1/1     Running    0             55s
    coredns-6f6b679f8f-llt46                   1/1     Running    0             55s
    etcd-lab4                                  1/1     Running    0             61s
    kube-apiserver-lab4                        1/1     Running    0             61s
    kube-controller-manager-lab4               1/1     Running    0             61s
    kube-proxy-bhlgm                           1/1     Running    0             30s
    kube-proxy-bptml                           1/1     Running    0             55s
    kube-scheduler-lab4                        1/1     Running    0             61s
    storage-provisioner                        1/1     Running    1 (27s ago)   59s
```

Далее установим labels для minikube и minikube-m02:
``` bash
    kubectl label nodes lab4 zone=west
    kubectl label nodes lab4-m02 zone=east
```

Вызвать список labels для nodes можно командой `kubectl get nodes --show-labels`, результат:
```bash
    NAME       STATUS   ROLES           AGE    VERSION   LABELS
    lab4       Ready    control-plane   2m2s   v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=lab4,kubernetes.io/os=linux,minikube.k8s.io/commit=210b148df93a80eb872ecbeb7e35281b3c582c61,minikube.k8s.io/name=lab4,minikube.k8s.io/primary=true,minikube.k8s.io/updated_at=2025_01_14T01_29_07_0700,minikube.k8s.io/version=v1.34.0,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=,zone=west
    lab4-m02   Ready    <none>          89s    v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=lab4-m02,kubernetes.io/os=linux,minikube.k8s.io/commit=210b148df93a80eb872ecbeb7e35281b3c582c61,minikube.k8s.io/name=lab4,minikube.k8s.io/primary=false,minikube.k8s.io/updated_at=2025_01_14T01_29_36_0700,minikube.k8s.io/version=v1.34.0,zone=east

```

Установим CNI применив манифест calico.yaml, скачанный с официального сайта. Загружаем и разворачиваем манифест `kubectl apply -f calicoctl.yaml`:
```bash
serviceaccount/calicoctl created
pod/calicoctl created
clusterrole.rbac.authorization.k8s.io/calicoctl created
clusterrolebinding.rbac.authorization.k8s.io/calicoctl created
```

Необходимо создать calico манифест для распределения пула адресов:
``` yaml
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
   name: zone-east-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
---
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
   name: zone-west-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "west"
```
Удалим дефолтный IPPool `kubectl delete ippools default-ipv4-ippool`   
Применим манифест `kubectl apply -f calico\ippools.yaml`:
```bash
    ippool.crd.projectcalico.org/zone-east-ippool created
    ippool.crd.projectcalico.org/zone-west-ippool created
```
1. Применим ранее созданный configMap:<br />
`kubectl apply -f configmaps\reactapp-env.yaml`<br />
2. Применим ранее созданный deployment:<br />
`kubectl apply -f deployment\frontend-deployment.yaml`<br />
3. Применим ранее созданный service:<br />
`kubectl apply -f services\nginx-service.yaml`
4. Осуществим проброс портов <br />`kubectl port-forward service/lab3-frontend-service 3000:3000`:
![Скрин](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab4/chrome.png 'Скрин')

После подключения к поду `kubectl exec -ti lab3-frontend-54bd764999-9h5jm -- sh` выполним пинг на FQDN и IP адрес:
```bash
frontend # ping lab3-frontend-service.lab3-frontend-54bd764999-rmhtz
ping: bad address 'lab3-frontend-service.lab3-frontend-54bd764999-rmhtz'
/frontend # ping lab3-frontend-service.default.svc.cluster.local
ping: bad address 'lab3-frontend-service.default.svc.cluster.local'
/frontend # ping 192.168.0.193
PING 192.168.0.193 (192.168.0.193): 56 data bytes
```

### Диаграмма
Схема организации calico, service и подов нарисованная в [draw.io](https://app.diagrams.net/).
![Диаграмма](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab3/lab4.drawio.png 'Диаграмма')