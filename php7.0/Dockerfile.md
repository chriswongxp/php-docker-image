# Create static network example:
docker network create -d bridge --gateway=172.18.0.1 --subnet=172.18.2.0/16 static

# Run container example:
docker run -d -e TZ="Asia/Shanghai" --name php7.0 --restart=always --network=static --ip 172.18.2.3 -v /srv/http/php:/var/www/html chriswongxp/php7.0

# Dockerfile:
FROM php:7.0-fpm

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \\<br>
&& { \\<br>
echo deb http://mirrors.163.com/debian/ stretch main non-free contrib; \\<br>
echo deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib; \\<br>
echo deb-src http://mirrors.163.com/debian/ stretch main non-free contrib; \\<br>
echo deb-src http://mirrors.163.com/debian/ stretch-updates main non-free contrib; \\<br>
echo deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib; \\<br>
echo deb-src http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib; \\<br>
} > /etc/apt/sources.list \\<br>
&& apt-get -y update --fix-missing \\<br>
&& apt-get upgrade -y \\<br>
&& apt-get install -y apt-file \\<br>
&& apt-get -y install zip libmcrypt-dev libzip-dev \\<br>
&& apt-get -y install apt-utils git curl \\<br>
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \\<br>
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev \\<br>
&& cp /usr/lib/x86_64-linux-gnu/libldap* /usr/lib \\<br>
&& pecl channel-update pecl.php.net \\<br>
&& pecl install redis \\<br>
&& docker-php-ext-enable redis \\<br>
&& pecl install xdebug-2.7.2 \\<br>
&& docker-php-ext-enable xdebug \\<br>
&& rm -rf /tmp/pear \\<br>
&& docker-php-ext-install ldap \\<br>
&& docker-php-ext-install mcrypt \\<br>
&& docker-php-ext-install pdo_mysql \\<br>
&& docker-php-ext-install mysqli \\<br>
&& docker-php-ext-install zip \\<br>
&& docker-php-ext-install intl \\<br>
&& docker-php-ext-install bcmath \\<br>
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \\<br>
&& docker-php-ext-install -j$(nproc) gd \\<br>
&& { \\<br>
echo date.timezone=PRC; \\<br>
echo xdebug.var_display_max_children=128; \\<br>
echo xdebug.var_display_max_data=1024; \\<br>
echo xdebug.var_display_max_depth=15; \\<br>
echo max_input_vars = 10000; \\<br>
} >> /usr/local/etc/php/php.ini \\<br>
&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
