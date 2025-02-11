# Lesson-3

## **Базові команди**
Docker вже був за інструкцією з сайту встановлений в попередньому завданні
```
sudo apt-get install lynx
mkdir lesson3
cd lesson3
nano Dockerfile
nano index.html
sudo docker build -t my-nginx .
sudo docker run -d -p 8080:80 my-nginx
lynx http://localhost:8080/
```
```
git add lesson3
git commit -m "2nd_commit"
git push
```
## **Корисні команди**
```
sudo docker ps
sudo docker ps -a
```
```
sudo docker run -d --name my-nginx -p 8080:80 nginx
```
Якщо забути додати ключ --name, тоді доводилося лише за ID видаляти/зупиняти контейнер
```
sudo docker logs my-nginx
sudo docker inspect my-nginx
```
sudo docker stop my-nginx
sudo docker start my-nginx
sudo docker rm my-nginx
```
sudo docker images
sudo docker rmi my-nginx

sudo docker rmi -f my-nginx
```
коли контейнер не видалив, а намагався видалити образ, доводилося форсити
```
    
