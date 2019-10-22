FROM php:7.2-fpm

# 添加源
# RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
# ADD sources.list /etc/apt/sources.list
RUN apt-get update && pecl channel-update https://pecl.php.net/channel.xml

# phpredis
RUN pecl install redis
RUN docker-php-ext-enable redis 

# swoole
RUN pecl install swoole
RUN docker-php-ext-enable swoole 

# mongodb
RUN pecl install mongodb
RUN docker-php-ext-enable mongodb


# BC Math
RUN docker-php-ext-install bcmath

# protobuf
RUN pecl install protobuf
RUN docker-php-ext-enable protobuf


# opcache
RUN docker-php-ext-install opcache
COPY opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# php配置文件
ADD php.ini    /usr/local/etc/php/php.ini

USER root

# 清理
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000