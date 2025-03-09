# Lesson-8

## Зупинка Docker'a
```
sudo systemctl stop docker.socket
sudo systemctl stop docker          # docker.service
```
## Створення манифестів
Опис в середині кожного YAML файлів

### Namespace
l8-namespace.yaml [testenv]
```
microk8s.kubectl apply -f l8-namespace.yaml
microk8s.kubectl get namespaces
```

### Pod
l8-po.yaml [image: apache2:latest]
```
microk8s.kubectl apply -f l8-service.yaml -n testenv
microk8s.kubectl get pods -n testenv
microk8s.kubectl describe pod l8-pod -n testenv

microk8s.kubectl delete pod l8-pod -n testenv
```

### Service
l8-service.yaml [apache2]
```
microk8s.kubectl apply -f l8-pod.yaml -n testenv
microk8s.kubectl get services -n testenv
microk8s.kubectl describe service l8-service -n testenv
microk8s.kubectl get endpoints l8-service -n testenv
```
**RESULT**
```
NAME         ENDPOINTS         AGE
l8-service   10.1.227.154:80   5m53s
```
```
microk8s.kubectl get services l8-service -o wide -n testenv
```
**RESULT**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
l8-service   ClusterIP   10.152.183.49   <none>        8080/TCP   19m   app=apache2,environment=testenv
```

## Deployment
l8-deployment.yaml
```
microk8s.kubectl apply -f l8-deployment.yaml -n testenv
microk8s.kubectl get deployments -n testenv
microk8s.kubectl describe deployment l8-deployment -n testenv

microk8s.kubectl delete deployment l8-deployment -n testenv
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
    
