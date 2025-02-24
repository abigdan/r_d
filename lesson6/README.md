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

## Копіювання скріншоту Grafana на віртуалку
```
scp -P 6022 "Grafana screenshot.png" rd@127.0.0.1:/home/rd/r_d/lesson6
```
