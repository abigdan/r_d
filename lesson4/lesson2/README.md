# Lesson-2

Спочатку треба прописати домени які будуть використовуватись для тестів у hosts файл хостової системі.
Треба використовувати адресу віртуальної машини. Під linux це має приблизно такий вигляд:
```
192.168.8.36	gitea.docker.loc
192.168.8.36	gitea.k8s.loc
```

## Розгортання Gitea за допомогою Docker compose

 1. перейдіть до папки docker та перевірте наявність та зміст `docker-compose.yml`
 2. для старту с виводом логу на екран: 
    ```shell
    sudo docker compose up 
    ```
    для старту в фоновому режимі:
    ```shell
    sudo docker compose up -d
    ```
    після вдалого старту gitea буде доступна за адресою віртуальної машини на порту 3000, наприклад <http://gitea.docker.loc:3000>
 3. для припинення:
    ```shell
    sudo docker compose down
    ```

## Розгортання Gitea за допомогою MicroK8S

 1. Post-install steps:
    за бажанням можна налаштувати права запуску від свого користувача без sudo
    ```shell
    sudo usermod -a -G microk8s $USER
    sudo chown -R $USER ~/.kube
    newgrp microk8s
    ```
 2. підключаємо потрібні аддони:
    ```shell
    sudo microk8s enable hostpath-storage
    sudo microk8s enable ingress
    sudo microk8s enable cert-manager
    sudo microk8s status
    ```
 3. додаємо namespace для gitea та перевіряємо нявні:
    ```shell
    sudo microk8s.kubectl create namespace gitea
    sudo microk8s.kubectl get ns
    ```
 4. перейдіть до папки k8s та застосуйте конфігурацію issuer для автоматичного випуску сертифікатів (перевірити зміст манифеста можна в файлі issuer.yaml):
    ```shell
    sudo microk8s.kubectl apply -f issuer.yaml
    ```
 5. далі треба створити сертифікат параметри якого можна подивитись в маніфесті cert.yaml:
    ```shell
    sudo microk8s.kubectl apply -f cert.yaml -n gitea
    ```
 6. додаємо та оновлюємо helm репозиторій для gitea:
    ```shell
    sudo microk8s.helm repo add gitea-charts https://dl.gitea.io/charts/
    sudo microk8s.helm repo update
    ```
 7. встановлюємо gitea у відповідний неймспейс, з кастомними параметрами для встановлення в microK8S можна ознайомитись в маніфесті values.yaml:
    ```shell
    sudo microk8s.helm install gitea gitea-charts/gitea -f ./values.yaml --namespace gitea
    ```
    після вдалого старту gitea буде доступна за доменною адресою яку ві прописали у hosts файл та маніфести сертифіката и helm, порт стандартний 80 або 443 (насправді завжди буде переадресація на захищений 443), наприклад <http://gitea.k8s.loc>
 8. перевірити стан сутностей gitea у кластері microK8S можна за допомогою наступних команд:
    ```shell
    sudo microk8s.kubectl get pods -n gitea
    sudo microk8s.kubectl get pvc -n gitea
    sudo microk8s.kubectl get ingress -n gitea
    sudo microk8s.kubectl get service -n gitea
    ```
 9. ознайомитись з параметрами values.yaml за замовчанням та перевірити усі можливі налаштування можна на офіційній сторінці з heln чартом:
    <https://artifacthub.io/packages/helm/gitea/gitea>
 10. якшо ви вирішили поексперементувати з налаштуваннями в values.yaml, зробіть копію цього файла та внесіть потрібні зміни у копію, після чого застосуйте зміни (зверніть увагу, треба вказувати ім'я нового маніфеста):
    
    ```shell
    sudo microk8s.helm upgrade gitea gitea-charts/gitea -f ./new_values.yaml --namespace gitea
    ```
 11. видалення gitea з кластеру:
 
   ```shell
   sudo microk8s.helm uninstall gitea --namespace gitea
   ```
