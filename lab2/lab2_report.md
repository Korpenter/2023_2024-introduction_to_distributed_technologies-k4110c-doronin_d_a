University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110C  
Author: Doronin Dmitrii Alekseevich  
Lab: Lab2  
Date of create: 12.10.2023  
Date of finished:  

## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."
#### Цель работы
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение. 
#### Ход выполнения работы
После скачиваения контейнера itdt-contained-frontend создаётся deployment описанный в файле frontend-deployment.yaml:
```shell
kubectl create -f frontend-deployment.yaml
```
<details>
<summary>frontend-deployment.yaml</summary>

```yaml
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
        env:
          - name: REACT_APP_USERNAME
            value: frontend
          - name: REACT_APP_COMPANY_NAME
            value: doronin
        ports:
          - containerPort: 3000
```
</details>
<details>
  
<summary>Terminal commands</summary>

![screenshot_1](https://github.com/Korpenter/krok-school-itmo/assets/141184937/88fb8962-cd8e-45a6-a816-0baf5d9ebd9e)
</details>

После развёртывания выбирается LoadBalancer сервис для доступа к подам, который описан в файле frontend-balancer.yaml. Сервис создаётся, и локальный порт пробрасывается на порт контейнера командами:
```shell
kubectl create -f frontend-balancer.yaml
kubectl -- port-forward service/frontend-port 8888:4110
```
<details>
<summary>frontend-Nodeport.yaml</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-balancer
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer

```
</details>
<details>
  
<summary>Terminal commands</summary>
  
![screenshot_2](https://github.com/Korpenter/krok-school-itmo/assets/141184937/370efa98-afb4-46b7-b54e-1582bca05990)
</details>
Открывается страница сервиса по адресу `http://localhost:8888`, которая при обновлении не меняется, потому что мы попадаем на 1 под:
<details>
  
<summary>Browser window</summary>

![screenshot_3](https://github.com/Korpenter/krok-school-itmo/assets/141184937/04eca102-8b0e-4bbf-af2a-a6886191edff)
</details>

Для просмотра логов пода сначала получается список всех подов, и после просматриваются логи обоих подов:
```shell
kubectl get pods
kubectl logs pod/frontend-57c48d687c-p9hmb
kubectl logs pod/frontend-57c48d687c-tm52j
```
<details>
  
<summary>Terminal commands</summary>

![screenshot_4](https://github.com/Korpenter/krok-school-itmo/assets/141184937/cda24b82-d004-4024-8b70-2cbbb7281d45)
</details>

#### Схема организации контейеров и сервисов
![DIAGRAMM](https://github.com/Korpenter/krok-school-itmo/assets/141184937/3320499e-a434-4a72-bee4-67af1f8ba9f6)
