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
l8-po.yaml [image: httpd:latest]
```
microk8s.kubectl apply -f l8-service.yaml -n testenv
microk8s.kubectl get pods -n testenv
microk8s.kubectl describe pod l8-pod -n testenv

microk8s.kubectl delete pod l8-pod -n testenv
```

### Service
l8-service.yaml [httpd]
```
microk8s.kubectl apply -f l8-pod.yaml -n testenv
microk8s.kubectl get services -n testenv
microk8s.kubectl describe service l8-service -n testenv
microk8s.kubectl get endpoints l8-service -n testenv
```
**RESULT**
```
NAME         ENDPOINTS         AGE
l8-service   10.1.227.167:80   7h34m
```
```
microk8s.kubectl get services l8-service -o wide -n testenv
```
**RESULT**
```
NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE     SELECTOR
l8-service   NodePort   10.152.183.49   <none>        8080:30080/TCP   7h34m   app=httpd,environment=testenv
```
```
microk8s.kubectl get nodes -o wide -n testenv
```
**RESULT**
```
NAME   STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
r-d    Ready    <none>   26d   v1.32.1   10.0.2.15     <none>        Ubuntu 24.04.2 LTS   6.8.0-52-generic   containerd://1.6.36
```

## Deployment
l8-deployment.yaml
```
microk8s.kubectl apply -f l8-deployment.yaml -n testenv
microk8s.kubectl get deployments -n testenv
microk8s.kubectl describe deployment l8-deployment -n testenv

microk8s.kubectl delete deployment l8-deployment -n testenv
```
```
microk8s.kubectl get deployment -o wide -n testenv
```
**RESULT**
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES         SELECTOR
l8-deployment   3/3     3            3           30s   l8-container   httpd:latest   app=httpd,environment=testenv
```

## Перевірка працездатності

### Наявні pods
```
microk8s.kubectl get pods -o wide -n testenv
```
NAME                            READY   STATUS    RESTARTS   AGE    IP             NODE   NOMINATED NODE   READINESS GATES
l8-deployment-bcb99f5dc-cglxd   1/1     Running   0          74m    10.1.227.169   r-d    <none>           <none>
l8-deployment-bcb99f5dc-q5nxx   1/1     Running   0          74m    10.1.227.181   r-d    <none>           <none>
l8-deployment-bcb99f5dc-wcdxz   1/1     Running   0          74m    10.1.227.174   r-d    <none>           <none>
l8-pod                          1/1     Running   0          142m   10.1.227.167   r-d    <none>           <none>
```

**Доступність вебсерера**

```
curl http://127.0.0.1:30080
```
```
<html><body><h1>It works!</h1></body></html>
```
**Видаляння окремого pod, щоб "не заважав" перевірці deployment**
```
microk8s.kubectl delete pod l8-pod -n testenv
```
```
microk8s.kubectl get pods -o wide -n testenv
```
NAME                            READY   STATUS    RESTARTS   AGE   IP             NODE   NOMINATED NODE   READINESS GATES
l8-deployment-bcb99f5dc-cglxd   1/1     Running   0          77m   10.1.227.169   r-d    <none>           <none>
l8-deployment-bcb99f5dc-q5nxx   1/1     Running   0          77m   10.1.227.181   r-d    <none>           <none>
l8-deployment-bcb99f5dc-wcdxz   1/1     Running   0          77m   10.1.227.174   r-d    <none>           <none>
```

**Доступність вебсерера для deployment**
Більше **l8-pod** немає, тому перевіряємо доступність вебсервера з deployment (успішно) 
```
curl http://127.0.0.1:30080
```
```
<html><body><h1>It works!</h1></body></html>
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
    
