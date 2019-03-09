FROM php:7.2-fpm
 
RUN docker-php-ext-install mysqli

#Installing Php extensions
RUN apt-get update && apt-get install -y \
    apt-utils \
    git \
    curl \
    unzip \
    libmcrypt-dev \
    libicu-dev \
    libxml2-dev libxslt1-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    mysql-client \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure hash --with-mhash \
    && docker-php-ext-install -j$(nproc) intl xsl gd zip pdo_mysql opcache soap bcmath json iconv

# Configuring php.ini
RUN apt-get install -y vim \
    && cd /usr/local/etc/php \
    && cp php.ini-development php.ini

# Install xdebug extension
RUN pecl install xdebug-2.6.1 && docker-php-ext-enable xdebug \
    #&& echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.default_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port = 9002" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini




