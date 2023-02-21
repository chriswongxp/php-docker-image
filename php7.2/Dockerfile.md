# Create static network example:
docker network create -d bridge --gateway=172.18.0.1 --subnet=172.18.2.0/16 static

# Run container example:
docker run -d -e TZ="Asia/Shanghai" --name php7.2 --restart=always --network=static --ip 172.18.2.2 -p 8301-8310:8301-8310 -p 8401-8410:8401-8410 -v /srv/http/php:/var/www/html chriswongxp/php7.2

# Dockerfile:
FROM php:7.2-fpm

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \\<br>
&& { \\<br>
echo date.timezone=PRC; \\<br>
echo xdebug.var_display_max_children=128; \\<br>
echo xdebug.var_display_max_data=1024; \\<br>
echo xdebug.var_display_max_depth=15; \\<br>
echo max_input_vars = 10000; \\<br>
} >> /usr/local/etc/php/php.ini \\<br>
&& pecl install redis-5.3.7 \\<br>
&& docker-php-ext-enable redis \\<br>
&& rm -rf /tmp/pear \\<br>
&& { \\<br>
echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free"; \\<br>
echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free"; \\<br>
echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free"; \\<br>
echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free"; \\<br>
} > /etc/apt/sources.list \\<br>
&& apt-get -y update --fix-missing \\<br>
&& apt-get upgrade -y \\<br>
&& apt-get -y install zip libmcrypt-dev apt-utils build-essential git curl libcurl4 libzip-dev libevent-dev \\<br>
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \\<br>
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev \\<br>
&& cp /usr/lib/x86_64-linux-gnu/libldap* /usr/lib \\<br>
&& pecl install xdebug-3.1.6 \\<br>
&& docker-php-ext-enable xdebug \\<br>
&& docker-php-ext-install ldap \\<br>
&& docker-php-ext-install pdo_mysql \\<br>
&& docker-php-ext-install mysqli \\<br>
&& docker-php-ext-install zip \\<br>
&& docker-php-ext-install intl \\<br>
&& docker-php-ext-install pcntl \\<br>
&& docker-php-ext-install sockets \\<br>
&& docker-php-ext-install bcmath \\<br>
&& pecl install event-3.0.8 \\<br>
&& docker-php-ext-enable event  \\<br>
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/freetype2 --with-jpeg-dir=/usr/include --with-png-dir=/usr/include \\<br>
&& docker-php-ext-install -j$(nproc) gd \\<br>
&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
