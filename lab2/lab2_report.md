University: [ITMO University](https://itmo.ru/ru/)   
Faculty: [FICT](https://fict.itmo.ru)   
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)   
Year: 2024/2025   
Group: K4111с   
Author: Shabashov Denis Igorevich   
Lab: Laboratory Work No. 2 "Deployment of a Web Service in Minikube, Access to the Web Interface of the Service. Service Monitoring."  
Date of create: 10.12.2024  
Date of finished: 12.12.2025

# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

## Описание
В данной лабораторной работе вы познакомитесь с развертыванием полноценного веб сервиса с несколькими репликами.

## Цель работы
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

---
## Deployment (развертывание)
Deployment - ресурс K8S, который позволяет автоматизировать процесс перехода от одной версии приложения к другой без прерывания работы системы.
* Поддержание системы в нужном состоянии (например, если мы удалим из развертывания 1 под, K8S запустет другой)
* Выполнение развёртываний с нулевым временем простоя системы
* Откат к предыдущему состоянию системы

---
## Ход работы

### Создание контейнера frontend-container
Скачиваем образ itdt-contained-frontend командой `docker pull ifilyaninitmo/itdt-contained-frontend:master`.

Проверяем, что появился образ itdt-contained-frontend - `docker images -a`.
Создаем контейнер на основе образа itdt-contained-frontend - `docker run -d --name frontend-container ifilyaninitmo/itdt-contained-frontend:master`.
Проверяем, что появился контейнер frontend_container - `docker ps -a`.  

### Создание Deployment

Запускаем minikube - `minikube start`. Иначе **context (контекст)** будет не minikube, а другой - например docker-desktop, и после запуска minikube, ваше развертывание не будет видно.

Пример манифеста для развертывания, также как и для пода можно найти в [официальной документации](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab2-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-app-lab2
  template:
    metadata:
      labels:
        app: frontend-app-lab2
    spec:
        containers:
        - name: frontend-app-lab2
          image: ifilyaninitmo/itdt-contained-frontend:master
          ports:
          - containerPort: 3000
          env:
          - name: REACT_APP_USERNAME
            value: shabashov
          - name: REACT_APP_COMPANY_NAME
            value: itmo
```

Чтобы запустить 2 экземпляра пода, используем свойства `replicas: 2`.

Шаблон пода задается в объекте `Template`. С помощью свойства `env` объявляем внутри подов переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME` со значениями `Anatolii` и `ITMO`, соответственно.

Переходим в папку с .yaml файлом и выполняем команду `kubectl create -f frontend-deployment.yaml`.

### Создание сервиса frontend-service

Создаем сервис для доступа к развертыванию - `minikube kubectl -- expose deployment frontend --port=3000 --target-port=3000 --name=frontend-service --type=LoadBalancer`.

Тип сервиса `NodePort`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-app-lab2-service
spec:
  type: NodePort
  selector:
    app: frontend-app-lab2
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30001
```

Пробрасываем локальный порт на порт контейнера - `minikube kubectl -- port-forward service/frontend-service 3000:3000`.

Открываем страницу `http://localhost:3000`.

![Frontend page](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab2/result.png 'Frontend page')

### Логи подов

Смотрим логи первого пода - `minikube kubectl -- logs pod/frontend-9c975bc96-b5q2z`.

Смотрим логи второго пода - `minikube kubectl -- logs pod/frontend-9c975bc96-kh262`.

Чтобы удалить развертывание используем команду - `kubectl delete deployments/frontend`.

Проверяем - `kubectl get deployments` и останавливаем minikube командой `minikube stop`.

### Итоги   
Ресурс deployment позволяет создавать несколько подов на основе одного контейнера.
В Kubernetes существует несколько типов сервисов (Services), каждый из которых предназначен для различных сценариев использования и обеспечивает разные способы доступа к подам. Основные типы сервисов включают:
1. ClusterIP

    Описание: Это тип сервиса по умолчанию. Он создает виртуальный IP-адрес (Cluster IP), который доступен только внутри кластера Kubernetes.
    Использование: Подходит для внутреннего общения между подами. Например, если у вас есть несколько микросервисов, которые должны взаимодействовать друг с другом, вы можете использовать ClusterIP.

2. NodePort

    Описание: Этот тип сервиса выделяет порт на каждом узле кластера, который перенаправляет трафик на соответствующий ClusterIP. Вы можете получить доступ к сервису, обращаясь к любому узлу с использованием выделенного порта.
    Использование: Удобен для доступа к сервисам из внешней сети, когда вы не используете LoadBalancer. Например, для тестирования приложения или разработки.

3. LoadBalancer

    Описание: Этот тип сервиса автоматически создает внешний балансировщик нагрузки (например, в облачных провайдерах, таких как AWS, GCP или Azure), который перенаправляет трафик на поды сервиса.
    Использование: Идеален для приложений, которые должны быть доступны из интернета. Он предоставляет внешний IP-адрес, который можно использовать для доступа к сервису.

4. ExternalName

    Описание: Этот тип сервиса позволяет вам использовать DNS-имя внешнего ресурса вместо IP-адреса. Он не создает ни ClusterIP, ни NodePort, а просто перенаправляет запросы на указанный DNS-адрес.
    Использование: Полезен для интеграции с внешними сервисами, такими как базы данных или API, которые находятся вне кластера Kubernetes.

5. Headless Service

    Описание: Это сервис без ClusterIP, который позволяет вам обращаться к подам напрямую. Он не создает виртуальный IP-адрес и не выполняет балансировку нагрузки.
    Использование: Используется в случаях, когда вам нужно получить доступ к каждому поду индивидуально, например, для StatefulSets или при использовании сервисов, которые требуют прямого доступа к подам.


    Именно поэтому логи первого и второго пода идентичны.

### Диаграмма
Схема организации контейнера и сервиса, нарисованная в [draw.io](https://app.diagrams.net/).
![Диаграмма](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab2/lab2.drawio.png 'Диаграмма')
