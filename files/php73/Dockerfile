FROM php:7.3-fpm-alpine
LABEL maintainer="aileshang0226@163.com"

ARG CONTAINER_PACKAGE_URL
ARG TZ

## 设置APK源
#RUN sed -i "s/dl-cdn.alpinelinux.org/${CONTAINER_PACKAGE_URL}/g" /etc/apk/repositories

# 更新源(tzdata、zeromq是为了处理时区文件不存在的问题)
ENV G=g++
RUN apk --update add --no-cache --virtual .build-deps autoconf $G libtool make curl-dev gettext-dev  \
 linux-headers git re2c tzdata zeromq shadow --repository http://${CONTAINER_PACKAGE_URL}/alpine/v3.11/main/ --allow-untrusted

# 安装php扩展
RUN docker-php-ext-install -j$(nproc) pdo_mysql \
    && docker-php-ext-install -j$(nproc) mysqli \
    && apk add --no-cache freetype freetype-dev libpng libpng-dev libjpeg-turbo libjpeg-turbo-dev \
     && docker-php-ext-configure gd \
      options="--with-gd --with-freetype-dir=/usr/include/ --with-png-dir=/usr/include/ --with-jpeg-dir=/usr/include/" \
       && docker-php-ext-install -j$(nproc) gd \
        && apk del freetype-dev libpng-dev libjpeg-turbo-dev \
    && docker-php-ext-install opcache

# Fix: https://github.com/docker-library/php/issues/240
RUN apk add gnu-libiconv libstdc++ --no-cache --repository \
 http://${CONTAINER_PACKAGE_URL}/alpine/edge/community/ --allow-untrusted

ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

ENV REDIS_EXTENSIONS_VERSION=5.1.1
# Install Redis extensions
RUN cd /home && wget http://pecl.php.net/get/redis-$REDIS_EXTENSIONS_VERSION.tgz \
    && tar -zxvf redis-$REDIS_EXTENSIONS_VERSION.tgz \
    && cd redis-$REDIS_EXTENSIONS_VERSION && phpize && ./configure --with-php-config=/usr/local/bin/php-config \
    && make && make install \
    && echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini

# Install composer and change it's cache home
RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer

# 时区问题处理
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

WORKDIR /data

# Write Permission
RUN usermod -u 1000 www-data

