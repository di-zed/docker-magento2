## How is it work?

Build Docker:
```code
docker-compose build
```

Build Up:
```code
docker-compose up -d
```

Stop Docker:
```code
docker-compose stop
```

Stop and Remove all Docker containers:
```code
docker stop $(docker ps -a -q) && sudo docker rm $(docker ps -a -q)
```

## Containers

### Nginx

- Host: nginx
- Port: 80
- SSL: 443
- Folder with configs: ./volumes/etc/nginx/conf.d
- Folder with projects: ./volumes/var/www
- Folder with SSL certificates: ./volumes/etc/ssl

Enter the container:
```code
docker-compose exec nginx /bin/bash
```

### Varnish

- Host: varnish
- Port: 6081

Enter the container:
```code
docker-compose exec varnish /bin/bash
```

Log to file:
```code
varnishncsa -a -w /var/log/varnish/access.log -D -P /var/run/varnishncsa.pid
```

### PHP 7.4

- Host: php74
- Port: 9000

Enter the container:
```code
docker-compose exec php74 /bin/bash
```

### PHP 8.1

- Host: php81
- Port: 9000

Enter the container:
```code
docker-compose exec php81 /bin/bash
```

### Blackfire

- https://blackfire.io/docs/up-and-running/docker
- https://maxhopei.github.io/2019/06/26/Integrating-Blackfire-io-with-Docker-Compose/

### MySQL

- Host: localhost
- Port: 3366
- User: root
- Password: 123456
- Folder for dumps: ./upload

Enter the container:
```code
docker-compose exec mysql /bin/bash
```

Upload dump:
```code
mysql -u root -p database < /upload/dump.sql
```

### PhpMyAdmin

- URL: http://localhost:8090/, http://pma.loc/

Enter the container:
```code
docker-compose exec phpmyadmin /bin/bash
```

### ElasticSearch v7

- Host: elasticsearch7
- Port: 9200

Enter the container:
```code
docker-compose exec elasticsearch7 /bin/bash
```

### Redis

- Host: redis
- Port: 6379

Enter the container:
```code
docker-compose exec redis /bin/bash
```

### RabbitMQ

- Host: rabbitmq
- Port: 5672
- URL: http://localhost:15672/, http://rabbitmq.loc/

Enter the container:
```code
docker-compose exec rabbitmq /bin/bash
```

### MailCatcher

- Host: mailcatcher
- Port: 1025
- URL: http://localhost:1080/, http://mailcatcher.loc/

### Node v6

Enter the container:
```code
docker-compose run node6 /bin/bash
```

### Node v12

Enter the container:
```code
docker-compose run node12 /bin/bash
```

## Configurations

### Xdebug

UFW, allow network if needed:
```code
sudo ufw allow in from 192.168.220.0/28 to any port 9000 comment xDebug9000
```

#### PhpStorm

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

#### SSL certificate

Generate:
```code
openssl req -newkey rsa:2048 -sha256 -keyout project.key -nodes -x509 -days 365 -out project.crt
```
- Common Name = project-domain.loc