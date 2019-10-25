FROM php:7.2-fpm

ENV TIMEZONE Asia/Shanghai

# 添加源
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone


RUN apt-get update && pecl channel-update https://pecl.php.net/channel.xml


# init
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
    curl \
    wget \
    zip \
    unzip \
    vim \
    git \
    locales \
    procps \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev \
    libmemcached-dev \
    libtool \
    libicu-dev \
    libxml2 \
    libxml2-dev \
    libgmp3-dev \
    libevent-dev \
    libzip-dev \
    zlib1g-dev \
    libldap2-dev \
    libbz2-dev \
    libjpeg62-turbo-dev \
    libtidy-dev \
    libsqlite3-dev \
    libsqlite3-0 \
    tmux \
    keychain \
    supervisor \
    && mkdir -p /etc/supervisord.d/log \
    && mkdir -p /etc/supervisord.d/conf \
    && rm -rf /var/lib/apt/lists/*

RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8


RUN apt-get install -y libbz2-dev \
    && docker-php-ext-install bz2

# zip
RUN docker-php-ext-configure zip --with-libzip && docker-php-ext-install zip


# PDO mysql
RUN docker-php-ext-install pdo_mysql

# phpredis
RUN pecl install redis
RUN docker-php-ext-enable redis 

# swoole
RUN pecl bundle swoole && cd swoole && phpize && ./configure --enable-async-redis --enable-mysqlnd --enable-openssl && make && make install
RUN docker-php-ext-enable swoole 

# mongodb
RUN pecl install mongodb
RUN docker-php-ext-enable mongodb

# sqlite
RUN docker-php-ext-install pdo_sqlite

# MySQLi
RUN docker-php-ext-install mysqli

# BC Math
RUN docker-php-ext-install bcmath

# XSL
RUN apt-get update && apt-get install -y libxslt-dev
RUN docker-php-ext-install xsl

#  pgsql
RUN docker-php-ext-install pgsql

# pgsql client
RUN mkdir -p /usr/share/man/man1 && \
    mkdir -p /usr/share/man/man7 && \
    # Install the pgsql client
    apt-get install -y postgresql-client


# pcntl
RUN docker-php-ext-install pcntl

# intl
RUN apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl 



# ImageMagick
# RUN docker-php-ext-install imagick && docker-php-ext-enable imagick

# YAML
# RUN docker-php-ext-install yaml && docker-php-ext-enable yaml

# xDebug
# RUN docker-php-ext-install xdebug && docker-php-ext-enable xdebug

# COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
#     sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
#     sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini




# protobuf
RUN pecl install protobuf
RUN docker-php-ext-enable protobuf


# opcache
RUN docker-php-ext-install opcache
COPY opcache.ini /usr/local/etc/php/conf.d/opcache.ini


###########################################################################
# SQL SERVER:
###########################################################################

ARG INSTALL_MSSQL=true

RUN set -eux; if [ ${INSTALL_MSSQL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      apt-get -y install freetds-dev libsybdb5 \
      && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so \
      && docker-php-ext-install mssql pdo_dblib \
      && php -m | grep -q 'mssql' \
      && php -m | grep -q 'pdo_dblib' \
    ;else \
      ###########################################################################
      # Ref from https://github.com/Microsoft/msphpsql/wiki/Dockerfile-for-adding-pdo_sqlsrv-and-sqlsrv-to-official-php-image
      ###########################################################################
      # Add Microsoft repo for Microsoft ODBC Driver 13 for Linux
      apt-get install -y apt-transport-https gnupg \
      && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
      && curl https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list \
      && apt-get update -yqq \
      # Install Dependencies
      && ACCEPT_EULA=Y apt-get install -y unixodbc unixodbc-dev libgss3 odbcinst msodbcsql17 locales \
      && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
      # link local aliases
      && ln -sfn /etc/locale.alias /usr/share/locale/locale.alias \
      && locale-gen \
      # Install pdo_sqlsrv and sqlsrv from PECL. Replace pdo_sqlsrv-4.1.8preview with preferred version.
      && pecl install pdo_sqlsrv sqlsrv \
      && docker-php-ext-enable pdo_sqlsrv sqlsrv \
      && php -m | grep -q 'pdo_sqlsrv' \
      && php -m | grep -q 'sqlsrv' \
    ;fi \
;fi


# composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/


# supervior
COPY supervisord.conf /etc/supervisord.conf

# php配置文件
ADD php.ini    /usr/local/etc/php/php.ini

USER root



# 清理
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
# ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]