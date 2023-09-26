University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110C  
Author: Doronin Dmitrii Alekseevich  
Lab: Lab1  
Date of create: 25.09.2023  
Date of finished:  

## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."
#### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".
#### Ход выполнения работы
После установки minikube и развёртки minikube cluster, скачиватся docker-образ vault командой:
```shell
docker pull vault:1.13.3
```
<details>
<summary>Terminal output</summary>
  
![screenshot_1](https://github.com/Korpenter/url-shortener/assets/141184937/ebdf5346-a0cc-4032-9f7b-d5d91ae413e2)
</details>


После загрузки образа создаётся описание манифеста - файл vault-pod.yaml:
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
    image: vault:1.13.3
    ports:
    - containerPort: 8200
```
Создаётся Pod и проверяется его состояние командами:
```shell
kubectl create -f vault-pod.yam
kubectl get pods
```
<details>
<summary>Terminal output</summary>
  
![screenshot_2](https://github.com/Korpenter/url-shortener/assets/141184937/4166009f-0bcc-419e-99c6-13b16e92e6eb)
</details>

Создаётся сервис для получения доступка к Vault, трафик перенаправляется на локальный порт, и для проверки корректности выполненых действий, открывается страница авторизации Vault `http://localhost:8200`,:
```shell
minikube kubectl -- expose pod vault --type=NodePort --port=8200
minikube kubectl -- port-forward service/vault 8200:8200
```
<details>
<summary>Terminal output</summary>
  
![screenshot_3](https://github.com/Korpenter/url-shortener/assets/141184937/c953489b-1887-429d-96a9-f19c661e8752)
</details>
<details>
<summary>Vault screenshot</summary>

![screenshot_4](https://github.com/Korpenter/url-shortener/assets/141184937/eac2028f-97d1-4983-ba3b-2e4b3d470a1c)
</details>

 и находится токен для авторизации в логах пода, выполнив команду:
```shell
kubectl logs service/vault
```
<details>
<summary>Terminal output</summary>
  
![screenshot_5](https://github.com/Korpenter/url-shortener/assets/141184937/c5a06311-6f5a-4416-94ef-8ccc48d4b0c0)
![screenshot_6](https://github.com/Korpenter/url-shortener/assets/141184937/8c64bf93-0373-43af-9527-7973a8bad58d)
</details>
<details>
<summary>Vault Screenshot</summary>

![screenshot_7](https://github.com/Korpenter/url-shortener/assets/141184937/27e4f510-ce16-44d6-bf26-13a00747a338)
</details>

minikube cluster останавливается командой:
```shell
minikube stop
```
<details>
<summary>Terminal output</summary>
  
![screenshot_8](https://github.com/Korpenter/url-shortener/assets/141184937/7c48287a-a3d4-486a-9536-b29b61d1bc0f)
</details>

#### Схема организации контейеров и сервисов
![DIAGRAMM](https://github.com/Korpenter/url-shortener/assets/141184937/6c4fb0f9-ab49-47e8-8865-4f5ddb6eb539)
