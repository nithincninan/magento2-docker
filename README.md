# Magento 2.4.x Docker + Xdebug(Phpstorm) + MailHog + Multiple Website + Blackfire + Redis/Redisinsight + Openserach + Rabbitmq + Opensearch dashboards

Magento 2.4.4-p13 Docker Environment

Services  : Nginx 1.26, PHP 8.1-fpm-buster, Mariadb 10.6

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
└── magento247
```

Magento 2.4.4-p13 Docker Setup:

1. Download and install docker app (windows/Mac)

    * Docker > Preferences > Resources > Advanced : at least 8 CPUs and 32.0 GB RAM
    * Local machine have alteast 32GB RAM(Recommended)    

2. Clone magento2-docker repository and Build the docker Images:

        * Goto "magento244" folder
        * docker-compose build
        * docker-compose up -d

3. Add System variable in environmental settings ```SHELL=/bin/bash``` (windows)

4. To show Running Containers use command ```docker ps```

```
λ docker ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS         PORTS                                                                                                         NAMES
2d1dc4d759b9   mailhog/mailhog:latest               "MailHog"                14 minutes ago   Up 1 second    0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp                                                                mail_244p13
0b6622e75157   nginx:1.26                           "/docker-entrypoint.…"   14 minutes ago   Up 1 second    0.0.0.0:80->80/tcp                                                                                            nginx_244p13
ef3284d69687   magento244p13-php                    "docker-php-entrypoi…"   14 minutes ago   Up 2 seconds   0.0.0.0:9000->9000/tcp                                                                                        php_244p13
be9fe2e1a0d5   redislabs/redisinsight:latest        "./docker-entry.sh n…"   14 minutes ago   Up 2 seconds   5000/tcp, 0.0.0.0:8001->8001/tcp                                                                              redisinsight_244p13
567d2fcf0195   mariadb:10.6                         "docker-entrypoint.s…"   14 minutes ago   Up 2 seconds   0.0.0.0:3306->3306/tcp                                                                                        mariadb_244p13
80cd246a10b2   opensearchproject/opensearch:2.2.0   "./opensearch-docker…"   14 minutes ago   Up 2 seconds   0.0.0.0:9200->9200/tcp, 9600/tcp, 0.0.0.0:9300->9300/tcp, 9650/tcp                                            opensearch_244p13
c423193d8afd   redis:7.2                            "docker-entrypoint.s…"   14 minutes ago   Up 2 seconds   6379/tcp                                                                                                      redis_244p13
049270ea0c8e   rabbitmq:3.9-management              "docker-entrypoint.s…"   14 minutes ago   Up 2 seconds   4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbitmq_244p13                                                                                                      redis
````

5. Install magento Instance:

```
           Connect to php container: 
           
           * docker exec -it php bash
           
           * cd /var/www/magento244
           
           * If its an existent project, clone project repository to the 'magento244', then update env.php and go to #8
           
           * If on-premesis, Install Magento Instance magento244 ( https://devdocs.magento.com/guides/v2.4/install-gde/composer.html )
           * if Cloud Instance, clone the files from https://github.com/magento/magento-cloud and use the branch accordingly.
          
          	    1. On-premesis:
          	    
          	        composer create-project --repository-url=https://repo.magento.com/ magento/project-enterprise-edition=2.4.4-p13 .
          	        * enter your Magento authentication keys
          	        
          	       Cloud: 
          	          
          	          git clone -b 2.4.4-p1-p8 https://github.com/magento/magento-cloud.git . & add below line in composer.json
          	          -->  "magento/product-enterprise-edition": "2.4.4-p13"
          	          
          	          ie,
          	          
          	          "magento/magento-cloud-metapackage": ">=2.4.4 <2.4.5",
                      "magento/composer-root-update-plugin": "^2.0.2",
                      "magento/product-enterprise-edition": "2.4.4-p13",
          		    
          		2. composer install 
          		     - Remove composer.lock file
          		     - composer install
          		     - Push the updated composer.lock file to your repository
          		
          		3. Install M2 via CLI(/var/www/magento244):
                       
                        bin/magento setup:install \
                        --base-url=http://magento244p13.loc/ \
                        --db-host=mariadb_244p13 \
                        --db-name=magento244 \
                        --db-user=mage244_user \
                        --db-password=mage244_pass \
                        --admin-firstname=admin \
                        --admin-lastname=admin \
                        --admin-email=admin@admin.com \
                        --admin-user=admin \
                        --admin-password=admin123 \
                        --language=en_US \
                        --currency=USD \
                        --timezone=America/Chicago \
                        --use-rewrites=1 \
                        --search-engine=opensearch \
                        --opensearch-host=opensearch \
                        --opensearch-index-prefix=magento244 \
                        --opensearch-port=9200 \
                        && chown -R www-data:www-data .
                           
                 3. Cross check if Opensearch is configured, if not update the below setting in app/etc/env.php:
                             
                        'system' => [
                            'default' => [
                                'catalog' => [
                                    'search' => [
                                        'engine' => 'opensearch',
                                        'opensearch_server_hostname' => 'opensearch_244p13',
                                        'opensearch_server_port' => '9200',
                                        'opensearch_index_prefix' => 'magento244',
                                    ]
                                ]
                            ]
                        ],
                        
                      Rabbitmq:
                      
                        'queue' => [
                            'amqp' => [
                                'host' => 'rabbitmq_244p13',
                                'port' => '5672',
                                'user' => 'guest',
                                'password' => 'guest',
                                'virtualhost' => '/'
                            ],
                            'consumers_wait_for_messages' => 1
                        ],
                           
                4. Enable Developer Mode: php bin/magento deploy:mode:set developer
                5. Make sure cache enabled : php bin/magento cache:enable
                6. Configure Redis default/page caching (if required)
                     php bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=redis --cache-backend-redis-port=6379 --cache-backend-redis-db=0
                     bin/magento setup:config:set --page-cache=redis --page-cache-redis-server=redis --page-cache-redis-db=1
         
                7. Configure Redis for session storage (if required)
                    php bin/magento setup:config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-port=6379 --session-save-redis-log-level=4 --session-save-redis-db=2
     
                8. Run the cli commands:
                    * php bin/magento setup:upgrade
                    * php bin/magento setup:di:compile
                    * php -dmemory_limit=6G bin/magento setup:static-content:deploy -f
                    * chmod -R 777 var pub generated app/etc/*

                    Note: File ownership and permissions refer https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/prerequisites/file-system/configure-permissions
          		    
```        

6. Configure your hosts file: 127.0.0.1 magento244p13.loc
   1. In windows:-  c:\Windows\System32\Drivers\etc\hosts.
   2. Mac/Ubuntu:-  /etc/hosts

7. Open http://magento244p13.loc/ 

8. Open http://magento244p13.loc/admin/

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

10. To Setup Muiltiple Website:

 10.1) Setup Websites, Stores and Storeviews in Magento Admin <br />
 
 10.2) Modify nginx config file in docker(docker/nginx/default.conf): <br />
 
         
         map $http_host $MAGE_RUN_CODE {
            magento244p13-second.loc second_website_code;
            magento244p13.loc base;
         }
         upstream fastcgi_backend {
              server php:9000;
          }
          server {
              listen 80;
              server_name magento244p13.loc magento244p13-second.loc;
              set $MAGE_ROOT /var/www/magento244;
              set $MAGE_MODE developer;
              fastcgi_param  MAGE_MODE $MAGE_MODE;

              #Setup multi websites
              set $MAGE_RUN_TYPE website;
              fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
              fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;

              include /var/www/magento244/nginx.conf.sample;
              error_log /var/log/nginx/error.log;
              access_log /var/log/nginx/access.log;
          }
          
       
  10.3). Configure your hosts file: 127.0.0.1 magento244p13.loc magento244p13-second.loc <br />
            1. In windows:-  c:\Windows\System32\Drivers\etc\hosts. <br />
            2. Mac/Ubuntu:-  /etc/hosts     <br />
  
  10.4). Modifty nginx.conf.sample(magento244/nginx.conf.sample)<br />
  
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
  
 11. Integrating Blackfire.io with Docker Compose:
  
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
                

 12. Integrate Redis/Redisinsight in Docker Compose:
   
     1. Add Redis to your network:
            
         docker-compose.yaml
   
            redis:
              container_name: redis
              image: redis:latest
            
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

13. Integrate Rabbitmq in Docker Compose:
    
    ```
    rabbitmq:
        container_name: rabbitmq_244p13
        image: rabbitmq:3.9-management
        ports:
            - "15672:15672"
            - "5672:5672"
    ```

    Open app/etc/env.php and add/update the corresponding data:
    ```
     'queue' => [
        'amqp' => [
            'host' => 'rabbitmq_244p13',
            'port' => '5672',
            'user' => 'guest',
            'password' => 'guest',
            'virtualhost' => '/'
        ],
        'consumers_wait_for_messages' => 1
    ],


14. Integrate Opensearch in Docker Compose:

    ```
    opensearch:
        image: opensearchproject/opensearch:2.2.0
        container_name: opensearch_244p13
        ports:
            - "9200:9200"
            - "9300:9300"
        environment:
            "discovery.type": "single-node"
            OPENSEARCH_INITIAL_ADMIN_PASSWORD: ${OPENSEARCH_INITIAL_ADMIN_PASSWORD}
            DISABLE_SECURITY_PLUGIN: true
        networks:
            - magento_network
    ```

    Open app/etc/env.php and add/update the corresponding data:
    ```
    'system' => [
        'default' => [
            'catalog' => [
                'search' => [
                    'engine' => 'opensearch',
                    'opensearch_server_hostname' => 'opensearch_244p13',
                    'opensearch_server_port' => '9200',
                    'opensearch_index_prefix' => 'magento244'
                ]
            ]
        ]
    ],  
