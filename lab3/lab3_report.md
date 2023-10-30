University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110C  
Author: Doronin Dmitrii Alekseevich  
Lab: Lab3  
Date of create: 26.10.2023  
Date of finished:

## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."
#### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
#### Ход выполнения работы
Ingress в Kubernetes – это API объект, который управляет внешним доступом к сервисам в кластере. Ingress может обеспечивать маршрутизацию по FQDN.
Перед тем как приступить в созданию сертификата и ingress, аналогично ЛР№2 создаются ConfigMap, Deployment с ReplicaSet, и LoadBalancer для доступа к подам, описанные в манифесте lab3-base:
```shell
kubectl apply -f lab3-base.yaml
kubectl get configmap
kubectl get deployment
kubectl get service
kubectl get pod
```
<details>
<summary>lab3-base.yaml</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: lab3-cfg
data:
  REACT_APP_USERNAME: "doronin"
  REACT_APP_COMPANY_NAME: "itmo"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ifilyaninitmo/itdt-contained-frontend:master
        envFrom:
          - configMapRef:
              name: lab3-cfg
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: lab3-service
spec:
  selector:
    app: frontend
  ports:
    - port: 3000
      protocol: TCP
      name: http
  type: LoadBalancer
```
</details>

![screenshot_1](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/80060236-2585-4fc7-aea3-db7428985a2e)

После этого, с использованием openssl создаётся TLS сертификат:
```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=lab3test.com"
```
В этой команде:
 - openssl req генерирует новый сертификат и ключ.
 - x509 указывает на создание самоподписанного сертификата.   
 - days 365 указывает срок действия сертификата (365 дней).   
 - newkey rsa:2048 создает новый ключ RSA длиной 2048 бит.   
 - tls.key и tls.crt - это имена файлов для ключа и сертификата соответственно.   
 - "/CN=lab3test.com" - это имя хоста, для которого предназначен сертификат.   

Этот сертификат добавляется в кластер в виде секрета:
```shell
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
```
  
![231030_03h36m15s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/851ab2a7-3a58-491c-96e9-d40a31f5be11)

Был установлен аддон Ingress для Minikube. Это необходимо для управления внешним доступом к сервисам в кластере. Аддон был активирован следующими командами:
```shell
minikube addons enable ingress
```
После активации Ingress, был создан ресурс Ingress, описание которого содержится в файле lab3-ingress.yaml. Применение конфигурации и проверка статуса Ingress производились с помощью команд:

```shell
kubectl apply -f lab3-ingress.yaml
kubectl get ingress
```

<details>
<summary>lab3-ingress.yaml</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
spec:
  rules:
  - host: lab3test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lab3-service
            port:
              number: 3000
  tls:
  - hosts:
    - lab3test.com
    secretName: tls-secret
```
</details>

![231030_03h38m45s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/4d787869-812c-4b64-9d0b-6eee106f762d)


Также был отредактирован файл hosts для доступа к сервису по доменному имени, добавив в hosts `minikube ip` и `FQDN`: `192.168.49.2 lab3test.com`.   

На этом этапе работы очень долго было не понятно почему не подключится к моему сервису ни из браузера, ни по пингу (fqdn). В итоге было обнаружено, что проблема в docker desktop -> https://github.com/kubernetes/ingress-nginx/issues/7686.
После удаления docker desktop, и установки docker, можно получить доступ к сервису в браузере и посмотреть сертификат:

![231030_03h39m27s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/e2dbc59b-926a-40e3-83fa-3b1b133eb1bf)

![231030_03h39m35s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/a952497c-aa33-4e26-90dc-e1ca18cd1d8a)


#### Схема организации контейеров и сервисов
![231030_04h52m58s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/5582dcb1-dd05-4796-9ede-48cdb30831b9)
