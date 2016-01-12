Docker php7 nginx1.9 sandbox
--------------------------

**php version:** PHP 7.0.2 with amqp redis msgpack mongodb ev

**nginx version:** nginx/1.9.9 with upload-progress-module pagespeed_module


## Run the stack

```bash
docker-compose -p sandbox  up -d
```

## Show services

```bash
docker-compose -p sandbox ps
```

## Install dependencies

```bash
docker run --rm -it -v $(pwd)/shared/composer:/root/.composer --volumes-from application_data_volume -w /var/www/ sandbox_php-cli composer install
```

## Run php cli

```bash
docker run --rm -it sandbox_php-cli bash
```

With application volume : 

```bash
docker run --rm -it --volumes-from application_data_volume -w /var/www/ sandbox_php-cli bash
```

And if you want use composer, don't forget to mount cache folder

```bash
docker run --rm -it -v $(pwd)/shared/composer:/root/.composer sandbox_php-cli composer
```

## Make writable directories

```bash
docker run --rm -it --volumes-from application_data_volume -w /var/www/ busybox chmod 777 -Rf var/cache var/logs
```

## All good ?

Visite http://127.0.0.1 or https://127.0.0.1

**http** use http protocol
**https** use http2 protocol

## Look at the logs

```bash
# all logs
docker-compose -p sandbox logs

# specific service
docker-compose -p sandbox logs nginx
```

## PHP modules 

```bash
docker run --rm -it sandbox_php-cli php -m
```

```
[PHP Modules]
amqp
bcmath
bz2
Core
ctype
curl
date
dom
ev
exif
fileinfo
filter
gd
gettext
hash
iconv
imagick
intl
json
libxml
mbstring
mcrypt
mongodb
msgpack
openssl
pcntl
pcre
PDO
pdo_sqlite
Phar
posix
readline
recode
redis
Reflection
session
SimpleXML
sockets
SPL
sqlite3
standard
tokenizer
uuid
xml
xmlreader
xmlwriter
Zend OPcache
zip
zlib

```
