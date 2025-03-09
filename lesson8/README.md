# Lesson-8

## Зупинка Docker'a
```
sudo systemctl stop docker.socket
sudo systemctl stop docker          # docker.service
```
## Створення манифестів
Опис в середині кожного YAML файлів

### Namespace testenv
```
microk8s.kubectl apply -f l8-namespace.yaml
microk8s.kubectl get namespaces
```

### Pod (image: linuxserver/openssh-server)
```
microk8s.kubectl apply -f l8-service.yaml -n testenv
microk8s.kubectl get pods -n testenv
microk8s.kubectl describe pod l8-pod -n testenv
```

### Service
openssh
```
microk8s.kubectl apply -f l8-pod.yaml -n testenv
microk8s.kubectl get services -n testenv
microk8s.kubectl describe service l8-service -n testenv
microk8s.kubectl get endpoints l8-service -n testenv
```
**RESULT**
NAME         ENDPOINTS         AGE
l8-service   10.1.227.154:22   5m53s

```
microk8s.kubectl get services l8-service -o wide -n testenv
```
**RESULT**
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
l8-service   ClusterIP   10.152.183.49   <none>        8022/TCP   19m   app=openssh,environment=testenv


У цьому прикладі всі Pod'и з `label: app=my-app` будуть отримувати трафік через сервіс `my-service`.

### **Перевірка доступності сервісу:**

Отримати список сервісів у кластері:
    
    ```
    microk8s.kubectl get services
    ```
    
Перевірити, які Pod'и обслуговує сервіс:
    
    ```
    microk8s.kubectl get endpoints my-service
             ```

Дізнатися IP-адресу сервісу:
    
    ```
    microk8s.kubectl get svc my-service -o wide
    ```


### **Приклад створення Deployment:**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment 
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container 
          image: nginx:latest
          ports:
            - containerPort: 80
```

### **Перевірка та управління Deployment:**

1. **Створення Deployment:**
    
    ```
    microk8s.kubectl apply -f deployment.yaml
    ```
    
2. **Перевірка статусу Deployment:**
    
    ```
    microk8s.kubectl get deployments
    ```
    
3. **Перегляд детальної інформації:**
    
    ```
    microk8s.kubectl describe deployment my-deployment
    ```
    
4. **Масштабування Deployment:**
    
    ```
    microk8s.kubectl scale deployment my-deployment --replicas=5
    ```
    
5. **Оновлення образу у Deployment:**
    
    ```
    microk8s.kubectl set image deployment my-deployment my-container=nginx:1.19
    ```
    
6. **Видалення Deployment:**
    
    ```
    microk8s.kubectl delete deployment my-deployment
    ```
    
