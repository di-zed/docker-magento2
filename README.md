# DiZed Docker Compose for the multiple Magento 2 projects

###### It can be used for other projects with similar environments too.

## Structure

**The most interesting folders:**

1. **./images** The folder with custom Docker Images.
2. **./local** Folder for local changes. It can be convenient to use it together with the docker-compose.local.yml file.
3. **./mounted** A folder for exchanging files between the PC and the Docker container.
4. **./volumes** The folder with Docker Volumes in Linux structure.
   1. **./volumes/etc/nginx/conf.d** Nginx configuration files for projects.
   2. **./volumes/etc/ssl** SSL certificates for projects.
   3. **./volumes/var/log** Logs from different services.
   4. **./volumes/var/www** Nginx projects.

## Setup

1. Copy the *./.env.sample* file to the *./.env*. Check it and edit some parameters if needed.
2. Copy the *./volumes/root/bash_history/mysql.sample* file to the *./volumes/root/bash_history/mysql* for saving command history in the MySQL container.
3. Copy the *./volumes/root/bash_history/php.sample* file to the *./volumes/root/bash_history/php* for saving command history in the PHP containers.
4. **Optional.** Copy the *./volumes/root/.blackfire.ini.sample* file to the *./volumes/root/.blackfire.ini* for configuration Blackfire if you have plans to use it.
5. **Optional.** Copy the *./docker-compose.local.yml.sample* file to the *./docker-compose.local.yml* and edit it, if you need some changes in the Docker configurations locally.
6. **Optional.** If your want to have local domains for PhpMyAdmin, RabbitMQ, and MailCatcher, you need to add the data to you local */etc/hosts* file, like:
    ```code
    127.0.0.1 pma.loc
    127.0.0.1 rabbitmq.loc
    127.0.0.1 mailcatcher.loc
    ```

## Process

**Docker Build**
```code
docker-compose build
```

**Docker Up**
```code
docker-compose up -d
```

Docker Up, if you use some changes for the local environment:

```code
docker-compose -f docker-compose.yml -f docker-compose.local.yml up -d
```

**Docker Stop**
```code
docker-compose stop
```

## Project setup

1. Copy the dir with the project to the *./volumes/var/www/project-name* folder.
2. Copy the database dump to the *./mounted* folder.
3. Go inside the MySQL container, create the database, and copy your dump to the database.
    ```code
    mysql -u root -p project_database < /home/mounted/project_dump.sql
    ```
4. Generate an SSL certificate if you have plans to use Varnish or need it for other aims.
    ```code
    openssl req -newkey rsa:2048 -sha256 -keyout project_name.key -nodes -x509 -days 365 -out project_name.crt
    ```
   - **Common Name** = project-domain.loc
   - Move the certificates to the *./volumes/etc/ssl/certs* and *./volumes/etc/ssl/private* folders.
5. Go to the *./volumes/etc/nginx/conf.d* and create the Nginx configuration for the project.
    ```nginx
    server {
        listen 80;
        server_name project-domain.loc www.project-domain.loc;
        set $MAGE_ROOT /var/www/project-name;
        set $FASTCGI_PASS "php81:9000";
        access_log /var/log/nginx/project_name.access.log;
        error_log /var/log/nginx/project_name.error.log;
        include conf.d/samples/magento243.conf;
    }
    
    server {
        listen 443 ssl http2;
        server_name project-domain.loc www.project-domain.loc;
        ssl_certificate /etc/ssl/docker/certs/project_name.crt;
        ssl_certificate_key /etc/ssl/docker/private/project_name.key;
        access_log /var/log/nginx/project_name.access.log;
        error_log /var/log/nginx/project_name.error.log;
        location / {
            proxy_pass http://varnish:6081;
            proxy_buffer_size 8k;
            proxy_busy_buffers_size 8k;
            proxy_buffers 1024 8k;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-Port 443;
            proxy_set_header Host $host;
            proxy_read_timeout 1200s;
        }
    }
    ```
6. Add the project host to your local */etc/hosts* file.
    ```code
    127.0.0.1 project-domain.loc
    ```
7. Add your own configurations to the project like connection to the DB, Redis, RabbitMQ, ElasticSearch, MailCatcher, build styles, etc.
8. **Optional.** Add the configurations to the *docker-compose.local.yml* if you use it.
9. Restart the Docker Compose or separate containers.
    ```code
    docker-compose stop && docker-compose -f docker-compose.yml -f docker-compose.local.yml up -d
    ```

## xDebug configuration

### PhpStorm

**Add Docker server (if it wasn't added yet)**

- File -> Settings -> Build, Execution, Deployment -> Docker -> [+]
- Name: Docker
- Connect to Docker daemon with: Unix socket

**Add external interpreter**

- File -> Settings -> PHP
- PHP language level: 7.4
- CLI Interpreter: php74, or add new: [...]
  - [+]
    - From: Docker Compose
    - Server: Docker
    - Configurations file(s): /.../docker-default/docker-compose.yml
    - Service: php74
  - Lifecycle: Connect to existent container

**Add PHP server**

- File -> Settings -> PHP -> Servers -> [+]
- Name: Docker
- Host: project-domain.loc
- Port: 80
- Debugger: Xdebug
- Use path mappings: yes
  - Absolute path on the server:
    - root = /var/www/project
    - pub = /var/www/project/pub (if needed)

**Add run configuration**

- Main menu -> Run -> Edit configurations
- [+] -> PHP Web Page
- Name: project-domain.loc
- Server: Docker

### Linux (optional)

If debug doesn't work, makes sense to try the next two steps:

1. Change xDebug host in PHP container settings.
    ```code
    ;for PHP v7.4:
    xdebug.remote_host=192.168.220.1
    ;for PHP v8.1
    xdebug.client_host=192.168.220.1
    ```
2. UFW, allow network if needed (in terminal):
    ```code
    sudo ufw allow in from 192.168.220.0/28 to any port 9000 comment xDebug9000
    ```

## Containers

### Nginx

- Host: nginx
- Port: 80
- SSL: 443
- Folder with configs: ./volumes/etc/nginx/conf.d
- Folder with projects: ./volumes/var/www
- Folder with SSL certificates: ./volumes/etc/ssl

```code
docker-compose exec nginx /bin/bash
```

### Varnish

- Host: varnish
- Port: 6081

```code
docker-compose exec varnish /bin/bash
```

### PHP 7.4

- Host: php74
- Port: 9000

```code
docker-compose exec php74 /bin/bash
```

### PHP 8.1

- Host: php81
- Port: 9000

```code
docker-compose exec php81 /bin/bash
```

### MySQL

- Host: mysql
- Port: 3306
- User: {see .env file}
- Password: {see .env file}
- Folder for dumps: ./mounted (/home/mounted)

```code
docker-compose exec mysql /bin/bash
```

### PhpMyAdmin

- URL: http://localhost:8090/, http://pma.loc/
- Username: {see .env file}
- Password: {see .env file}

```code
docker-compose exec phpmyadmin /bin/bash
```

### ElasticSearch v7

- Host: elasticsearch7
- Port: 9200

```code
docker-compose exec elasticsearch7 /bin/bash
```

### Redis

- Host: redis
- Port: 6379

```code
docker-compose exec redis /bin/bash
```

### RabbitMQ

- Host: rabbitmq
- Port: 5672
- URL: http://localhost:15672/, http://rabbitmq.loc/
- Username: {see .env file}
- Password: {see .env file}

```code
docker-compose exec rabbitmq /bin/bash
```

### MailCatcher

- Host: mailcatcher
- Port: 1025
- URL: http://localhost:1080/, http://mailcatcher.loc/

### Node v6

```code
docker-compose run node6 /bin/bash
```

### Node v12

```code
docker-compose run node12 /bin/bash
```

### Blackfire (optional, see docker-compose.local.yml.sample)

- https://blackfire.io/docs/up-and-running/docker
- https://maxhopei.github.io/2019/06/26/Integrating-Blackfire-io-with-Docker-Compose/