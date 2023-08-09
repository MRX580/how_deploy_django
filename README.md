<h1>Простое руководство, как деплоить Django проект</h1>

<h2>Деплой проводился на операционной системе ubuntu 20</h2>

<h3>Подготовка всех компонентов</h3>
Первым делом нам нужно обновить пакеты линукса


    sudo apt update
    sudo apt upgrade
<strong>Устанавливаем nginx</strong>

    sudo apt install nginx

<strong>Устанавливаем gunicorn</strong>

    sudo apt install gunicorn

<strong>Устанавливаем python 3.9 + venv</strong>

    sudo apt install python3.9 python3.9-venv

С подготовкой компонентов закончили, переходим в установке и настройке

<h3>Установка проекта на сервер</h3>

Переходим в директорию /home

    cd /home


<strong>Создаем виртуальное окружение</strong>

    python3.9 -m venv venv

<strong>Переходим в него и устанавливаем все нужные зависимости</strong>

    source venv/bin/activate
    pip install gunicorn
    pip install django

<h3>Импортируем наш проект с гихаба</h3>

    git clone <url>


<h3>Настройка проекта (!)</h3>

<strong>Начнем с nginx, создадим наш конфиг с проектом</strong>

<blockquote>Очень важно не запутаться с путями, читайте внимательно чтобы не пропустить ничего</blockquote>

Для начала создадим наш файл конфигурации nginx

    sudo nano /etc/nginx/sites-available/my_django_project

<h2>КАК УКАЗЫВАТЬ ПУТЬ?</h2>

Заходим в директорию где у нас находится manage.py и пишем там:

    pwd
Получаем: <code>/home/mysite</code>

<h3>И то что получили записываем в /static/ и /media/ (!)</h3>

и записываем туда (не забываем указать your_ip):

    server {
    
        listen 80;
        server_name your_ip;
    
        location /static/ {
            root /home/mysite;
        }
      
        location /media/ {
            root /home/mysite;
        }
        location / {
            include proxy_params;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
    }


Переносим конфигурацию в рабочее пространство

    sudo ln -s /etc/nginx/sites-available/my_django_project /etc/nginx/sites-enabled/

<strong>перезагружаем nginx</strong>

    service nginx restart

Nginx настроен.

<h3>Настройка gunicorn</h3>

Идем в каталог /etc/systemd/system/

    cd /etc/systemd/system/

Создаем файл gunicorn.service

    nano gunicorn.service

И записываем туда:

    [Unit]
    Description=gunicorn daemon
    Requires=gunicorn.socket
    After=network.target
    
    [Service]
    User=root
    WorkingDirectory=/home/mysite
    ExecStart=/home/venv/bin/gunicorn --workers 5 --bind unix:/run/gunicorn.sock your_project_wsgi.wsgi:application

    [Install]
    WantedBy=multi-user.target

В WorkingDirectory записываем путь к manage.py

В ExecStart записываем виртуальное окружение с gunicorn

(!) И не забываем указать your_project_wsgi, wsgi называеться так же как вы назвали проект, или же можно найти в settings в переменной: WSGI_APPLICATION

Сохраняем и выходим, и создаем еще один файл в той же директории под названием gunicorn.socket

    nano gunicorn.socket

И записываем туда

    [Unit]
    Description=gunicorn socket
    
    [Socket]
    ListenStream=/run/gunicorn.sock
    
    [Install]
    WantedBy=sockets.target


Сохраняем и закрываем, менять ничего не нужно

Теперь нам нужно запустить сервисы gunicorn

    sudo systemctl enable gunicorn
    sudo systemctl start gunicorn
    service gunicorn restart

После всех манипуляций у вас должен заработать проект, если у вас не получилось, можно написать мне в телеграмм https://t.me/MRXlllll
