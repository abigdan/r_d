# Lesson-5

## **Підготовка контейнерів**
Використаний образ з попереднього завдання
```
nginx з додаванням наступних RUN:
RUN apt-get install -y lynx              --- замість curl для віддаленого підключення до вебсерверу
RUN apt-get install -y net-tools         --- ifconfig, netstat...
RUN apt-get install -y iputils-ping      --- ping

А ось додати curl не вдалось - постійно якась помилка з недоступністю якогось IP, де дзеркало цього пакету. Намагався використовувати VPN в різних країнах - не допомогло, різні IP були недоступні (хоча це не рф чи білорусь, а належність Британії). На хостовій системі проблем не виникало або включення VPN допомагало, а ось при створенні образу докером - постійно з curl (в Інтернеті подібні історії про докер відомі, але методи лікування накшталт apt-get update && не допомогли, а редагувати список джерел для мене нереально при створенні образу)
```
### Створення контейнерів
```
sudo docker run -d --name ngnx1-bridge -p 8081:80 nginx-l5
sudo docker run -d --name ngnx2-bridge -p 8082:80 nginx-l5
sudo docker run -d --name ngnx3-host –network host nginx-l5
sudo docker run -d --name ngnx4-none –network none -p 8084:80 nginx-l5

sudo docker network create -d macvlan --subnet=192.168.100.0/24 --gateway=192.168.100.1 -o parent=enp0s3 macvlan
sudo docker run -d --name ngnx5-macvlan –network macvlan -p 8085:80 nginx-l5
```
### Додаткові команди
```
sudo netstat -tulnp | grep :80

git add lesson5
git commit -m "5th_commit"
git push
```
### Результат команди: ip a
```
sudo docker exec -it ngnx1-bridge ip a > ngnx1-bridge.txt
sudo docker exec -it ngnx2-bridge ip a > ngnx2-bridge.txt
sudo docker exec -it ngnx3-host ip a > ngnx3-host.txt
sudo docker exec -it ngnx4-none ip a > ngnx4-none.txt
sudo docker exec -it ngnx5-macvlan ip a > ngnx5-macvlan.txt
ip a > host.txt
```

Як видно, перелік мережевих адаптерів контейнера ngnx3-host (--network host) абсолютно ідентичний до переліку безпосередньо на хостовій системі
Для мережеі none відповідний контейнер ngnx4-none має лише localhost (127.0.0.1) адаптер
Налаштування адаптерів контейнерів ngnx1-bridge та nginx2-bridge відрізняються IP адресою (172.17.0.2 та 172.17.0.3)

## **Доступність контейнерів**
Очевидно, що контейнер з типом мережі NONE не зможе "достукатися" до будь-чого, як і будь-хто не зможе "достукатися" до цього конетейнера

### Приклади **успішної** доступності з __ХОСТОВОЇ__ системи
#### ngnx1-bridge
```
ping 172.17.0.2
lynx http://172.17.0.2:80
lynx http://localhost:8081
```

#### ngnx1-bridge
```
ping 172.17.0.3
lynx http://172.17.0.3:80
lynx http://localhost:8082


Доступність з bridge:
sudo docker exec -it ngnx1-bridge /bin/bash

Вдалося "достукатися" до ngnx2-bridge (172.17.0.3) та відкрити сторінку веб-серверу, а також пропінкувати хостові IP.

Але зайти на 80-й порт хосту чи контейнеру ngnx3-host не пощастило ( 

## **Корисні команди**
```
sudo docker tag nginx-from-ubuntu abigdan/nginx-from-ubuntu:0.2
sudo docker push abigdan/nginx-from-nginx:0.1
```
