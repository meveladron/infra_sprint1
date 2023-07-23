# infra_sprint1
# Проект Kittygram

Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## Стек технологии
- Python 3.x
- node.js 9.x.x
- frontend: React
- backend: Django
- nginx
- gunicorn

## Способ установки
<br>1. Клонируем репозиторий
```
git clone git@github.com:meveladron/infra_sprint1.git
```

<br>2. Создаём и активируем виртуальное окружение
```
python3 -m venv venv
```
```
source venv/bin/activate
```

<br>3. Устанавливаем зависимости для проекта
```
pip install -r requirements.txt 
```

<br>4. Из папки backend выполняем миграции
```
cd infra_sprint1/backend/
```
```
python manage.py migrate
```

<br>5. Создаём суперпользователя
```
python manage.py createsuperuser
```

<br>6. В файле settings.py в директории infra_sprint1/backend/kittygram_backend/ в переменную ALLOWED_HOSTS добавляем локальные адреса, доменное имя или внешний IP
```
ALLOWED_HOSTS = ['127.0.0.1', '0.0.0.0', 'localhost', 'xxx.xxx.xxx.xxx']
```

<br>7. в файле settings.py изменяем значение переменной DEBUG с True на False
```
DEBUG = False
```

<br>8. Устанавливаем node.js
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

<br>9. Из директории infra_sprint1/frontend устанавливаем зависимости для frontend-приложения
```
npm i
```

<br>10. Для backend-приложения устанавливаем WSGI-сервер gunicorn
```
pip install gunicorn
```

<br>11. Для автозапуска WSGI сервера создаём файл конфигурации
```
sudo nano /etc/systemd/system/gunicorn.service
```

<br>12. В данный файл вносим настройки
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>  # (!) заменить на собственное
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 backend.wsgi

[Install]
WantedBy=multi-user.target 
```

<br>13. Запускаем созданную службу и ставим не автозапуск
```
sudo systemctl start gunicorn
```
```
sudo systemctl enable gunicorn
```

<br>14. Устанавливаем веб-сервер Nginx и запускаем его
```
sudo apt install nginx -y
```
```
sudo systemctl start nginx 
```

<br>15. Открываем порты и активируем файерволл
```
sudo ufw allow 'Nginx Full'
```
```
sudo ufw allow OpenSSH
```
```
sudo ufw enable
```

<br>16. Для корректной работы статики из директории infra_sprint1/frontend/, компилируем frontend-приложение и копируем его в системную директорию веб-сервера
```
npm run build
```
```
sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/infra_sprint1/
```

<br>17. Из директории infra_sprint1/backend/, собираем статику для админки Django и копируем её в системную директорию веб-сервера
```
infra_sprint1/backend/kittygram_backend/settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT

STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```
```
python manage.py collectstatic
```
```
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/
```

<br>18. Установите библиотеку python-dotenv в активированном виртуальном окружении
```
pip install python-dotenv
```

<br>19. В корневой директории проекта создайте файл .env
```
touch .env
```
<br>20. Откройке файлы .env в редакторе nano и укажите необъодимые переменные и их значения.
```
sudo nano .env
```
```
SECRET_KEY='token'
DEBUG=True
ALLOWED_HOSTS=<ip адрес сервера> 127.0.0.1 localhost
```
<br>21. Внесите импорты установленной библиотеки в файле settings проекта, вызовите функцию load_dotenv(), указываем значения переменных из файла .env
```
import os
from pathlib import Path

from dotenv import load_dotenv 

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.getenv('SECRET_KEY')

DEBUG = os.environ.get('DEBUG')

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS').split(' ')
```


<br>22. Создаём мамку media для медиафайлов в директории веб-сервера, изменяем права доступа
```
cd /var/www/infra_sprint1/
```
```
mkdir media
```
```
sudo chown -R <имя_пользователя> /var/www/infra_sprint1/media/
```

<br>23. Под sudo открываем файл конфигурации веб-сервера и изменяем его настройки на следующие
```
sudo nano /etc/nginx/sites-enabled/default 
```
```
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }

    location / {
    root    /var/www/infra_sprint1;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```

<br>24. Проверяем файл конфигурации веб-сервера, перезагружаем его конфиг, если нет ошибок
```
sudo nginx -t
```
```
sudo systemctl reload nginx
```

<br>25. Если необходим SSL-сертификат, то получаем его для вашего доменного имени при помощи certbot
```
sudo apt install snapd
```
```
sudo snap install core; sudo snap refresh core
```
```
sudo snap install --classic certbot
```
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```
```
sudo certbot --nginx
```
