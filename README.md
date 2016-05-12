Docker php7 nginx1.9 sandbox
--------------------------

**php version:** PHP 7.0.6 with amqp redis msgpack mongodb ev apcu zmq uuid imagick zip pthreads (cli) 

**nginx version:** nginx/1.9.9 with upload-progress-module pagespeed_module

## Setup dist file

`cp php-cli/php70/auth.json.dist php-cli/php70/auth.json` and then replace by your own github token.


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

## Config files

* **php fpm** : `php-fpm/php70/php-fpm.conf`
* **php fpm pool www** : `php-fpm/php70/www.pool.conf`
* **nginx** : `nginx/nginx.conf`
* **nginx www vhost** : `nginx/www.conf`

## PHP modules 

```bash
docker run --rm -it sandbox_php-cli php -m
```

```
[PHP Modules]
amqp
apcu
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
mysqli
mysqlnd
openssl
pcntl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
posix
pthreads
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
zmq
```
