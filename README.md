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

```
λ docker ps
   CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                            NAMES
   3ad5fb3428c6        nginx:1.14-alpine        "nginx -g 'daemon of…"   7 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp                               nginx
   409c95c7d215        mysql:5.7                "docker-entrypoint.s…"   7 seconds ago       Up 5 seconds        0.0.0.0:3306->3306/tcp, 33060/tcp                mysql
   79d53ca16356        mailhog/mailhog:latest   "MailHog"                7 seconds ago       Up 5 seconds        0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp   mail
   3badaa76979b        docker_php               "docker-php-entrypoi…"   8 seconds ago       Up 6 seconds        0.0.0.0:9000->9000/tcp                           php
````

5. To Run on a Running Container(php): 
      ```docker exec -it php bash```
6. Install Magento Instance in folder ```magento23```
7. Configure your hosts file: 127.0.0.1 magento23.loc 
   1. In windows:-  c:\Windows\System32\Drivers\etc\hosts.
   2. Mac/Ubuntu:-  /etc/hosts
8. Open magento23.loc

```
Note: While installing the Mailhog, if we get the below error:
"ERROR: Get https://registry-1.docker.io/v2/mailhog/mailhog/manifests/latest: unauthorized: incorrect username or password"

Try first:
$ docker logout
and then pull again. As it is public repo you shouldn't need to login
```

