# Lesson-5

## **Підготовка контейнерів**
Використаний образ з попереднього завдання
```
nginx-from-ubuntu
```
Створення контейнерів
```
sudo docker run -d --name ngnx1-bridge -p 8081:80 nginx-from-ubuntu
sudo docker run -d --name ngnx2-bridge -p 8082:80 nginx-from-ubuntu
sudo docker run -d --name ngnx3-host –network host nginx-from-ubuntu
sudo docker run -d --name ngnx4-none –network none -p 8084:80 nginx-from-ubuntu
```
Додаткові команди
```
sudo apt-get install net-tools
sudo netstat -tulnp | grep :80
```


## **Корисні команди**
```
sudo docker tag nginx-from-ubuntu abigdan/nginx-from-ubuntu:0.2
sudo docker push abigdan/nginx-from-nginx:0.1
```
