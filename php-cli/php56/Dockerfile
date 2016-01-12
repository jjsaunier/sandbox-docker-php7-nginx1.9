FROM ubuntu:14.04

MAINTAINER johann_27@hotmail.fr

ENV DEBIAN_FRONTEND noninteractive
ENV PHP_INI_DIR /etc/php5
ENV PHP_VERSION 5.6.16
ENV PHP_FILENAME php-5.6.16.tar.xz
ENV PHP_SHA256 8ef43271d9bd8cc8f8d407d3ba569de9fa14a28985ae97c76085bb50d597de98
ENV PHP_AUTOCONF /usr/bin/autoconf
ENV ENV_TYPE development

# persistent / runtime deps
RUN apt-get update && apt-get install -y ca-certificates wget librecode0 libmagickwand-dev imagemagick libsqlite3-0 libxml2 --no-install-recommends && rm -r /var/lib/apt/lists/*

# phpize deps
RUN apt-get update && apt-get install -y autoconf file g++ gcc libc-dev make pkg-config re2c --no-install-recommends && rm -r /var/lib/apt/lists/*

RUN mkdir -p $PHP_INI_DIR/conf.d

RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "0BD78B5F97500D450838F95DFE857D9A90D90EC1";
RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3";

RUN buildDeps=" \
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
    " \
    && set -x \
    && apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
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
        --disable-cgi \
        --enable-intl \
        --enable-pcntl \
        --enable-mbstring \
        --enable-force-cgi-redirect \
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
        --enable-phar \
        --enable-sockets \
        --with-pdo-mysqlnd \
        --with-gd \
        --with-pear \
        --with-tsrm-pthreads \
        --enable-phar=shared \
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

ENV PHP_AMQP_BUILD_DEPS libtool automake git pkg-config librabbitmq-dev libzmq-dev

RUN apt-get update && apt-get install -y $PHP_AMQP_BUILD_DEPS --no-install-recommends && rm -r /var/lib/apt/lists/*

#Must be keep
RUN apt-get update && apt-get install -y libmagickwand-dev libvips-dev libgsf-1-dev libmagickcore-dev libevent-dev --no-install-recommends && rm -r /var/lib/apt/lists/*

RUN cd /usr/lib && \
    git clone git://github.com/alanxz/rabbitmq-c.git && \
    cd rabbitmq-c && \
    git checkout v0.7.1 && \
    git submodule update --init && \
    autoreconf -i && \
    ./configure && \
    make && \
    make install

RUN docker-php-ext-install-pecl phar
RUN docker-php-ext-install-pecl imagick

RUN docker-php-ext-install-pecl amqp
RUN php -m | grep amqp

RUN docker-php-ext-install-pecl big_int

RUN docker-php-ext-install-pecl msgpack-0.5.7
RUN php -m | grep msgpack

RUN docker-php-ext-install-pecl ev
RUN php -m | grep ev

RUN docker-php-ext-install-pecl mongo
RUN php -m | grep mongo

RUN docker-php-ext-install-pecl redis
RUN php -m | grep redis

RUN docker-php-ext-install-pecl zip
RUN php -m | grep zip

#CLEAN
RUN apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps $PHP_AMQP_BUILD_DEPS
RUN apt-get clean all
RUN rm -Rf /usr/lib/rabbitmq-c

# Create php.ini
RUN curl https://raw.githubusercontent.com/php/php-src/PHP-$PHP_VERSION/php.ini-$ENV_TYPE -o $PHP_INI_DIR/php.ini

# Yes my lord, i'm gentle
RUN sed -i 's/\;date\.timezone\ \=/date\.timezone\ \=\ Europe\/Paris/g' $PHP_INI_DIR/php.ini
RUN sed -i "s/error_reporting = .*/error_reporting = E_ALL ^ E_STRICT/" $PHP_INI_DIR/php.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php && sudo mv composer.phar /usr/local/bin/composer
RUN apt-get update && apt-get install -y git && rm -r /var/lib/apt/lists/*
RUN mkdir -p /root/.composer
