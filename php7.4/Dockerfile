FROM php:7.4-fpm

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
&& { \
echo date.timezone=PRC; \
echo xdebug.var_display_max_children=128; \
echo xdebug.var_display_max_data=1024; \
echo xdebug.var_display_max_depth=15; \
echo max_input_vars = 10000; \
} >> /usr/local/etc/php/php.ini \
&& pecl channel-update pecl.php.net \
&& pecl install redis-5.3.7 \
&& docker-php-ext-enable redis \
&& { \
echo "deb https://mirrors.tencent.com/debian/ bullseye main non-free contrib"; \
echo "deb-src https://mirrors.tencent.com/debian/ bullseye main non-free contrib"; \
echo "deb https://mirrors.tencent.com/debian-security/ bullseye-security main"; \
echo "deb-src https://mirrors.tencent.com/debian-security/ bullseye-security main"; \
echo "deb https://mirrors.tencent.com/debian/ bullseye-updates main non-free contrib"; \
echo "deb-src https://mirrors.tencent.com/debian/ bullseye-updates main non-free contrib"; \
echo "deb https://mirrors.tencent.com/debian/ bullseye-backports main non-free contrib"; \
echo "deb-src https://mirrors.tencent.com/debian/ bullseye-backports main non-free contrib"; \
} > /etc/apt/sources.list \
&& apt-get -y update --fix-missing \
&& apt-get upgrade -y \
&& apt-get -y install zip libmcrypt-dev apt-utils build-essential git curl libcurl4 libzip-dev libevent-dev \
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev \
&& cp /usr/lib/x86_64-linux-gnu/libldap* /usr/lib \
&& pecl install xdebug-3.1.6 \
&& docker-php-ext-enable xdebug \
&& docker-php-ext-install ldap \
&& docker-php-ext-install pdo_mysql \
&& docker-php-ext-install mysqli \
&& docker-php-ext-install zip \
&& docker-php-ext-install intl \
&& docker-php-ext-install pcntl \
&& docker-php-ext-install sockets \
&& docker-php-ext-install bcmath \
&& pecl install event-3.0.8 \
&& docker-php-ext-enable event \
&& docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) gd \
&& rm -rf /tmp/pear \
&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer