# Create static network example:
docker network create -d bridge --gateway=172.18.0.1 --subnet=172.18.2.0/16 static

# Run container example:
docker run -d -e TZ="Asia/Shanghai" --name php8.2 --restart=always --network=static --ip 172.18.2.2 -v /srv/http/php:/var/www/html chriswongxp/php8.2

# Dockerfile:
FROM php:8.2-fpm

# 修改源、更新系统、安装php需要的相关依赖
RUN sed -i 's@http://deb.debian.org@https://mirrors.tencent.com@g' /etc/apt/sources.list.d/debian.sources \\<br>
&& apt-get -y update --fix-missing \\<br>
&& apt-get upgrade -y \\<br>
&& apt-get -y install zip libmcrypt-dev apt-utils build-essential git curl libcurl4 libzip-dev libevent-dev \\<br>
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \\<br>
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libssl-dev libmagickwand-dev libmagickcore-dev

# 安装php扩展
RUN docker-php-ext-install sockets \\<br>
&& docker-php-ext-install pdo_mysql \\<br>
&& docker-php-ext-install zip \\<br>
&& docker-php-ext-install intl \\<br>
&& docker-php-ext-install pcntl \\<br>
&& docker-php-ext-install bcmath \\<br>
&& docker-php-ext-configure gd --with-freetype --with-jpeg \\<br>
&& docker-php-ext-install -j$(nproc) gd \\<br>
&& pecl channel-update pecl.php.net \\<br>
&& pecl install event \\<br>
&& pecl install redis \\<br>
&& pecl install xdebug \\<br>
&& docker-php-ext-enable redis \\<br>
&& docker-php-ext-enable xdebug \\<br>
&& docker-php-ext-enable event \\<br>
&& rm -rf /tmp/pear

# 修改php配置文件
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \\<br>
&& { \\<br>
echo date.timezone=PRC; \\<br>
echo xdebug.var_display_max_children=128; \\<br>
echo xdebug.var_display_max_data=1024; \\<br>
echo xdebug.var_display_max_depth=15; \\<br>
echo max_input_vars = 10000; \\<br>
} >> /usr/local/etc/php/php.ini

# 安装php composer工具
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
