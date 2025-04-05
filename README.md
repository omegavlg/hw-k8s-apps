# Домашнее задание к занятию "`Запуск приложений в K8S`" - `Дедюрин Денис`

---
## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

### Ответ:

1. Создаем манифест и применяем его:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "81"
        ports:
        - containerPort: 81
        name: http-port
```

```
sudo kubectl apply -f deployment.yaml
```

Проверяем, создались ли поды:

```
sudo kubectl get pods
```

<img src = "img/01.png" width = 100%>

2. Увеличиваем количество реплик до **2** (меняем **replicas: 1** → **replicas: 2**) и повторно применяем манифест:

```
sudo kubectl apply -f deployment.yaml
```

Проверяем, создались ли поды:

```
sudo kubectl get pods
```

<img src = "img/02.png" width = 100%>

3. Создаем сервис, который обеспечит доступ до реплик приложений и применяем его:

```
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
spec:
  ports:
    - name: nginx
      protocol: TCP
      port: 80
    - name: multitool
      protocol: TCP
      port: 81
  selector:
    app: nginx
```

```
sudo kubectl apply -f service.yaml
```

Проверяем, создался ли сервис:

```
sudo kubectl get svc
```

<img src = "img/03.png" width = 100%>


4. Создаем отдельный Pod с приложением multitool:









---
## Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

### Ответ:

