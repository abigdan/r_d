# Lesson-3

### **Перевірка версії Docker**

```
docker --version
```

### **Запуск контейнера з образу**

```
docker run hello-world
```
Ця команда запускає офіційний тестовий контейнер, що перевіряє коректність встановлення Docker.


### **Список запущених контейнерів**

```
docker ps
```

### **Список усіх контейнерів (включаючи зупинені)**

```
docker ps -a
```

### **Запуск контейнера у фоновому режимі**

```
docker run -d --name my-nginx -p 8080:80 nginx
```

### **Перегляд логів контейнера**

```
docker logs my-nginx
```

### **Перегляд специфікації контейнера**

```
docker inspect my-nginx
```

### **Зупинка контейнера**

```
docker stop my-nginx
```

### **Видалення контейнера**

```
docker rm my-nginx
```

### **Список завантажених образів**

```
docker images
```

### **Видалення образу**

```
docker rmi nginx
```


## **Docker не запускається**

```
sudo systemctl start docker
```

Якщо є проблеми зі стартом:

```
sudo systemctl status docker
sudo journalctl -u docker --no-pager | tail -n 50
```

### **Проблема з правами доступу**

Якщо Docker вимагає `sudo`, перевірте групу:

```
groupadd docker
usermod -aG docker $USER
newgrp docker
```

### **Перевірити чи працює в системі containerd**

```
sudo systemctl status containerd
```


## **Створення власного образу Docker**

1. Створіть директорію `myapp` і перейдіть у неї:
    
    ```
    mkdir myapp && cd myapp
    ```
    
2. Створіть файл `Dockerfile`:
    
    ```
    FROM ubuntu:latest
    RUN apt update && apt install -y curl
    CMD ["echo", "Hello from my custom container!"]
    ```
    
3. Побудуйте образ:
    
    ```
    docker build -t my-custom-image .
    ```
    
4. Запустіть контейнер з нового образу:
    
    ```
    docker run my-custom-image
    ```


## **Запуск простого веб-сервера у Docker**

1. Створіть файл `index.html`:
    
    ```
    <html>
    <body>
      <h1>Hello, Docker!</h1>
    </body>
    </html>
    ```
    
2. Створіть `Dockerfile` для веб-сервера Nginx:
    
    ```
    FROM nginx:latest
    COPY index.html /usr/share/nginx/html/index.html
    ```
    
3. Побудуйте образ:
    
    ```
    docker build -t my-nginx .
    ```
    
4. Запустіть контейнер:
    
    ```
    docker run -d -p 8080:80 my-nginx
    ```
    
5. Перевірте роботу сервера, відкривши у браузері:
    
    ```
    curl http://localhost:8080
    ```
    

    
