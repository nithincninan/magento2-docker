FROM php:7.2-fpm
 
RUN docker-php-ext-install mysqli

#Installing Php extensions
RUN apt-get update && apt-get install -y \
    apt-utils \
    git \
    curl \
    unzip \
    vim \
    nano \
    libmcrypt-dev \
    libicu-dev \
    libxml2-dev libxslt1-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    mariadb-client \
    msmtp \
    mailutils \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure hash --with-mhash \
    && docker-php-ext-install -j$(nproc) intl xsl gd zip pdo_mysql opcache soap bcmath json iconv pcntl sockets

# Configuring php.ini
RUN apt-get install -y vim \
    && cd /usr/local/etc/php \
    && cp php.ini-development php.ini

# Memory Limit, Execution time and opcache settings - php.ini
RUN echo "memory_limit = 2G" > /usr/local/etc/php/conf.d/memory-limit.ini
RUN echo "max_execution_time = 1800" > /usr/local/etc/php/conf.d/max-execution-time.ini
RUN echo "opcache.enable = 1" > /usr/local/etc/php/conf.d/opcache.ini
RUN echo "opcache.save_comments = 1" > /usr/local/etc/php/conf.d/opcache.ini
RUN echo "upload_max_filesize = 1024M" > /usr/local/etc/php/conf.d/upload-max-size.ini

# Install xdebug extension
RUN pecl install xdebug-2.6.1 && docker-php-ext-enable xdebug \
    #&& echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.default_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port = 9002" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install Composer
RUN cd /usr/src && curl -sS http://getcomposer.org/installer | php
RUN cd /usr/src && mv composer.phar /usr/bin/composer

# Configure MailHog
#RUN echo "mailhub=mail:1025" >> /etc/msmtp/msmtp.conf \
#    && echo 'sendmail_path = "/usr/sbin/msmtp -t"' > /usr/local/etc/php/conf.d/mail.ini

#Xdiff install - Magento ECG Code Audit tool
RUN curl -L -o /tmp/xdiff.tar.gz "http://www.xmailserver.org/libxdiff-0.23.tar.gz" \
    && mkdir -p /usr/src/xdiff \
    && tar -C /usr/src/xdiff -zxvf /tmp/xdiff.tar.gz --strip 1 \
    && rm /tmp/xdiff.tar.gz \
    && cd /usr/src/xdiff \
    && ./configure  && make && make install \
    \
    &&  pecl install xdiff && docker-php-ext-enable xdiff

RUN apt-get install tree
