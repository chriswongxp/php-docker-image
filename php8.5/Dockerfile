FROM php:8.5-fpm

RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
&& { \
echo date.timezone=Asia/Shanghai; \
echo xdebug.var_display_max_children=128; \
echo xdebug.var_display_max_data=1024; \
echo xdebug.var_display_max_depth=15; \
echo max_input_vars = 10000; \
} >> /usr/local/etc/php/php.ini \
&& pecl channel-update pecl.php.net \
&& pecl install redis-6.3.0 \
&& docker-php-ext-enable redis \
&& { \
echo "Types: deb"; \
echo "URIs: https://mirrors.tencent.com/debian/"; \
echo "Suites: trixie trixie-updates"; \
echo "Components: main contrib non-free non-free-firmware"; \
echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg"; \
echo; \
echo "Types: deb"; \
echo "URIs: https://mirrors.tencent.com/debian-security/"; \
echo "Suites: trixie-security"; \
echo "Components: main contrib non-free non-free-firmware"; \
echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg"; \
} > /etc/apt/sources.list.d/debian.sources \
&& apt-get -y update --fix-missing \
&& apt-get upgrade -y \
&& apt-get -y install zip libmcrypt-dev apt-utils build-essential git curl libcurl4t64 libzip-dev libevent-dev \
&& apt-get -y install libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libfreetype-dev \
&& apt-get -y install libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev \
&& cp /usr/lib/x86_64-linux-gnu/libldap* /usr/lib \
&& pecl install xdebug-3.5.3 \
&& docker-php-ext-enable xdebug \
&& docker-php-ext-install ldap \
&& docker-php-ext-install pdo_mysql \
&& docker-php-ext-install mysqli \
&& docker-php-ext-install zip \
&& docker-php-ext-install intl \
&& docker-php-ext-install pcntl \
&& docker-php-ext-install sockets \
&& docker-php-ext-install bcmath \
&& pecl install event-3.2.0 \
&& docker-php-ext-enable event \
&& docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) gd \
&& rm -rf /tmp/pear \
&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# ---- 网络诊断工具（对齐 php5.4：ip/telnet/nslookup/curl/ping/netstat）----
RUN apt-get -y install iproute2 telnet bind9-dnsutils net-tools iputils-ping

# ---- event 须在 sockets 之后加载：conf.d 字母序 event 先于 sockets，8.5 下加载即报 undefined symbol: socket_ce ----
RUN mv /usr/local/etc/php/conf.d/docker-php-ext-event.ini /usr/local/etc/php/conf.d/zzz-docker-php-ext-event.ini
