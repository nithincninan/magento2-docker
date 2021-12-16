FROM php:7.4-fpm-buster

RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/privet && echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/privet && echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/privet

RUN apt-get update

RUN apt-get -qq install -y \
    apt-utils \
    git \
    curl \
    unzip \
    vim \
    nano \
    libzip-dev \
    libmcrypt-dev \
    libicu-dev \
    libxml2-dev libxslt1-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    mariadb-client \
    mailutils \
    golang-go \
    && docker-php-ext-configure hash --with-mhash \
    && docker-php-ext-configure gd --with-jpeg --with-freetype \
    && docker-php-ext-install -j$(nproc) intl xsl gd zip pdo_mysql opcache soap bcmath json iconv pcntl sockets

RUN docker-php-ext-install mysqli

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

# Configuring php.ini
RUN apt-get install -y vim \
    && cd /usr/local/etc/php \
    && cp php.ini-development php.ini

# Memory Limit, Execution time and opcache settings - php.ini
RUN echo "memory_limit = 4G" > /usr/local/etc/php/conf.d/memory-limit.ini
RUN echo "max_execution_time = 1800" > /usr/local/etc/php/conf.d/max-execution-time.ini
RUN echo "opcache.enable = 1" > /usr/local/etc/php/conf.d/opcache.ini
RUN echo "opcache.save_comments = 1" > /usr/local/etc/php/conf.d/opcache.ini
RUN echo "upload_max_filesize = 1024M" > /usr/local/etc/php/conf.d/upload-max-size.ini

# Install xdebug extension
RUN pecl install xdebug && docker-php-ext-enable xdebug \
    && echo "xdebug.mode=develop,debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.start_with_request = yes" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install Composer
RUN cd /usr/src && curl -sS http://getcomposer.org/installer | php
RUN cd /usr/src && mv composer.phar /usr/bin/composer

# Configure MailHog
RUN apt-get install --no-install-recommends --assume-yes --quiet ca-certificates curl git &&\
    rm -rf /var/lib/apt/lists/*
RUN curl -Lsf 'https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz' | tar -C '/usr/local' -xvzf -
ENV PATH /usr/local/go/bin:$PATH
RUN go get github.com/mailhog/mhsendmail
RUN cp /root/go/bin/mhsendmail /usr/bin/mhsendmail
RUN echo 'sendmail_path = /usr/bin/mhsendmail --smtp-addr mailhog:1025' > /usr/local/etc/php/conf.d/mail.ini

#Xdiff install
#RUN curl -L -o /tmp/xdiff.tar.gz "http://www.xmailserver.org/libxdiff-0.23.tar.gz" \
#    && mkdir -p /usr/src/xdiff \
#    && tar -C /usr/src/xdiff -zxvf /tmp/xdiff.tar.gz --strip 1 \
#    && rm /tmp/xdiff.tar.gz \
#    && cd /usr/src/xdiff \
#    && ./configure  && make && make install \
#    \
#    &&  pecl install xdiff && docker-php-ext-enable xdiff

#RUN apt-get install tree
