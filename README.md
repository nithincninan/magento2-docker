# Magento 2.4.x Docker + Xdebug(Phpstorm) + MailHog + Multiple Website + Blackfire + Redis/Redisinsight + Elasticsearch + Rabbitmq

Magento 2.4.x Docker Environment

Services  : Nginx 1.14, PHP 7.4-fpm-buster, Mariadb 10.4

Tree
```
├── CHANGELOG.md
├── README.md
├── docker
│   ├── nginx
│   │   ├── Dockerfile
│   │   ├── default.conf
│   │   └── m2-config
│   │       └── nginx.conf
│   └── php
│       ├── Dockerfile
│       └── www.conf
├── docker-compose.dev.yml
├── docker-compose.yml
└── magento24
```

Magento 2.4.x Docker Setup:

1. Download and install docker app (windows/Mac)

    * Docker > Preferences > Resources > Advanced : at least 4 or 5 CPUs and 8.0 GB RAM
    * Local machine have alteast 16GB RAM(Recommended)    

2. Clone magento2-docker repository and Build the docker Images:

        * Goto "magento2-docker" folder and and create "magento24" folder(mkdir magento24)
        * docker-compose build
        * docker-compose up -d

3. Add System variable in environmental settings ```SHELL=/bin/bash``` (windows)

4. To show Running Containers use command ```docker ps```

```
λ docker ps
   9edaa3a3fd13   nginx:1.14-alpine               "nginx -g 'daemon of…"   12 minutes ago   Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp                                                                                                                     nginx
   8e5ce3425639   mailhog/mailhog:latest          "MailHog"                12 minutes ago   Up 12 minutes       0.0.0.0:1025->1025/tcp, :::1025->1025/tcp, 0.0.0.0:8025->8025/tcp, :::8025->8025/tcp                                                                  mail
   24173dd2faf6   magento2-docker_php             "docker-php-entrypoi…"   12 minutes ago   Up 12 minutes       0.0.0.0:9000->9000/tcp, :::9000->9000/tcp                                                                                                             php
   9552f52c3ce8   redis:latest                    "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes       6379/tcp                                                                                                                                              redis
   42756510ba9b   mariadb:10.4                    "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes       0.0.0.0:3306->3306/tcp, :::3306->3306/tcp                                                                                                             mariadb
   ee97a9094860   rabbitmq:3-management           "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes       4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, :::5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp, :::15672->15672/tcp   rabbitmq
   6944566615b7   elasticsearch:7.6.2             "/usr/local/bin/dock…"   12 minutes ago   Up 12 minutes       0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp                                                                  elasticsearch
   7dab612ad6ec   redislabs/redisinsight:latest   "bash ./docker-entry…"   12 minutes ago   Up 12 minutes       0.0.0.0:8001->8001/tcp, :::8001->8001/tcp                                                                                                             redisinsight
````

5. Install magento Instance:

```
           Connect to php container: 
           
           * docker exec -it php bash
           
           * cd /var/www/magento24
           
           * If its an existent project, clone project repository to the 'magento24', then update env.php and go to #8
           
           * Install Magento Instance in magento24 ( https://devdocs.magento.com/guides/v2.4/install-gde/composer.html )
          
          	    1. composer create-project --repository-url=https://repo.magento.com/ magento/project-enterprise-edition=2.4.2 .
          		    * enter your Magento authentication keys
          		    
          		2. Install M2 via CLI(/var/www/magento24):
                       
                       php bin/magento setup:install \
                               --db-host=mariadb \
                               --db-name=mage24_db \
                               --db-user=mage24_user \
                               --db-password=mage24_pass \
                               --base-url=http://magento24.loc/ \
                               --backend-frontname=admin \
                               --admin-user=admin \
                               --admin-password=admin123 \
                               --admin-email=nithincninan@gmail.com \
                               --admin-firstname=nithin \
                               --admin-lastname=ninan \
                               --language=en_US \
                               --currency=USD \
                               --timezone=America/Chicago \
                               --use-rewrites=1 \
                               --skip-db-validation \
                               --search-engine=elasticsearch7 \
                               --elasticsearch-host=elasticsearch \
                               --elasticsearch-port=9200 \
                           && chown -R www-data:www-data .         
                           
                 3. Cross check if ES is configured, if not update the below setting in app/etc/env.php:
                             
                        'system' => [
                                'default' => [
                                    'catalog' => [
                                        'search' => [
                                            'engine' => 'elasticsearch7',
                                            'elasticsearch7_server_hostname' => 'elasticsearch',
                                            'elasticsearch7_server_port' => '9200',
                                            'elasticsearch7_index_prefix' => 'magento24_index'
                                        ]
                                    ]
                                ]
                            ],
                           
                4. Enable Developer Mode: php bin/magento deploy:mode:set developer
                5. Make sure cache enabled : php bin/magento cache:enable
                6. Configure Redis default/page caching
                     php bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=redis --cache-backend-redis-port=6379 --cache-backend-redis-db=0
                     bin/magento setup:config:set --page-cache=redis --page-cache-redis-server=redis --page-cache-redis-db=1
         
                7. Configure Redis for session storage
                    php bin/magento setup:config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-port=6379 --session-save-redis-log-level=4 --session-save-redis-db=2
     
                8. Run the cli commands:
                    * php bin/magento setup:upgrade
                    * php bin/magento setup:di:compile
                    * php -dmemory_limit=6G bin/magento setup:static-content:deploy -f
                    * chown -R www-data:www-data .
          		    
```        

6. Configure your hosts file: 127.0.0.1 magento24.loc 
   1. In windows:-  c:\Windows\System32\Drivers\etc\hosts.
   2. Mac/Ubuntu:-  /etc/hosts

7. Open http://magento24.loc/ 

8. Open http://magento24.loc/admin/

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
            magento24-second.loc second_website_code;
            magento24.loc base;
         }
         upstream fastcgi_backend {
              server php:9000;
          }
          server {
              listen 80;
              server_name magento24.loc magento24-second.loc;
              set $MAGE_ROOT /var/www/magento24;
              set $MAGE_MODE developer;
              fastcgi_param  MAGE_MODE $MAGE_MODE;

              #Setup multi websites
              set $MAGE_RUN_TYPE website;
              fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
              fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;

              include /var/www/magento24/nginx.conf.sample;
              error_log /var/log/nginx/error.log;
              access_log /var/log/nginx/access.log;
          }
          
       
  10.3). Configure your hosts file: 127.0.0.1 magento24.loc magento24-second.loc <br />
            1. In windows:-  c:\Windows\System32\Drivers\etc\hosts. <br />
            2. Mac/Ubuntu:-  /etc/hosts     <br />
  
  10.4). Modifty nginx.conf.sample(magento24/nginx.conf.sample)<br />
  
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
    

   Using Redis for session/cache
    -  app/etc/env.php

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
                   'log_level' => '4',
                   'max_concurrency' => '6',
                   'break_after_frontend' => '5',
                   'break_after_adminhtml' => '30',
                   'first_lifetime' => '600',
                   'bot_first_lifetime' => '60',
                   'bot_lifetime' => '7200',
                   'disable_locking' => '0',
                   'min_lifetime' => '60',
                   'max_lifetime' => '2592000',
                   'sentinel_master' => '',
                   'sentinel_servers' => '',
                   'sentinel_connect_retries' => '5',
                   'sentinel_verify_master' => '0'
               ]
           ],
   ...
       'cache' => [
               'frontend' => [
                   'default' => [
                       'id_prefix' => 'a47_',
                       'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
                       'backend_options' => [
                           'server' => 'redis',
                           'database' => '0',
                           'port' => '6379',
                           'password' => '',
                           'compress_data' => '1',
                           'compression_lib' => ''
                       ]
                   ],
                   'page_cache' => [
                       'id_prefix' => 'a47_',
                       'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
                       'backend_options' => [
                           'server' => 'redis',
                           'database' => '1',
                           'port' => '6379',
                           'password' => '',
                           'compress_data' => '0',
                           'compression_lib' => ''
                       ]
                   ]
               ],
               'allow_parallel_generation' => false
           ],
   ...
   ```
# 13. Integrate Rabbitmq in Docker Compose:
    
    ```
    rabbitmq:
        container_name: rabbitmq
        image: rabbitmq:3-management
        ports:
            - "15672:15672"
            - "5672:5672"
    ```

    Open app/etc/env.php and add/update the corresponding data:
    ```
     'queue' => [
        'amqp' => [
            'host' => 'rabbitmq',
            'user' => 'mdc_user',
            'exchange_id' => 'mdc',
            'password' => 'mdc_pass',
            'port' => '5672',
            'virtualhost' => '/'
        ],
        'consumers_wait_for_messages' => 0
    ],

# 14. Integrate Elasticsearch in Docker Compose:

    ```
    elasticsearch:
        image: elasticsearch:7.6.2
        container_name: elasticsearch
        ports:
            - "9200:9200"
            - "9300:9300"
        environment:
            "discovery.type": "single-node"
    ```

    Open app/etc/env.php and add/update the corresponding data:
    ```
     'system' => [
        'default' => [
            'catalog' => [
                'search' => [
                    'engine' => 'elasticsearch7',
                    'elasticsearch7_server_hostname' => 'elasticsearch',
                    'elasticsearch7_server_port' => '9200',
                    'elasticsearch7_index_prefix' => 'magento2_index'
                ]
            ]
        ]
    ],

**For Performace tunning(docker):** (optional - either use #2 or #3)

```
           1. Computer, Cores & RAM :
           
                * Docker > Preferences > Resources > Advanced : at least 4 or 5 CPUs and 8.0 GB RAM
                * Local machine have alteast 16GB RAM(Recommended)
                
           2. Use “delegated”/"cached" Volume Mounts for the files what is necessary for:
           
                * Setup docker-compose.dev.yaml [ which will sync only app/composer files (Use after #1-#7) ]:
                
                   * docker-compose -f docker-compose.dev.yml up -d
                   * docker cp magento24 php:/var/www/ (move all m2 files to php container)
                   * Enable the below options(docker-compose.dev.yml) and run : docker-compose -f docker-compose.dev.yml up -d
                    - ./magento24/app:/var/www/magento24/app:delegated
                    - ./magento24/composer.json:/var/www/magento24/composer.json:delegated
                    - ./magento24/composer.lock:/var/www/magento24/composer.lock:delegated
                   * docker exec -it php bash then goto cd /var/www/magento24 and Run "chown -R www-data:www-data ."
                        
                        
                * Make sure modifying the files except (app/composer files) should be sync using "docker cp" command:
                
                ie, * once CLI completes(setup:upgrade / di:compile / content:deploy).
                    * Go to local machine DIR - ({{LOCALHOST-DIR}}/magento2-docker/magento24) and create sync_24.sh
                   (sycn generated/pub-static directory from continer to local host)
                   
                        1. vi sync_24.sh and update:
                   
                            rm -rf pub\static\*
                            docker cp php:/var/www/magento24/pub/static pub/
                            rm -rf generated\code\*
                            docker cp php:/var/www/magento24/generated/code generated/
                    
                        2. Run sync_24.sh(from localhost)
                   
                        3. docker exec -it php bash then goto cd /var/www/magento24 and Run "chown -R www-data:www-data ."
                
           
           3. Use unison docker sync for performance tunning.  
```