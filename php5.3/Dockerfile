FROM nkirnos/php5.3.29:latest

RUN mkdir -p /var/www/html

WORKDIR /usr/src

RUN curl -O https://pecl.php.net/get/redis-2.2.7.tgz \
&& tar zxf redis-2.2.7.tgz \
&& cd redis-2.2.7 \
&& phpize \
&& ./configure --with-php-config=/usr/bin/php-config \
&& make -j$(nproc) \
&& make install -j$(nproc)

WORKDIR /etc/php5/etc

RUN sed -i '$a extension=redis.so' php.ini \
&& sed -i '$a max_input_vars = 10000' php.ini \
&& sed -i 's/Europe\/Moscow/PRC/g' /etc/php5/etc/php.ini