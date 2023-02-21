# Run container example:
docker run -d -e TZ="Asia/Shanghai" --name php5.3 --restart=always --network=static --ip 172.18.2.5 -v /srv/http/php:/var/www/html chriswongxp/php5.3

# Dockerfile
FROM nkirnos/php5.3.29:latest

RUN mkdir -p /var/www/html

WORKDIR /usr/src

RUN curl -O https://pecl.php.net/get/redis-2.2.7.tgz \\<br>
&& tar zxf redis-2.2.7.tgz \\<br>
&& cd redis-2.2.7 \\<br>
&& phpize \\<br>
&& ./configure --with-php-config=/usr/bin/php-config \\<br>
&& make -j$(nproc) \\<br>
&& make install -j$(nproc)

WORKDIR /etc/php5/etc

RUN sed -i '$a extension=redis.so' php.ini \\<br>
&& sed -i '$a max_input_vars = 10000' php.ini \\<br>
&& sed -i 's/Europe\\/Moscow/PRC/g' /etc/php5/etc/php.ini
