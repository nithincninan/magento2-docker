# Magento 2.3 Docker + Xdebug + Phpstorm

Magento 2.3 Docker Environment

Services  : Nginx 1.14, PHP 7.2-fpm, MySQL 5.7

Tree
```
├── docker
│   ├── nginx
│       └── Dockerfile
│   │   └── default.conf
│   └── php
│       └── Dockerfile
│       └── www.conf
├── docker-compose.yml
├── magento23
│   ├── nginx.conf.sample
└── Readme.md
```

Magento 2.3 Docker Setup:

1. Download and install docker app (windows/Mac)
2. Build the docker Images: ```docker-compose up --build```
3. Add environmental path ```SHELL=/bin/bash``` (windows)
4. To show Running Containers use command ```docker ps```

```$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
56825dc47ad1        mysql:5.7           "docker-entrypoint.s…"   2 days ago          Up 2 days           0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
1bdebcad9e44        nginx:1.14-alpine   "nginx -g 'daemon of…"   2 days ago          Up 2 days           0.0.0.0:80->80/tcp                  nginx
06cd42a19e23        docker_php          "docker-php-entrypoi…"   2 days ago          Up 27 hours         0.0.0.0:9000->9000/tcp              php
````

5. To Run on a Running Container(php): 
      ```docker exec -it php bash```
6. Install Magento Instance in folder ```magento23```
7. Configure your hosts file: 127.0.0.1 magento23.loc
8. Open magento23.loc
