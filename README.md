# Magento 2.4.2 Docker + Xdebug(Phpstorm) + MailHog + Multiple Website + Blackfire + Redis/Redisinsight + Elasticsearch + Rabbitmq

Magento 2.4.2 Docker Environment

Services  : Nginx 1.14, PHP 7.4-fpm-buster, Mariadb 10.4

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

Magento 2.4 Docker Setup:

1. Download and install docker app (windows/Mac)
2. Build the docker Images: ```docker-compose up --build```
3. Add System variable in environmental settings ```SHELL=/bin/bash``` (windows) 
4. To show Running Containers use command ```docker ps```

```
λ docker ps
   CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                            NAMES
   3ad5fb3428c6        nginx:1.14-alpine        "nginx -g 'daemon of…"   7 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp                               nginx
   409c95c7d215        mysql:5.7                "docker-entrypoint.s…"   7 seconds ago       Up 5 seconds        0.0.0.0:3306->3306/tcp, 33060/tcp                mariadb
   79d53ca16356        mailhog/mailhog:latest   "MailHog"                7 seconds ago       Up 5 seconds        0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp   mail
   3badaa76979b        docker_php               "docker-php-entrypoi…"   8 seconds ago       Up 6 seconds        0.0.0.0:9000->9000/tcp                           php
   0c8e1f030202        elasticsearch:7.6.2      "/usr/local/bin/dock…"   9 days ago          Up 26 hours         0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp                                                 elasticsearch
   3497490e7765   rabbitmq:3-management         "docker-entrypoint.s…"   26 hours ago        Up 26 hours         4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, :::5672->5672/tcp, 15671/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp, :::15672->15672/tcp   rabbitmq
````

5. To Run on a Running Container(php): 
      ```docker exec -it php bash```
6. Install Magento Instance in folder ```magento242```
7. Configure your hosts file: 127.0.0.1 magento24.loc 
   1. In windows:-  c:\Windows\System32\Drivers\etc\hosts.
   2. Mac/Ubuntu:-  /etc/hosts
8. Open magento242.loc
9. MailHog:- You are able to see all the emails from docker(Magento Instance) on http://localhost:8025/

     Eg: php -r "\$from = \$to = 'youremail@gmail.com'; \$x = mail(\$to, 'subject'.time(), 'Hello World', 'From: '. \$from); var_dump(\$x);"
              o/p - bool(true)

```
Note: While installing the Mailhog, if we get the below error:
"ERROR: Get https://registry-1.docker.io/v2/mailhog/mailhog/manifests/latest: unauthorized: incorrect username or password"

Try first:
$ docker logout
and then pull again. As it is public repo you shouldn't need to login
```

# 10. To Setup Muiltiple Website:

 10.1) Setup Websites, Stores and Storeviews in Magento Admin <br />
 10.2) Modify nginx config file in docker(docker/nginx/default.conf): <br />
 
         
         map $http_host $MAGE_RUN_CODE {
            magento242-second.loc second_website_code;
            magento242.loc base;
         }
         upstream fastcgi_backend {
              server php:9000;
          }
          server {
              listen 80;
              server_name magento242.loc magento242-second.loc;
              set $MAGE_ROOT /var/www/magento242;
              set $MAGE_MODE developer;
              fastcgi_param  MAGE_MODE $MAGE_MODE;

              #Setup multi websites
              set $MAGE_RUN_TYPE website;
              fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
              fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;

              include /var/www/magento242/nginx.conf.sample;
              error_log /var/log/nginx/error.log;
              access_log /var/log/nginx/access.log;
          }
          
       
  10.3). Configure your hosts file: 127.0.0.1 magento242.loc magento242-second.loc <br />
            1. In windows:-  c:\Windows\System32\Drivers\etc\hosts. <br />
            2. Mac/Ubuntu:-  /etc/hosts     <br />
  
  10.4). Modifty nginx.conf.sample(magento242/nginx.conf.sample)<br />
  
   PHP entry point:- Add the below lines before include statement:   
   
    # START - Multisite customization
    fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
    fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;
    # END - Multisite customization
   
 ie, 

   ```
    # PHP entry point for main application
   location ~ ^/(index|get|static|errors/report|errors/404|errors/503|health_check)\.php$ {
       try_files $uri =404;
       fastcgi_pass   fastcgi_backend;
       fastcgi_buffers 1024 4k;

       fastcgi_param  PHP_FLAG  "session.auto_start=off \n suhosin.session.cryptua=off";
       fastcgi_param  PHP_VALUE "memory_limit=756M \n max_execution_time=18000";
       fastcgi_read_timeout 600s;
       fastcgi_connect_timeout 600s;

       fastcgi_index  index.php;
       fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
       # START - Multisite customization
       fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
       fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;
       # END - Multisite customization
       include        fastcgi_params;
   }
   ```
# 11. Integrating Blackfire.io with Docker Compose:
  
        Official Documentation: https://blackfire.io/docs/integrations/docker/index
  
  11.1. Pre-requisites:
  
    1. Add a Blackfire Account: https://blackfire.io/ (Blackfire stores all profiles on their server)
    
        >>>> You can see credentials Server ID, Server Token, Client ID and Client Token in https://blackfire.io/my/settings/credentials
        Chrome: https://blackfire.io/docs/integrations/chrome
        Firefox: https://blackfire.io/docs/integrations/firefox
        
   11.2. Integrate Blackfire with your Docker Compose:
   
    1. Add Blackfire Agent to your network:
            
         docker-compose.yaml
            
            blackfire:
                    container_name: blackfire_vs
                    image: blackfire/blackfire
                    ports: ["8707"]
                    environment:
                        # Exposes BLACKFIRE_* environment variables from the host
                        BLACKFIRE_SERVER_ID: ### Add server id ####
                        BLACKFIRE_SERVER_TOKEN: ### Add server token ###
                        BLACKFIRE_CLIENT_ID: ### Add client id ###
                        BLACKFIRE_CLIENT_TOKEN: ### Add client Token ###
  
    2. Add Blackfire PHP Probe and CLI tool to your application container && Point your PHP Probe to your Agent (blackfire.ini)
    
         Dockerfile:
         
            RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
                && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
                && mkdir -p /tmp/blackfire \
                && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
                && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so \
                && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
                && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz
            
            RUN mkdir -p /tmp/blackfire \
                && curl -A "Docker" -L https://blackfire.io/api/v1/releases/client/linux_static/amd64 | tar zxp -C /tmp/blackfire \
                && mv /tmp/blackfire/blackfire /usr/bin/blackfire \
                && rm -Rf /tmp/blackfire
                

# 12. Integrate Redis/Redisinsight in Docker Compose:
   
     1. Add Redis to your network:
            
         docker-compose.yaml
   
            redis:
              container_name: redis
              #image: redis:latest
              image: redis:5.0.5
            
            redisinsight:
              container_name: redisinsight
              image: redislabs/redisinsight:latest
              ports:
                  - 8001:8001
    

   Using Redis for session and cache
   Open ./magento/app/etc/env.php and add/update the corresponding data:

   ```...
       'session' => [
           'save' => 'redis',
           'redis' => [
               'host' => 'redis',
               'port' => '6379',
               'password' => '',
               'timeout' => '2.5',
               'persistent_identifier' => '',
               'database' => '2',
               'compression_threshold' => '2048',
               'compression_library' => 'gzip',
               'log_level' => '1',
               'max_concurrency' => '6',
               'break_after_frontend' => '5',
               'break_after_adminhtml' => '30',
               'first_lifetime' => '600',
               'bot_first_lifetime' => '60',
               'bot_lifetime' => '7200',
               'disable_locking' => '0',
               'min_lifetime' => '60',
               'max_lifetime' => '2592000'
           ]
       ],
   ...
       'cache' => [
           'frontend' => [
               'default' => [
                   'backend' => 'Cm_Cache_Backend_Redis',
                   'backend_options' => [
                       'server' => 'redis',
                       'database' => '0',
                       'port' => '6379'
                   ]
               ]
           ]
       ]
   ...
   ```

 
