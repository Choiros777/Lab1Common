[Шпаргалка по разметке mark down](https://gist.github.com/OlgaMurzina/a094b7bfa6f15b25c683309885d68dbb#links)
# 📝 Лабораторная работа №1 (без *)
В этот раз, мы всей командой решили подготовить минисервер в виде виртуальной машины, чтобы не ставить вторую систему на ноутбук. С первого семестра у нас остался VirtualBox. Мы создали новую виртуальную машину и установили на нее ОС Ubuntu 26.04.   

# 📝 Что мы сделали:


# 🛠️ Как это было
## День 1-й. Ставим nginx и пробуем выпустить ssl-сертификат  
### Ставим nginx  
Первым делом установили nginx.
   ```bash
    sudo apt install nginx
   ```
После установки проверяем статус службы nginx:
   ```bash
    sudo service nginx status
   ```
Все хорошо, служба работает (вот за это мы и любим Ubuntu в частности, и Linux в целом):
   ```text
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-09-28 10:30:50 UTC; 17s ago
 Invocation: 2918cc0afb4940d9ac9a5727991a90a5
       Docs: man:nginx(8)
    Process: 7691 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 7693 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 7728 (nginx)
      Tasks: 9 (limit: 6353)
     Memory: 7.3M (peak: 16.5M)
        CPU: 54ms
     CGroup: /system.slice/nginx.service
             ├─7728 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─7731 "nginx: worker process"
             ├─7732 "nginx: worker process"
             ├─7733 "nginx: worker process"
             ├─7734 "nginx: worker process"
             ├─7735 "nginx: worker process"
             ├─7736 "nginx: worker process"
             ├─7737 "nginx: worker process"
             └─7738 "nginx: worker process"

Sep 28 10:30:50 Ubuntu systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Sep 28 10:30:50 Ubuntu systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
   ```
   Посмотрим что находится в файле настроек nginx:
   ```bash
    cat /etc/nginx/nginx.conf
   ```
<details>
  <summary>Видим содержимое файла /etc/nginx/nginx.conf</summary>

   ```text
        user www-data;
        worker_processes auto;
        worker_cpu_affinity auto;
        pid /run/nginx.pid;
        error_log /var/log/nginx/error.log;
        include /etc/nginx/modules-enabled/*.conf;
        
        events {
            worker_connections 768;
            # multi_accept on;
        }
        
        http {
        
            ##
            # Basic Settings
            ##
        
            sendfile on;
            tcp_nopush on;
            types_hash_max_size 2048;
            server_tokens build; # Recommended practice is to turn this off
        
            # server_names_hash_bucket_size 64;
            # server_name_in_redirect off;
        
            include /etc/nginx/mime.types;
            default_type application/octet-stream;
        
            ##
            # SSL Settings
            ##
        
            ssl_protocols TLSv1.2 TLSv1.3; # Dropping SSLv3 (POODLE), TLS 1.0, 1.1
            ssl_prefer_server_ciphers off; # Don't force server cipher order.
        
            ##
            # Logging Settings
            ##
        
            access_log /var/log/nginx/access.log;
        
            ##
            # Gzip Settings
            ##
        
            gzip on;
        
            # gzip_vary on;
            # gzip_proxied any;
            # gzip_comp_level 6;
            # gzip_buffers 16 8k;
            # gzip_http_version 1.1;
            # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
        
            ##
            # Virtual Host Configs
            ##
        
            include /etc/nginx/conf.d/*.conf;
            include /etc/nginx/sites-enabled/*;
        }
        
        
        #mail {
        #	# See sample authentication script at:
        #	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
        #
        #	# auth_http localhost/auth.php;
        #	# pop3_capabilities "TOP" "USER";
        #	# imap_capabilities "IMAP4rev1" "UIDPLUS";
        #
        #	server {
        #		listen     localhost:110;
        #		protocol   pop3;
        #		proxy      on;
        #	}
        #
        #	server {
        #		listen     localhost:143;
        #		protocol   imap;
        #		proxy      on;
        #	}
        #}
   ```
</details>
<details>
  <summary>Пробуем посмотреть что выдаст браузер по этому url: http://localhost/</summary>

![img.png](Screenshots/welcome_nginx.png)

</details>

### Выпускаем самоподписной ssl-сертификат
Пробуем выпустить самоподписной ssl-сертификат для локальных имен: Notes.app, locallhost и локального ip 127.0.0.1:
   ```bash
    openssl req -x509 -newkey rsa:4096 \
      -keyout notes-app.key \
      -out notes-app.crt \
      -days 1095 -nodes \
      -subj "/CN=Notes.app" \
      -addext "subjectAltName=DNS:Notes.app,DNS:localhost,IP:127.0.0.1"   
  ```
В итоге получили 2 файла:
1. notes-app.key
2. notes-app.crt

<details>
  <summary>Это скриншот создания самоподписного ssl-сертификата"</summary>

![img.png](Screenshots/ssl_certcreate.png)

</details>

## День 2-й. Оживляем наш frontend по https
### Сборка frontend
Для начала надо собрать наш frontend по-взрослому командой:
   ```bash
  npm run build
  ```
Установили npm:
   ```bash
  sudo apt install npm
  ```
В папке frontend выполнили установку зависимостей:
   ```bash
  npm install
  ```
Затем в папке frontend выполнили сборку:
   ```bash
  npm run build
  ```

<details>
  <summary>ops неудачка </summary>

![img.png](Screenshots/build_fail.png)

</details>

Для экономии времени просто отключили проверку типов в файле frontend/package.json, исправив строку сборки так:
"build": "~~tsc -b &&~~ vite build", чтобы в итоге получилось:
   ```text
    ...
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "oxlint",
    "preview": "vite preview"
  }
  ...
```
Теперь сборка проходит и результат сборки находится в папке dist. Перенесли папку dist со всем содержимым в папку /var/www/Notes.app:

   ```bash
    sudo mv dist /var/www/Notes.app 
  ```
### Подключаем самоподписной ssl-сертификат к nginx
Положили файл notes-app.key в каталог /etc/ssl/private/:
   ```bash
    sudo mv notes-app.key /etc/ssl/private/notes-app.key 
  ```
Положили файл notes-app.crt в каталог /etc/ssl/certs/:
   ```bash
    sudo mv notes-app.crt /etc/ssl/certs/notes-app.crt 
  ```
Создали файл настройки нашего сервиса:
   ```bash
    sudo nano /etc/nginx/sites-available/notes-app 
  ```
![img.png](Screenshots/notes_app1.png)

Сделали симлинк в sites-enabled:
   ```bash
    sudo ln -s /etc/nginx/sites-available/notes-app  /etc/nginx/sites-enabled/notes-app 
  ```

Дали команду nginx перечитать конфигурацию и перезапуститься:
   ```bash
    sudo nginx -t
    sudo service nginx reload
   ```
Теперь в браузере по адресу https://localhost видим страницу своего frontend, согласившись открыть небезопасный сайт, т.к. браузер не доверяет самоподписному сертификату. 

## День 3-й. Настраиваем nginx на проксирование Backend
### Разбираемся с базой данных PostgreSQL на Ubuntu
Поставили PostgreSQL:
   ```bash
    sudo apt install -y postgresql postgresql-contrib    
   ```
Дальше настроим доступ на наш PostgreSQL сервер снаружи, чтобы можно было подключиться используя PgAdmin:
   ```bash
    sudo cp /etc/postgresql/18/main/postgresql.conf /etc/postgresql/18/main/postgresql.conf.bak
    sudo nano /etc/postgresql/18/main/postgresql.conf   
   ```
Нашли строку в /etc/postgresql/18/main/postgresql.conf:

   ```text
    #listen_addresses = 'localhost'
   ```
и заменили ее так:
   ```text
    listen_addresses = '*'
   ```
Разрешили подключения в pg_hba.conf
   ```bash
    sudo cp /etc/postgresql/18/main/pg_hba.conf /etc/postgresql/18/main/pg_hba.conf.bak
    sudo nano /etc/postgresql/18/main/pg_hba.conf   
   ```
Нашли строку в /etc/postgresql/18/main/pg_hba.conf:

   ```text
    host    all             all             127.0.0.1/32            scram-sha-256
   ```
и заменили ее так:
   ```text
    host    all             all             0.0.0.0/0            scram-sha-256
   ```
Заменили пароль пользователю postgres:
   ```bash
    sudo -u postgres psql
   ```
   ```sql
    ALTER USER postgres WITH PASSWORD 'passwordexample';
   ```
где 'passwordexample' дан только для примера. Реальный пароль другой и здесь опубликован не будет.

Перечитали конфигурацию:
   ```bash
    sudo systemctl reload postgresql
   ```
Теперь можем подключиться с любого компа, который видит ip-адрес нашей виртуалки к установленному на ней PostgreSQL серверу.
Подключились PgAdmin 4 и создали базу для бэкенда, как в лабе 0.
Выполнили шаги по установке бэкенда, как в лабе 0.

### Добавляем проксирование backend в конфигурацию нашего сервиса
Пытаемся доделать настройки нашего сервиса:
   ```bash
    sudo nano /etc/nginx/sites-available/notes-app 
  ```
Сейчас файл настройки нашего сайта выглядит так:

![img.png](Screenshots/notes_app2.png)

Дали команду nginx перечитать конфигурацию и перезапуститься:
   ```bash
    sudo nginx -t
    sudo service nginx reload
   ```
Теперь сервис "Наши заметки" полностью работает (если не забыть запустить хотя бы одни экземпляр нашего backend) и доступен по https, например https://127.0.0.1 или https://localhost.
Осталось настроить редирект с http на https. 

### Hастраиваем редирект с http на https
Снова открываем на редактирование файл настройки нашего сервиса, чтобы сделать редирект с http на https:
   ```bash
    sudo nano /etc/nginx/sites-available/notes-app 
  ```
Чтобы не мешал дефолтный сайт nginx отключаем его:
   ```bash
    sudo unlink /etc/nginx/sites-enabled/default 
  ```
Дали команду nginx перечитать конфигурацию и перезапуститься:
   ```bash
    sudo nginx -t
    sudo service nginx reload
   ```
Теперь сервис "Наши заметки" полностью работает (если не забыть запустить хотя бы одни экземпляр нашего backend) и доступен как по http, так и по https.

На сегодня пока все.

На текущий момент, файл настройки нашего сайта выглядит так:
![img.png](Screenshots/notes_app2.png)

## День 4-й. Почти всё, но ещё не всЁ
### Добавляем свою страницу на обработку ошибки 404 вместо стандартной
<details>
  <summary>Сейчас отображение ошибки 404 выглядит вот так</summary>

![img.png](Screenshots/404_nginx.png)

</details>

Скопировали файл Lab_1/404.html в корень нашего сайта:

```bash
    sudo cp 404.html /var/www/Notes.app
   ```

<details>
  <summary>После нашего волшебства над файлом /etc/nginx/sites-available/notes-app отображение ошибки 404 стало выглядеть красиво</summary>

![img.png](Screenshots/404_best.png)

</details>

В итоге, файл настройки сервиса "Наши заметки" выглядит так:

![img.png](Screenshots/notes_app.png)

### Настраиваем второй виртуальный сервер (второй сайт)
Эх, что-то с памятью моей стало, хорошо, что записала. Вспоминаем, что делали в первый день:
Выпустили сертификат для нашего второго сайта. Назовем домен: site2.local и добавим ещё для интереса www.site2.local:
   ```bash
    openssl req -x509 -newkey rsa:4096 \
      -keyout site2-local.key \
      -out site2-local.crt \
      -days 1095 -nodes \
      -subj "/CN=site2.local" \
      -addext "subjectAltName=DNS:site2.local,DNS:www.site2.local"   
  ```
Положили файл site2-local.key в каталог /etc/ssl/private/:
   ```bash
    sudo mv site2-local.key /etc/ssl/private/site2-local.key 
  ```
Положили файл site2-local.crt в каталог /etc/ssl/certs/:
   ```bash
    sudo mv site2-local.crt /etc/ssl/certs/site2-local.crt 
  ```
Создали папку нашего второго сайта и положили туда Lab_1/index.html и Lab_1/404.html
```bash
  sudo mkdir /var/www/site2.local
  sudo cp  Lab_1/index.html Lab_1/404.html /var/www/site2.local
  ```
```bash
  sudo cp  Lab_1/index.html Lab_1/404.html /var/www/site2.local
  ```

Создали файл настройки сайта site2.local, скопировав наш /etc/nginx/sites-available/notes-app:
   ```bash
    sudo cp /etc/nginx/sites-available/notes-app /etc/nginx/sites-available/site2-local
  ```
и поправили его так:

![img.png](Screenshots/site2-local.png)

Сделали симлинк в sites-enabled:
   ```bash
    sudo ln -s /etc/nginx/sites-available/site2-local  /etc/nginx/sites-enabled/site2-local 
  ```

Дали команду nginx перечитать конфигурацию и перезапуститься:
   ```bash
    sudo nginx -t
    sudo service nginx reload
   ```
Вставили резолвинг на site2.local и www.site2.local в /etc/hosts

![img.png](Screenshots/etc-hosts.png)

Набрали в браузере http://www.site2.local/ согласились с рисками и увидели это:

![img.png](Screenshots/site2.png)

### Настраиваем заглушку для неизвестных виртуальных серверов (сайтов)
Чтобы nginx ничего не выдал лишнего, кроме наших 2-х сайтов, делаем заглушку:

  ```bash
    sudo nano /etc/nginx/sites-available/000-catch-all.conf
  ```
Содержание файла /etc/nginx/sites-available/000-catch-all.conf:

   ```text
    # Неизвестные HTTP-имена
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
    
        server_name _;
    
        return 444;
    }
    
    # Неизвестные HTTPS-имена
    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
    
        server_name _;
    
        ssl_certificate     /etc/ssl/certs/default.crt;
        ssl_certificate_key /etc/ssl/private/default.key;
    
        return 444;
    }
   ```

Создали сертификат для https заглушки:

```bash
sudo openssl req -x509 -nodes -newkey rsa:2048 \
  -days 3650 \
  -keyout /etc/ssl/private/default.key \
  -out /etc/ssl/certs/default.crt \
  -subj "/CN=invalid.local" \
  -addext "subjectAltName=DNS:invalid.local"
```
Теперь веб-сервер не отдает ничего лишнего (разумеется после чтения конфигурации и перезапуска nginx)  
   ```bash
    sudo nginx -t
    sudo service nginx reload
   ```

## День 5-й. Ну теперь-то всё или ещё не всЁ
### Настраиваем лимиты запросов на /api/
В конфиг /etc/nginx/nginx.conf внутри http { ... }
   ```bash
    sudo nano /etc/nginx/nginx.conf
   ```
добавили:
   ```text
    # Лимит на каждый IP клиента:
    # в среднем до 5 запросов/секунду
    limit_req_zone $binary_remote_addr zone=api_per_ip:10m rate=5r/s;   
   ```
Включили лимит в location /api/ нашего сервиса:
   ```bash
    sudo nano /etc/nginx/sites-available/notes-app
   ```
Чтобы получилось так:

![img.png](Screenshots/limit_req.png)



