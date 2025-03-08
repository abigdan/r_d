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


### **Перегляд ресурсів у конкретному просторі імен:**

```
microk8s.kubectl get pods -n my-namespace
```

### **Видалення простору імен:**

```
microk8s.kubectl delete namespace my-namespace
```

#### **Приклад створення Pod:**

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

Після створення YAML-файлу, застосуйте його в кластері за допомогою команди:

```
microk8s.kubectl apply -f my-pod.yaml
```

Щоб перевірити, чи Pod успішно створений, виконайте:

```
microk8s.kubectl get pods
```

Для отримання детальної інформації про Pod використовуйте:

```
microk8s.kubectl describe pod my-pod
```


### **Приклад створення Service**

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

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
    
