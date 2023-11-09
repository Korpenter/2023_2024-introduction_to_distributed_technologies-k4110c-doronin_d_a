University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110C  
Author: Doronin Dmitrii Alekseevich  
Lab: Lab4  
Date of create: 30.10.2023  
Date of finished: 11.9.2023

## Лабораторная работа №4 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."
#### Цель работы
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.
#### Ход выполнения работы

Для выполненияя задания и использования calico, требуется запустить minikube с уустановкой соответствующего CNI плагина, а также задав требуемое количество нод (2):
```shell
minikube start --network-plugin=cni --cni=calico --nodes 2 -p lab4
```
После чего проверяется состояние кластера. Для этого проверяется количество нод и подов calico:
```shell
kubectl get nodes
kubectl get pod -l k8s-app=calico-node -A
```
</details>

![231030_03h42m09s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/abdbd2b0-3f33-49af-94b3-9520b0f0b3ca)


Для того, чтобы создать успешно создать IPPool сначала требуется пометить ноды. В данном случае ноды помечаются по зонам (zone):
```shell
kubectl label nodes lab4 zone=central  
kubectl label nodes lab4-m02 zone=east
kubectl get nodes -l zone=central
kubectl get nodes -l zone=east
```
![231030_03h42m46s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/4bd526f6-aaa4-4e26-83eb-c7d33e3f5a8a)


После чего создаётся два IPPool-а для обоих зон, используя calicoctl и манифест ippool.yaml и удаляется IPPool по умолчанию:
```shell
calicoctl delete ippools default-ipv4-ippool --allow-version-mismatch
calicoctl create -f ippool.yaml --allow-version-mismatch
calicoctl get ippool -o wide --allow-version-mismatch
```
<details>
<summary>ippool.yaml</summary>

```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: central-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "central"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name:  east-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
```
</details>

![231030_03h43m38s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/9f45ba1b-0f8a-46fd-9933-c3baea2cabba)


```shell

```
Переиспользуется deployment из ЛР3, после чего можно подключится к сервису по `minikube tunnel`, и зайти в браузер, где будет видно, что поды меняются так-как был выбран сервис LoadBalancer:
```shell
kubectl apply -f lab3-base.yaml
kubectl get pod -o wide
minikube profile list
minikube tunnel -p=lab4
```
![231030_03h46m14s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/32f5e217-f257-461b-a1ca-5601c094b567)

![231030_03h58m31s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/eb5389e8-a550-4949-bdf6-6dbc18ddcfd0)


После подключения к одному из подов, используя `kubectl exec -it <pod> -- /bin/sh`, и получения FQDN подов, используя `nslookup <ip>`, другой под пингуется командой `ping <FQDN or IP>`:

![231030_04h14m34s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/717fba87-046c-4d0c-8aa3-8f5c3328e7e1)


#### Схема организации контейеров и сервисов
![231030_05h00m19s_screenshot](https://github.com/Korpenter/2023_2024-introduction_to_distributed_technologies-k4110c-doronin_d_a/assets/141184937/ef2b46cf-b477-4878-acb5-b5b7c7080e76)

