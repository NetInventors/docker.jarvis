FROM php:7.2-apache
RUN apt-get update && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libxslt1-dev \
        libcurl4-gnutls-dev \
        libmagickwand-6.q16-dev \
        libssl-dev \
        libc6-dev \
        zlib1g-dev \
        libmemcached-dev \
        libicu-dev \
        libc-client-dev \
        libkrb5-dev \
        unzip \
        wget \
        mysql-client \
        imagemagick \
        ghostscript \
        curl \
        python \
        build-essential \
        git \
        ca-certificates \
        gcc \
        make \
        autoconf \
        pkg-config \
        cron \
        supervisor \
    && ln -s /usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16/MagickWand-config /usr/bin/ \
    && docker-php-ext-install \
        iconv \
        opcache \
        zip \
        curl \
        pdo \
        pdo_mysql \
        mysqli \
        mbstring \
        soap \
        ftp \
        intl \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && rm -rf /var/lib/apt/lists/*

RUN adduser --disabled-password --home /var/www --shell /usr/sbin/nologin --gecos '' --ingroup www-data netinventors
RUN mkdir -p /var/www/.opcache && chown netinventors:www-data /var/www/.opcache

RUN pecl channel-update pecl.php.net \
    && pecl install mcrypt-1.0.1 \
    && echo "extension=mcrypt.so" > /usr/local/etc/php/conf.d/ext-mcrypt.ini \
    && pecl install imagick \
    && echo "extension=imagick.so" > /usr/local/etc/php/conf.d/ext-imagick.ini \
    && pecl install apcu \
    && echo "extension=apcu.so" > /usr/local/etc/php/conf.d/ext-apcu.ini \
    && pecl install memcached \
    && echo "extension=memcached.so" > /usr/local/etc/php/conf.d/ext-memcached.ini \
    && pecl install redis \
    && echo "extension=redis.so" > /usr/local/etc/php/conf.d/ext-redis.ini \
    && pecl install xdebug \
    && echo "zend_extension=xdebug.so" > /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.remote_port=\"9000\"" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.remote_host=\"172.17.0.1\"" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.profiler_enable=0" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.profiler_output_dir=\"/tmp\"" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.max_nesting_level=400" >> /usr/local/etc/php/conf.d/ext-xdebug.ini \
    && echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/ext-xdebug.ini

COPY src/php/php.ini /usr/local/etc/php/php.ini
COPY src/php/ext-opcache.ini /usr/local/etc/php/conf.d/ext-opcache.ini

# IONCUBE LOADER
ADD http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz /tmp/
RUN tar xvfC /tmp/ioncube_loaders_lin_x86-64.tar.gz /tmp/ \
    && rm /tmp/ioncube_loaders_lin_x86-64.tar.gz \
    && mkdir -p /usr/local/ioncube \
    && cp /tmp/ioncube/ioncube_loader_lin_7.2.so /usr/local/ioncube \
    && rm -rf /tmp/ioncube \
    && echo "zend_extension = /usr/local/ioncube/ioncube_loader_lin_7.2.so" > /usr/local/etc/php/conf.d/00-ioncube.ini

# APACHE
COPY src/apache/apache.conf /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite headers && a2ensite 000-default.conf \
    && mkdir -p /usr/local/etc/php/apache2 \
    && cp -R /usr/local/etc/php/conf.d /usr/local/etc/php/apache2 \
    && echo 'export PHP_INI_SCAN_DIR=/usr/local/etc/php/apache2/conf.d' >> /etc/apache2/envvars
ENV APACHE_RUN_USER netinventors

# CRON
ADD src/cron/application /etc/cron.d/application
RUN chmod 0644 /etc/cron.d/application

# COMPOSER
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && mkdir -p /var/www/.composer && chown netinventors:www-data /var/www/.composer  \
    && mkdir -p /var/www/.npm && chown netinventors:www-data /var/www/.npm  \
    && mkdir -p /var/www/.config && chown netinventors:www-data /var/www/.config \
    && rm /usr/local/etc/php/conf.d/ext-apcu.ini \
    && rm /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    && rm /usr/local/etc/php/conf.d/ext-xdebug.ini

# SUPERVISOR
COPY src/supervisor/supervisor.conf /etc/supervisor/
COPY src/supervisor/conf.d/ /etc/supervisor/conf.d/

VOLUME ["/var/www/html", "/var/log/app"]

COPY src/entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 80

