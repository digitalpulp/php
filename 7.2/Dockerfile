FROM php:7.2-fpm-alpine
MAINTAINER digitalpulp

VOLUME /var/www
WORKDIR /var/www

RUN apk add --no-cache --virtual .dd-build-deps libpng-dev libjpeg-turbo-dev postgresql-dev libxml2-dev $PHPIZE_DEPS \
   && apk add --no-cache --update libmemcached-libs zlib \
   && set -xe \
   && cd /tmp/ \
   && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \
   && apk add --no-cache --update --virtual .memcached-deps zlib-dev libmemcached-dev cyrus-sasl-dev \
   && pecl install igbinary \
   && ( \
        pecl install --nobuild memcached && \
        cd "$(pecl config-get temp_dir)/memcached" && \
        phpize && \
        ./configure --enable-memcached-igbinary && \
        make -j$(nproc) && \
        make install && \
        cd /tmp/ \
    )  \
   && docker-php-ext-enable igbinary memcached \
   && rm -rf /tmp/* \
   && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
   && docker-php-ext-install gd mbstring pdo pdo_mysql mysqli pdo_pgsql zip \
   && docker-php-ext-install opcache bcmath soap \
   && pecl install redis \
   && pecl install xdebug \
   && docker-php-ext-enable redis xdebug \
   && apk add --no-cache libpng libjpeg libpq libxml2 libmemcached \
   && apk add imagemagick \
   && apk del .dd-build-deps .memcached-deps .phpize-deps\
   && update-ca-certificates

COPY drupal-*.ini /usr/local/etc/php/conf.d/
COPY xdebug.ini /usr/local/etc/php/conf.d/
COPY drupal-fpm.conf /usr/local/etc/php-fpm.d
