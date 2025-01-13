University: [ITMO University](https://itmo.ru/ru/)   
Faculty: [FICT](https://fict.itmo.ru)   
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)   
Year: 2024/2025   
Group: K4111с   
Author: Shabashov Denis Igorevich   
Lab: Laboratory Work No. 3 "Certificates and Secrets in Minikube, Secure Data Storage"
Date of create: 12.01.2024  
Date of finished: 13.01.2025


# Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

## Описание
В данной лабораторной работе вы познакомитесь с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

## Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

## Ход работы
### Создание контейнера frontend-container
Скачиваем образ itdt-contained-frontend командой `docker pull ifilyaninitmo/itdt-contained-frontend:master`.
Запускаем кластер через minikube: `minikube start`

### Создание ConfigMap

Создаем в директории manifests/configmaps файл reactapp-env.yaml со следующим содержимым:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: reactapp
data:
  name: shabashov
  value: itmo_2025
```

Применяем манифест: `kubectl apply -f manifests\configmaps\reactapp-env.yaml`

### Создание ReplicaSet:

Создаем в директории manifests/replicasets файл frontend-replicaset.yaml со следующим содержимым:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: reactapp
data:
  name: shabashov
  value: itmo_2025
```

Применяем манифест: `kubectl apply -f manifests\replicasets\frontend-replicaset.yaml`

### Создание Ingress:

1. Подключение аддона Ingress `minikube addons enable ingress`
2. Генерирование ключа `openssl genrsa -out tls.key 2048`
3. Генерирование сертификата `openssl req -new -x509 -key tls.key -out tls.crt -days 365 -subj "/CN=shabashov.com/O=itmo"`
4. Создание Secret в K8S: `kubectl create secret tls shabashov-tls --cert=tls.crt --key=tls.key`

Создаем в директории manifests/services файл nginx-service.yaml со следующим содержимым:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: lab3-frontend-service
spec:
  type: NodePort
  selector:
    app: frontend-app-lab3
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      name: http
```

Применяем манифест: `kubectl apply -f manifests\services\nginx-service.yaml`

Создаем в директории manifests/ingress файл ingress.yaml со следующим содержимым:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab3-frontend-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - shabashov.com
    secretName: shabashov-tls
  rules:
  - host: shabashov.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lab3-frontend-service
            port:
              number: 3000
```

Применяем манифест: `kubectl apply -f manifests\ingress\ingress.yaml`

Запуск туннелирования, если кластер развернут через minikube под windows или macos: `minikube tunnel`

Запуск приложения в браузере:
![Скрин](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab3/firefox.png 'Скрин')

### Диаграмма
Схема организации ingress, service и подов нарисованная в [draw.io](https://app.diagrams.net/).
![Диаграмма](https://github.com/countenum404/2024_2025-introduction_to_distributed_technologies-k4111c-shabashov_d_i/blob/main/img/lab3/lab3.drawio.png 'Диаграмма')
