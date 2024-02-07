# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-clusterip
  labels:
    app: deploy-clusterip
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-clusterip
  template:
    metadata:
      labels:
        app: deploy-clusterip
      namespace: default
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.4
        imagePullPolicy: IfNotPresent
      - name: multitool
        image: wbitt/network-multitool
        imagePullPolicy: IfNotPresent
        env:
          - name: HTTP_PORT
            value: '8080'
          - name: HTTPS_PORT
            value: '8081'
```

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
  namespace: default
spec:
  ports:
    - name: nginx
      port: 9001
      targetPort: 80
    - name: multitool
      port: 9002
      targetPort: 8080
  selector:
    app: deploy-clusterip
  type: ClusterIP
```

<p align="center">
    <img width="1200 height="600" src="/img/deploy_clusterip.png">
</p>

3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

<p align="center">
    <img width="1200 height="600" src="/img/curl_clusterip1.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/curl_clusterip1.png">
</p>

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
  namespace: default
spec:
  ports:
    - name: http-nginx
      port: 9001
      nodePort: 30080
      targetPort: 80
  selector:
    app: deploy-clusterip
  type: NodePort
```

2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

<p align="center">
    <img width="1200 height="600" src="/img/nodeport_src.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/curl_nodeport.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/browser_nodeport.png">
</p>    

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

