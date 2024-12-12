University: [ITMO University](https://itmo.ru/ru/)   
Faculty: [FICT](https://fict.itmo.ru)   
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)   
Year: 2024/2025   
Group: K4111c   
Author: Shabashov Denis Igorevich   
Lab: Laboratory Work No. 1 "Installing Docker and Minikube, My First Manifest"  
Date of create: 29.11.2024  
Date of finished: 05.12.2025

## Описание
Это первая лабораторная работа в которой вы сможете протестировать Docker, установить Minikube и развернуть свой первый "под".

## Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

## Kubernetes (K8S) 
### Суть технологии
**Kubernetes** - платформа, которая автоматизирует распределение и выполнение контейнеров приложений для запуска в **кластере** более эффективным образом.

  Задача Kubernetes заключается в координации кластера компьютеров, работающего как **одно целое**. Абстрактные объекты в Kubernetes позволяют развертывать контейнеризированные приложения в кластер, **не привязывая** их к отдельным машинам. 

### Схема кластера
Кластер Kubernetes состоит из двух типов ресурсов:
* Мастер - ведущий узел, который управляет кластером.
* Рабочие узлы - машины, на которых выполняются приложения.

**Узел** - это виртуальная машина или физический компьютер, который выполняет роль рабочего узла в кластере Kubernetes.

У каждого узла есть **Kubelet** - агент, управляющий узлом и взаимодействующий с ведущим узлом Kubernetes. Узел также имеет инструменты для выполнения контейнерных операций, например, **Docker** или rkt. Узлы взаимодействуют с ведущим узлом посредством **API Kubernetes**, который предлагает ведущий узел.

> Запрос -> API Server -> Kubelet -> Pod

### Pod
Pod - минимальная развертываемая единица в K8S, набор из одного и более контейнеров, имеющих общее пространство имен и тома общей файловой системы.

> Надо отметить, что контейнеры имеют собственные изолированные файловые системы, но они могут совместно использовать данные, пользуясь ресурсом K8S, который называется **Volume (том)**.

У каждого пода есть уникальный IP-адрес. Для описания пода пишут манифест (manifest file).

Kubernetes-кластер может быть развернут на **физических** или **виртуальных** машинах. Чтобы начать работать с Kubernetes, можно использовать **Minikube**. 

--- 
## Minikube
### Суть технологии
**Minikube** - это упрощённая реализация Kubernetes, которая создает виртуальную машину на вашем локальном компьютере и разворачивает простой кластер с одним узлом. Minikube доступен для Linux, macOS и Windows.

---
## Ход работы

> Установить Docker на рабочий компьютер
> Установить Minikube используя оригинальную инструкцию
> 
### Создание контейнера vault
Скачиваем образ vault командой `docker pull vault`.  
Проверяем, что появился образ vault - `docker images`.
Создаем контейнер на основе образа vault - `docker run -d --name vault vault`.  
Проверяем, что появился контейнер vault - `docker ps -a`.  
![Контейнер vault]( 'Контейнер vault')

### Создание Pod
Запускаем minikube - `minikube start`.  
Проверяем, что появился узел - `kubectl get nodes`.  

Создадим **manifest file**, в котором будет описан наш под.  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: hashicorp/vault:latest
    ports:
    - containerPort: 8200
    env:
    - name: VAULT_ADDR
      value: "http://127.0.0.1:8200"
```
Переходим в папку с .yaml файлом и выполняем команду `kubectl create -f vault_pod.yaml`.

### Создание сервиса
Создаем сервис для доступа к Pod - `minikube kubectl -- expose pod vault --type=NodePort --port=8200`.

Перенаправляем трафик с Pod на локальный - `minikube kubectl -- port-forward service/vault 8200:8200`.

Открываем страницу авторизации Vault `http://localhost:8200`.

### Поиск токена
Чтобы найти токен для авторизации, открываем второй терминал и используем команду `minikube kubectl -- logs service/vault`.

> Root Token: hvs.4EhD9E1CRpRP4v0D5kv3pHQl

![Successful authorization](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab1/vault.png 'Successful authorization')

Работа выполнена - останавливаем узел командой `minikube stop`.

### Диаграмма
Схема организации контейнера и сервиса, нарисованная в [draw.io](https://app.diagrams.net/).

![Диаграмма](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab1/lab1.drawio.png 'Диаграмма')
