# Magento 2 Docker + Xdebug + Phpstorm

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
1. Download and install docker app (windows/Mac)
2. Build the docker Images: ```docker-compose up --build```
3. Install Magento Instance in folder ```magento23```
4. Configure your hosts file: 127.0.0.1 magento23.loc
5. Open magento23.loc
