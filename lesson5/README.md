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

git add lesson5
git commit -m "5th_comit"
git push
```
Результат команди: ip a
```
sudo docker exec -it ngnx1-bridge ip a > ngnx1-bridge.txt
sudo docker exec -it ngnx2-bridge ip a > ngnx2-bridge.txt
sudo docker exec -it ngnx3-host ip a > ngnx3-host.txt
sudo docker exec -it ngnx4-none ip a > ngnx4-none.txt
ip a > host.txt

Як видно, перелік мережевих адаптерів контейнера ngnx3-host (--network host) абсолютно ідентичний до переліку безпосередньо на хостовій системі
Для мережеі none відповідний контейнер ngnx4-none має лише localhost (127.0.0.1) адаптер
Налаштування адаптерів контейнерів ngnx1-bridge та nginx2-bridge відрізняються IP адресою (172.17.0.2 та 172.17.0.3)

```
apt-get install iputils-ping

## **Корисні команди**
```
sudo docker tag nginx-from-ubuntu abigdan/nginx-from-ubuntu:0.2
sudo docker push abigdan/nginx-from-nginx:0.1
```
