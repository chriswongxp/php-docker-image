FROM php:8.2-fpm

# 修改源、更新系统、安装php需要的相关依赖
RUN sed -i 's@http://deb.debian.org@https://mirrors.tencent.com@g' /etc/apt/sources.list.d/debian.sources \
&& apt-get -y update --fix-missing \
&& apt-get upgrade -y \
&& apt-get -y install zip libmcrypt-dev apt-utils build-essential git curl libcurl4 libzip-dev libevent-dev \
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libssl-dev libmagickwand-dev libmagickcore-dev

# 安装php扩展
RUN docker-php-ext-install sockets \
&& docker-php-ext-install pdo_mysql \
&& docker-php-ext-install zip \
&& docker-php-ext-install intl \
&& docker-php-ext-install pcntl \
&& docker-php-ext-install bcmath \
&& docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) gd \
&& pecl channel-update pecl.php.net \
&& pecl install event \
&& pecl install redis \
&& pecl install xdebug \
&& docker-php-ext-enable redis \
&& docker-php-ext-enable xdebug \
&& docker-php-ext-enable event \
&& rm -rf /tmp/pear

# 修改php配置文件
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
&& { \
echo date.timezone=PRC; \
echo xdebug.var_display_max_children=128; \
echo xdebug.var_display_max_data=1024; \
echo xdebug.var_display_max_depth=15; \
echo max_input_vars = 10000; \
} >> /usr/local/etc/php/php.ini

# 安装php composer工具
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
