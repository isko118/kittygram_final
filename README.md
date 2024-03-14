# Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

### Описание проекта
Пользователи могут регистрироваться, загружать фотографии с описанием и смотреть питомцев других пользователей.

### Установка
1. Склонируйте репозиторий на свой компьютер:
    ```
    git clone git@github.com:1emd/kittygram_final.git
    ```
2. Создайте файл `.env` и заполните его своими данными. Все необходимые переменные перечислены в файле `.env.example`, находящемся в корневой директории проекта.

### Создание Docker-образов

1. Вместо `USERNAME` вставьте свой логин на DockerHub:

    ```
    cd frontend
    docker build -t USERNAME/kittygram_frontend .
    cd ../backend
    docker build -t USERNAME/kittygram_backend .
    cd ../nginx
    docker build -t USERNAME/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```
    docker push USERNAME/kittygram_frontend
    docker push USERNAME/kittygram_backend
    docker push USERNAME/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```
    ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME USERNAME@SERVER_IP_ADDRESS 
    ```

2. Создайте на сервере директорию `kittygram`:

    ```
    mkdir kittygram
    ```

3. Установите Docker Compose на сервер:

    ```
    sudo apt update
    sudo apt install curl
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    sudo apt install docker-compose
    ```

4. Скопируйте файлы `docker-compose.production.yml` и `.env` в директорию `kittygram/` на сервере:

    ```
    scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml USERNAME@SERVER_IP_ADDRESS:/home/USERNAME/kittygram/docker-compose.production.yml
    ```
    
    Где:
    - `PATH_TO_SSH_KEY` - путь к файлу с вашим SSH-ключом
    - `SSH_KEY_NAME` - имя файла с вашим SSH-ключом
    - `USERNAME` - ваше имя пользователя на сервере
    - `SERVER_IP_ADDRESS` - IP-адрес вашего сервера

5. Запустите Docker Compose в режиме демона:

    ```
    sudo docker-compose -f /home/USERNAME/kittygram/docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в `/backend_static/static/`:

    ```
    sudo docker-compose -f /home/USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py migrate
    sudo docker-compose -f /home/USERNAME/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker-compose -f /home/USERNAME/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. Откройте конфигурационный файл Nginx в редакторе nano:

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки `location` в секции `server`:

    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте правильность конфигурации Nginx:

    ```
    sudo nginx -t
    ```

    Если вы получаете следующий ответ, значит, ошибок нет:

    ```
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Перезапустите Nginx:

    ```
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан и находится в директории:

    ```
    kittygram/.github/workflows/main.yml
    ```

2. Для адаптации его к вашему серверу добавьте секреты в GitHub Actions:

    ```
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # IP-адрес сервера
    USER                           # имя пользователя
    SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # пароль для SSH-ключа
    TELEGRAM_TO                    # ID вашего телеграм-аккаунта
    TELEGRAM_TOKEN                 # токен вашего бота
    ```

### Технологии
[![Python](https://img.shields.io/badge/-Python-blue?style=flat&logo=python&logoColor=yellow)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-092E20?style=flat&logo=django&logoColor=white)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/-Django%20REST%20Framework-FF1709?style=flat&logo=django&logoColor=white)](https://www.django-rest-framework.org/
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-336791?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)

### CI/CD

[![Workflow Status](https://img.shields.io/github/workflow/status/isko118/kittygram_final/CI?label=Workflow&logo=github&style=flat-square)](https://github.com/isko118/kittygram/actions)


### Автор
[Iskander Nasyrov](https://github.com/isko118) [![GitHub](https://img.shields.io/badge/-Iskander_Nasyrov-181717?style=flat&logo=github&logoColor=white)](https://github.com/isko118)

