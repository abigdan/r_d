# Lesson-6

## Підготовка образів та мережі (тека lesson6/fluent)
```
sudo docker build -t fluentd-loki .

sudo docker network create --driver bridge net_l6
```

## Запуск контейнерів та перевірка логів
```
sudo docker run -d --name fluentd --network net_l6 -v $(pwd)/fluentd.conf:/fluentd/etc/fluent.conf -p 24224:24224 -p 24224:24224/udp fluentd-loki

sudo docker run -d --name log_container busybox sh -c 'while true; do echo "Log string posted at $(date)"; sleep 2; done'

sudo docker logs log_container
sudo docker logs fluentd
```

## Налаштування loki та grafana (тека lesson6)
```
sudo docker run -d --name loki --network net_l6 -p 3100:3100 grafana/loki:latest

sudo docker run -d --name grafana  --network my_network -p 3000:3000 -e GF_SECURITY_ADMIN_PASSWORD=r0b0t -e "GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH=/etc/grafana/dashboards/default-dashboard.json" -e GF_SERVER_ROOT_URL=http://localhost:3000 -v $(pwd)/provisioning:/etc/grafana/provisioning grafana/grafana
```



# **Логування та події в Docker**

## **Методи логування в Docker**

### **Перегляд логів контейнера**

Створимо контейнер з логом:

```
docker run -d --name log_container busybox sh -c 'while true; do echo "Log string posted at $(date)"; sleep 2; done'
```

Логи контейнера можна переглянути за допомогою команди:

```
docker logs log_container
```

Або в реальному часі (поточний потік логів):

```
docker logs -f log_container
```


### **Керування сховищем логів**

Щоб уникнути переповнення диска, можна обмежити розмір файлу логів. У випадку використання драйвера `json-file`, логи зберігаються у каталозі:

```
/var/lib/docker/containers/<container_id>/<container_id>-json.log
```

Щоб визначити, який файл логів належить конкретному контейнеру, використовуйте команду:

```
docker inspect --format='{{.LogPath}}' <container_id>
```

Також можна переглянути весь вміст директорії з логами:

```
ls -lh /var/lib/docker/containers/*/*.log
```

Якщо лог-файл займає занадто багато місця, його можна очистити:

```
truncate -s 0 /var/lib/docker/containers/<container_id>/<container_id>-json.log
```

```
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 nginx
```

Ця команда обмежує лог-файл 10 МБ та зберігає 3 файли логів.

---

## Перегляд подій контейнерів

Перегляд подій у реальному часі:

```
docker events
```

Ця команда відображає такі події, як старт/стоп контейнера, перезапуск, створення томів, зміни мережі тощо.


Щоб отримати події лише для певного контейнера, використовуйте:

```
docker events --filter container=<container_id>
```

Або, наприклад, відфільтрувати події лише по типу `start`:

```
docker events --filter event=start
```

---

## Інтеграція з системами централізованого логування


#### **Тестування стандартного JSON-файлового логування**

```
docker run --rm --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 busybox echo "Test JSON log"
```

Перевірити створені логи можна командою:

```
cat /var/lib/docker/containers/$(docker ps -lq)/*/json.log
```

#### **Тестування системного журналу (syslog)**

```
docker run --rm --log-driver=syslog --log-opt syslog-address=udp://127.0.0.1:514 busybox echo "Test syslog log"
```

Логи можна перевірити за допомогою:

```
tail -f /var/log/syslog
```

#### **Тестування журналу systemd (journald)**

```
docker run --rm --log-driver=journald busybox echo "Test journald log"
```

Перегляд логів у `journald`:

```
journalctl -u docker.service --no-pager | grep "Test journald log"
```

---
### **Створення централізованої системи збору логів з використанням Fluentd та Loki**

#### **Крок 1: Створити свою мережу**

```
docker network create --driver bridge my_network
```

#### **Крок 2: Запуск Fluentd для збору логів**

##### **Створіть `Dockerfile`:**

```Dockerfile
FROM fluent/fluentd:v1.16-debian 
USER root 
RUN gem install fluent-plugin-loki 
USER fluent
```

##### **Зберіть образ:**

```
docker build -t fluentd-loki . 
```


Створимо конфігураційний файл `fluentd.conf`:

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

Тепер запустимо Fluentd:

```
docker run -d --name fluentd \
  --network my_network \
  -v $(pwd)/fluentd.conf:/fluentd/etc/fluent.conf \
  -p 24224:24224 -p 24224:24224/udp \
  fluentd-loki
```

Цей контейнер приймає логи через порт 24224.

#### **Крок 3: Перенаправлення логів log_container у Fluentd**

```
docker run -d --name log_container_fluentd \
  --network my_network \
  --log-driver=fluentd --log-opt fluentd-address=localhost:24224 \
  busybox sh -c 'while true; do echo "Fluentd test log: $(date)"; sleep 2; done'
```

Тепер `log_container_fluentd` передає свої логи у Fluentd.

#### **Крок 4: Запуск Loki для зберігання логів**

```
docker run -d --name loki --network my_network -p 3100:3100 grafana/loki:latest
```

Loki тепер доступний за `http://localhost:3100` і приймає логи від Fluentd.

#### **Крок 5: Запуск Grafana для перегляду логів**

##### **Створіть конфіг для підключення Loki**

```
mkdir -p provisioning/datasources
```

datasources.yaml
```yaml
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
```



```
docker run -d --name grafana  --network my_network -p 3000:3000 -e GF_SECURITY_ADMIN_PASSWORD=admin -e "GF_DASHBOARD_DEFAULT_HOME_DASHBOARD_PATH=/etc/grafana/dashboards/default-dashboard.json" -e GF_SERVER_ROOT_URL=http://localhost:3000 -v $(pwd)/provisioning:/etc/grafana/provisioning grafana/grafana
```

Після запуску Grafana доступна за `http://localhost:3000`. Логін/пароль за замовчуванням – `admin/admin`.

#### **Крок 6:  Перевірка логів**

1. Перейдіть у **Explore** у Grafana.
    
2. Виберіть джерело логів **Loki**.
    
3. Виконайте запит `{job="docker-logs"}`.
    
4. Ви побачите логи, які Fluentd отримав від `log_container_fluentd` та передав у Loki.

