FROM phusion/baseimage:latest

MAINTAINER saunier.johann@gmail.com

ENV DEBIAN_FRONTEND noninteractive
ENV PHP_INI_DIR /etc/php7
ENV PHP_VERSION 7.0.6
ENV PHP_FILENAME php-7.0.6.tar.xz
ENV PHP_SHA256 1b237a9455e5476a425dbb9d99966bad68107747c601958cb9558a7fb49ab419
ENV PHP_USER www-data
ENV PHP_AUTOCONF /usr/bin/autoconf
ENV ENV_TYPE development
ENV PHP_AMQP_BUILD_DEPS libtool automake git pkg-config librabbitmq-dev

CMD ["/sbin/my_init"]

RUN apt-get update && apt-get upgrade -y

# persistent / runtime deps
RUN apt-get install -y \
    ca-certificates \
    wget \
    librecode0 \
    uuid-dev \
    mysql-client \
    libmagickwand-dev \
    libsasl2-dev \
    imagemagick \
    libmagickwand-dev \
    libmagickcore-dev \
    libsqlite3-0 \
    libxml2 \
    autoconf \
    file \
    g++ \
    gcc \
    libc-dev \
    make \
    pkg-config \
    re2c \
    libcurl4-openssl-dev \
    libreadline6-dev \
    librecode-dev \
    libzip-dev \
    libsqlite3-dev \
    libssl-dev \
    libxml2-dev \
    xz-utils \
    libicu-dev \
    libpcre3-dev \
    libbz2-dev \
    libpq-dev \
    libt1-dev \
    libjpeg8 \
    libpng12-dev \
    libfreetype6-dev \
    libmcrypt-dev \
    libgd2-xpm-dev \
    libgmp-dev \
    curl \
    $PHP_AMQP_BUILD_DEPS \
    libmagickwand-dev \
    libvips-dev \
    libgsf-1-dev \
    libmagickcore-dev \
    libevent-dev \
    libzmq-dev \
    --no-install-recommends

RUN mkdir -p $PHP_INI_DIR/conf.d

RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763";

RUN set -x \
    && curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" \
    && echo "$PHP_SHA256 *$PHP_FILENAME" | sha256sum -c - \
    && curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" \
    && gpg --verify "$PHP_FILENAME.asc" \
    && mkdir -p /usr/src/php \
    && tar -xf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 \
    && rm "$PHP_FILENAME"*

RUN cd /usr/src/php && ./configure \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --enable-intl \
        --enable-pcntl \
        --enable-mbstring \
        --with-mcrypt=/usr \
        --with-pcre-regex \
        --enable-pdo \
        --with-openssl \
        --with-openssl-dir=/usr/bin \
        --with-sqlite3=/usr \
        --with-pdo-sqlite=/usr \
        --enable-inline-optimization \
        --with-icu-dir=/usr \
        --with-curl=/usr/bin \
        --with-bz2 \
        --enable-sockets \
        --with-mysql=/usr/bin \
        --with-mysqli \
        --with-pdo-mysql \
        --with-gd \
        --with-pear \
        --enable-maintainer-zts \
        --with-tsrm-pthreads \
        --enable-pthreads=shared \
        --with-jpeg-dir=/usr \
        --with-png-dir=/usr \
        --with-xpm-dir=/usr \
        --with-freetype-dir=/usr \
        --enable-gd-native-ttf \
        --with-mcrypt \
        --enable-exif \
        --with-gettext \
        --enable-bcmath \
        --with-openssl \
        --with-readline \
        --with-recode \
        --with-zlib \
    && make -j"$(nproc)" \
    && make install \
    && { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; }

COPY docker-php-ext-* /usr/local/bin/

RUN cd /usr/lib && \
    git clone --depth=1 -b v0.7.1 git://github.com/alanxz/rabbitmq-c.git && \
    cd rabbitmq-c && \
    git submodule update --init && \
    autoreconf -i && \
    ./configure && \
    make && \
    make install

RUN docker-php-ext-install-pecl imagick-3.4.0RC6
RUN php -m | grep imagick

RUN docker-php-ext-install-pecl amqp-1.7.0alpha2
RUN php -m | grep amqp

RUN docker-php-ext-install-pecl msgpack-2.0.1
RUN php -m | grep msgpack

RUN docker-php-ext-install-pecl ev-beta
RUN php -m | grep ev

RUN docker-php-ext-install-pecl uuid
RUN php -m | grep uuid

RUN docker-php-ext-install-pecl mongodb
RUN php -m | grep mongodb

#RUN docker-php-ext-install-pecl redis
#RUN php -m | grep redis

# Currently not available through PECL :-(
RUN cd /etc && git clone --depth=1 -b php7 https://github.com/phpredis/phpredis.git \
    && cd /etc/phpredis \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && touch $PHP_INI_DIR/conf.d/ext-redis.ini \
    && echo 'extension=redis.so' >> $PHP_INI_DIR/conf.d/ext-redis.ini

RUN docker-php-ext-install-pecl zip
RUN php -m | grep zip

RUN docker-php-ext-install-pecl zmq-1.1.3
RUN php -m | grep zmq

RUN docker-php-ext-install-pecl pthreads
RUN php -m | grep pthreads

RUN docker-php-ext-install-pecl apcu-5.1.3
RUN php -m | grep apcu

RUN docker-php-ext-enable opcache

# Create php.ini
RUN curl https://raw.githubusercontent.com/php/php-src/PHP-$PHP_VERSION/php.ini-$ENV_TYPE -o $PHP_INI_DIR/php.ini

# Yes my lord, i'm gentle
RUN sed -i 's/\;date\.timezone\ \=/date\.timezone\ \=\ Europe\/Paris/g' $PHP_INI_DIR/php.ini
RUN sed -i "s/error_reporting = .*/error_reporting = E_ALL ^ E_STRICT/" $PHP_INI_DIR/php.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer
RUN mkdir -p /root/.composer

VOLUME /root/.composer

#CLEAN
RUN apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o \
    APT::AutoRemove::SuggestsImportant=false $buildDeps $PHP_AMQP_BUILD_DEPS
RUN rm -Rf /usr/lib/rabbitmq-c
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#Secure
RUN chmod 755 /etc/container_environment
RUN chmod 644 /etc/container_environment.sh /etc/container_environment.json
