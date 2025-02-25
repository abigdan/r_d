# Lesson-7

# **Docker Compose** YAML

docker-compose.yaml
```
version: '1.1'

# сервіси / контейнери
services:

  # контейнер накопичення логів для візуалізації у Grafana
  loki:
    # образ контейнера
    image: grafana/loki:latest
    # список портів (звідку:куди)
    ports:
      - "3100:3100"
    # список мереж
    networks:
      - net_l6

  # контейнер з драйвером централізованого збору логів Fluent
  fluentd-loki:
    # параметри створення образа
    build:
      context: ./fluent
      dockerfile: dockerfile
    # списки портів (звідки:куди)
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    # підключення тому - передача файлу конфігурації в контейнер в режимі read-only
    volumes:
      - ./fluent/fluentd.conf:/fluentd/etc/fluent.conf:ro
    # список мереж
    networks:
      - net_l6
    # список залежних сервісів (тобто ці сервіси/контейнери мають бути зібрані та запущені раніше, ніж поточний сервіс/контейнер)
    depends_on: 
      - loki

  # контейнер зі створення штучних подій на основі busybox
  log_container:
    image: busybox
    command: "sh -c 'while true; do echo \"Fluentd test log: $(date)\"; sleep 2; done'"
    # список мереж
    networks:
      - net_l6
    # передача подій до fluentd
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: log_container.logs
    # список залежних сервісів (тобто ці сервіси/контейнери мають бути зібрані та запущені раніше, ніж поточний сервіс/контейнер)
    depends_on:
      - fluentd-loki

  # контейнер з grafana для візуалізації та аналітики даних з джерела loki
  grafana:
    image: grafana/grafana
    # підключення тому - передача файлу конфігурації в контейнер в режимі read-only
    volumes:
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
    # список мереж
    networks:
      - net_l6
    # списки портів (звідки:куди)
    ports:
      - "8088:3000"
    # перелік змінних, що зберігаються у файлі .env
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${gf_pass}
      - GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH=${gf_path}
      - GF_SERVER_ROOT_URL=${gf_url}
    # список залежних сервісів (тобто ці сервіси/контейнери мають бути зібрані та запущені раніше, ніж поточний сервіс/контейнер)
    depends_on:
      - loki

# налаштування мережі
networks:
  net_l6:
    driver: bridge
```




## Опис ключових елементів `**docker-compose.yml**`

- `version` – версія формату файлу `docker-compose.yml`.
    
- `services` – список сервісів, які будуть створені.
    
- `image` – образ контейнера, який буде використаний.
    
- `build` – вказує шлях до Dockerfile для створення кастомного образу.
    
- `ports` – проброс портів між хост-системою та контейнером (`"8080:80"` означає, що порт 8080 на хості буде перенаправлений на порт 80 контейнера).
    
- `volumes` – монтування локальних директорій або файлів у контейнер (`"./html:/usr/share/nginx/html"`).
    
- `environment` – змінні середовища, які будуть передані у контейнер.
    
- `depends_on` – визначає залежності між сервісами (наприклад, один сервіс буде запущений після іншого).
    
- `networks` – дозволяє вказати кастомні мережі, в яких працюють сервіси.
    
- `restart` – політика перезапуску контейнерів (`always`, `unless-stopped`, `no`, `on-failure`).
    
- `command` – дозволяє перевизначити команду запуску контейнера.
    
- `healthcheck` – перевіряє стан сервісу і визначає, коли він працює стабільно.
    
- `secrets` – дозволяє передавати секретні дані у контейнер (наприклад, паролі або токени).
    
- `logging` – налаштування драйвера логування (`json-file`, `syslog`, `fluentd` тощо).
    
- `extra_hosts` – дозволяє додавати кастомні DNS-записи у контейнер.
    
- `deploy` – параметри розгортання для Docker Swarm (наприклад, кількість реплік).


## Управління багатоконтейнерними додатками

### **Запуск сервісів**

```
docker compose up -d
```

Ця команда створює та запускає всі сервіси у фоновому режимі.

### **Перегляд працюючих контейнерів**

```
docker compose ps
```

### **Зупинка сервісів**

```
docker compose down
```

Ця команда зупиняє всі сервіси та видаляє контейнери.

### **Перезапуск сервісів**

```
docker compose restart
```

### **Логування**

```
docker compose logs -f
```

---

## Приклади використання Docker Compose

### **Запуск веб-додатку (WordPress + MySQL)**

```
version: '3.8'
services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: user
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: rootpassword
```

Цей файл створює два сервіси – WordPress і MySQL, які працюють разом.

### **Використання змінних середовища**

Для зберігання змінних середовища можна використовувати `.env` файл:

```
version: '3.8'
services:
  app:
    image: myapp
    environment:
      - APP_ENV=${APP_ENV}
```

Файл `.env`:

```
APP_ENV=production
```


### **Створення централізованої системи збору логів з використанням Fluentd та Loki**


Dockerfile
```
FROM fluent/fluentd:v1.16-debian 
USER root 
RUN gem install fluent-plugin-loki 
USER fluent
```

fluentd.conf
```
<source>
  @type forward
</source>

<match **>
  @type loki
  endpoint_url "http://loki:3100"
  labels {"job":"docker-logs"}
</match>
```

datasources.yaml
```
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
```

docker-compose.yaml
```
version: '3.8'

services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    networks:
      - my_network

  fluentd-loki:
    build:
      context: ./fluentd-loki
      dockerfile: Dockerfile
    ports:
      - "24224:24224"
      - "24224:24224/udp"
    volumes:
      - ./fluentd-loki/fluentd.conf:/fluentd/etc/fluent.conf:ro
    networks:
      - my_network
    depends_on: 
      - loki

  log_container_fluentd:
    image: busybox
    command: "sh -c 'while true; do echo \"Fluentd test log: $(date)\"; sleep 2; done'"
    networks:
      - my_network
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: log_container_fluentd.logs
    depends_on:
      - fluentd-loki

  grafana:
    image: grafana/grafana
    volumes:
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
    networks:
      - my_network
    ports:
      - "80:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD}
      - GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH=${GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH}
      - GF_SERVER_ROOT_URL=${GF_SERVER_ROOT_URL}
    depends_on:
      - loki

networks:
  my_network:
    driver: bridge
```

.env
```
GF_SECURITY_ADMIN_PASSWORD=${SUPERSECRETSENSITIVEDATA}
GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH=/etc/grafana/dashboards/default-dashboard.json
GF_SERVER_ROOT_URL=http://localhost
```
