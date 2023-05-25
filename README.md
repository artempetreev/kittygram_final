# Kittygram

## Использованные технологии

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white)![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white)![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E)![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)


### Функционал сайта 

После регистрации у вас есть возможность:

- Редактировать, добавлять, удалять, просматривать котов;
- Добавлять новые / присваивать существующие достижения; 
- Просматривать чужих котов и их достижения;

### Демонстрация сайта
Посмотреть рабочую версию сайта переходите по ссылке: https://kittypetreev.hopto.org/

## Установка 

1. Форкните репозиторий, клонируйте его, и перейдите в него:

    ```bash
    git clone ...
    ```
    ```bash
    cd kittygram_final
    ```
2. Создайте файл .env и заполните его своими данными:

    ```bash
    POSTGRES_DB=...
    POSTGRES_USER=...
    POSTGRES_PASSWORD=...
    DB_HOST=...
    DB_PORT=...
    ```

### Создание Docker-образов

1. Для создания Docker-образов последовательно выполните следующие команды, замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram

    ```bash
    mkdir kittygram
    ```

3. Для установки docker compose на сервер, поочередно выполните следующие команды:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. Скопируйте на сервер в директорию kittygram/ файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```

5. Переходите в директорию kittygram/ и запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:

    ```bash
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
    ```

9. Проверьте работоспособность конфига и перезапустите Nginx:

    ```bash
    sudo nginx -t 
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории

    ```bash
    kittygram/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_PASSWORD - пароль от аккаунта DockerHub
    DOCKER_USERNAME - логин DockerHub
    HOST - IP адресс сервера
    USER - логин на сервере
    SSH_KEY - SSH ключ
    SSH_PASSPHRASE - пароль от сервера
    TELEGRAM_TO - ваш Telegram ID
    TELEGRAM_TOKEN - токена вашего Telegram бота
    ```