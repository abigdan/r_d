# Lesson-4

### **Структура Dockerfile:**

Кожен Dockerfile складається з набору інструкцій, серед яких:

- `FROM` – визначає базовий образ.
    
- `RUN` – виконує команди під час створення образу.
    
- `COPY` / `ADD` – копіюють файли з локальної машини в контейнер, але мають відмінності:
    
    - `COPY` просто копіює файли або директорії у контейнер.
        
    - `ADD` також може завантажувати файли за URL та автоматично розпаковувати архіви (наприклад, `.tar`).
        
- `WORKDIR` – задає робочу директорію всередині контейнера.
    
- `CMD` / `ENTRYPOINT` – визначають команду, яка буде виконуватися при запуску контейнера, але мають важливі відмінності:
    
    - **CMD** використовується для задання команди за замовчуванням, але її можна перевизначити під час запуску контейнера (`docker run <image> <new_command>`),  
        використання COPY робить Dockerfile більш передбачуваним та зрозумілим.
        
    - **ENTRYPOINT** задає основну команду контейнера, яку важко змінити при запуску, вона надає більшу гнучкість у поєднанні з аргументами (`docker run <image> <args>`).
        
- `EXPOSE` – вказує, який порт контейнер відкриває для взаємодії.

### **Приклад простого Dockerfile:**

```
# Визначаємо базовий образ
FROM ubuntu:latest

# Оновлюємо пакети та встановлюємо curl
RUN apt update && apt install -y curl

# Копіюємо локальний файл у контейнер
COPY index.html /var/www/html/index.html

# Визначаємо команду, яка виконується при запуску контейнера
CMD ["echo", "Docker container is running"]
```

## Створення власного Docker образу

Для створення власного образу потрібно:

### **Крок 1: Створення робочої директорії**

```
mkdir my-docker-image && cd my-docker-image
```

### **Крок 2: Створення Dockerfile**

```
touch Dockerfile
nano Dockerfile
```

### **Крок 3: Написання Dockerfile**

```
FROM ubuntu:latest
RUN apt update && apt install -y nginx
COPY index.html /var/www/html/
CMD ["nginx", "-g", "daemon off;"]
```

### **Крок 4: Створення образу**

```
docker build -t my-nginx-image .
```

### **Крок 5: Перевірка створеного образу**

```
docker images
```

### **Крок 6: Запуск контейнера з власного образу**

```
docker run -d -p 8080:80 my-nginx-image
```

Перевірте роботу сервера, відкривши у браузері `http://localhost:8080`

## Управління образами через Docker Hub


### **У якому форматі зберігаються Docker образи?**

Docker образи зберігаються у вигляді **багатошарових файлових систем (UnionFS layers)**, що працюють за принципом "Copy-on-Write". Кожен Docker образ складається з набору шарів:

- **Базовий образ** (наприклад, `ubuntu:latest`)
    
- **Шари змін (**`******RUN******`**************,** `******************COPY******************`**,** `******************ADD******************`**)**
    
- **Останній шар – контейнерний рівень, який змінюється під час виконання**
    

Docker використовує такі файлові системи для зберігання образів:

- **OverlayFS (Overlay2)** – найбільш ефективний драйвер для сучасних систем.
    
- **AUFS** – старий драйвер, який використовувався раніше.
    
- **Btrfs та ZFS** – підтримують більш розширені можливості збереження даних.
    

Щоб перевірити, який драйвер використовується у вашій системі:

```
docker info | grep "Storage Driver"
```

---

Docker Hub – це публічний репозиторій образів, який дозволяє зберігати, публікувати та завантажувати Docker образи. Крім Docker Hub, існують альтернативні рішення для керування образами:

### **Авторизація в Docker Hub**

```
docker login -u po1yak
```

### **Тегування образу перед публікацією**

```
docker tag my-nginx-image mydockerhubuser/my-nginx-image:latest
```

### **Публікація образу в Docker Hub**

```
docker push mydockerhubuser/my-nginx-image:latest
```

### **Завантаження образу з Docker Hub**

```
docker pull mydockerhubuser/my-nginx-image:latest
```

## Оптимізація Dockerfile

### **Використання мінімальних базових образів**

Замість `ubuntu:latest`, використовуйте легковагові образи:

```
FROM alpine:latest
```

### **Об'єднання команд RUN**

Замість:

```
RUN apt update
RUN apt install -y nginx
```

Краще писати:

```
RUN apt update && apt install -y nginx && rm -rf /var/lib/apt/lists/*
```

Це зменшує кількість шарів у образі.

### **Використання** `**.dockerignore**`

Файл `.dockerignore` допомагає виключити непотрібні файли при копіюванні в образ.

```
touch .dockerignore
```

Приклад `.dockerignore`:

```
node_modules
.git
*.log
```

### **Мультистейдж-збірка**

Цей підхід дозволяє створювати ефективні образи, збираючи артефакти в одному контейнері та використовувати їх в іншому.

простий додаток на Go
```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, Docker!")
}

func main() {
	http.HandleFunc("/", handler)
	fmt.Println("Server is running on port 8080...")
	http.ListenAndServe(":8080", nil)
}

```

Dockerfile
```
# Етап 1: збірка додатка
FROM golang:latest AS builder

WORKDIR /app

# Копіюємо вихідний код у контейнер
COPY main.go .

# Компілюємо Go-додаток у виконуваний файл
RUN go mod init app && go mod tidy && go build -o app

# Етап 2: створення легковагового фінального образу
FROM alpine:latest

WORKDIR /root/

# Копіюємо скомпільований файл з попереднього етапу
COPY --from=builder /app/app .

# Визначаємо команду запуску
CMD ["./app"]

```
