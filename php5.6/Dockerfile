FROM php:5.6-fpm

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
&& { \
echo deb http://mirrors.163.com/debian/ stretch main non-free contrib; \
echo deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib; \
echo deb-src http://mirrors.163.com/debian/ stretch main non-free contrib; \
echo deb-src http://mirrors.163.com/debian/ stretch-updates main non-free contrib; \
echo deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib; \
echo deb-src http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib; \
} > /etc/apt/sources.list \
&& apt-get -y update --fix-missing \
&& apt-get upgrade -y \
&& apt-get install -y apt-file \
&& apt-get -y install zip libmcrypt-dev libzip-dev \
&& apt-get -y install apt-utils git curl \
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype6-dev \
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev \
&& cp /usr/lib/x86_64-linux-gnu/libldap* /usr/lib \
&& pecl channel-update pecl.php.net

RUN pecl install redis-2.2.7 \
&& docker-php-ext-enable redis \
&& rm -rf /tmp/pear

RUN docker-php-ext-install ldap \
&& docker-php-ext-install mcrypt \
&& docker-php-ext-install pdo_mysql \
&& docker-php-ext-install mysqli \
&& docker-php-ext-install zip \
&& docker-php-ext-install intl \
&& docker-php-ext-install bcmath

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
&& docker-php-ext-install -j$(nproc) gd \
&& sed -i '$a date.timezone=PRC' /usr/local/etc/php/php.ini \
&& sed -i '$a max_input_vars = 10000' /usr/local/etc/php/php.ini \
&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer