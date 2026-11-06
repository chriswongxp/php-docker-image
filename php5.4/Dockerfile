FROM centos:7

# ---- yum 源：centos 7.9.2009 vault（centos7 已 EOL）+ remi php54 + EPEL ----
RUN rm -f /etc/yum.repos.d/*.repo \
&& printf '[base]\nname=CentOS-7.9.2009-base\nbaseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/os/x86_64/\ngpgcheck=0\nenabled=1\n\n[updates]\nname=CentOS-7.9.2009-updates\nbaseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/updates/x86_64/\ngpgcheck=0\nenabled=1\n' > /etc/yum.repos.d/CentOS-Vault.repo \
&& printf '[remi-php54]\nname=Remi php54\nbaseurl=https://mirrors.aliyun.com/remi/enterprise/7/php54/x86_64/\ngpgcheck=0\nenabled=1\n\n[remi-safe]\nname=Remi safe\nbaseurl=https://mirrors.aliyun.com/remi/enterprise/7/safe/x86_64/\ngpgcheck=0\nenabled=1\n' > /etc/yum.repos.d/remi-php54.repo \
&& printf '[epel]\nname=EPEL-7\nbaseurl=https://mirrors.aliyun.com/epel/7/x86_64/\ngpgcheck=0\nenabled=1\n' > /etc/yum.repos.d/epel.repo

# ---- PHP 5.4 + 扩展（对齐 php5.3 容器 php -m 清单）----
RUN yum install -y --setopt=tsflags=nodocs \
    php54-php-cli php54-php-fpm php54-php-common \
    php54-php-mysqlnd php54-php-pdo php54-php-pgsql php54-php-sqlite3 \
    php54-php-gd php54-php-mbstring php54-php-mcrypt php54-php-soap \
    php54-php-xml php54-php-imap php54-php-bz2 php54-php-ftp \
    php54-php-gettext php54-php-process php54-php-pecl-zip php54-php-pear \
    php54-php-devel \
    php54-php-pecl-memcache \
&& yum clean all

# ---- 编译工具（pecl 源码编译 redis / timezonedb）----
RUN yum install -y --setopt=tsflags=nodocs gcc make \
&& yum clean all

# ---- redis-2.2.7（与 php5.3 环境一致，pecl 源码编译）----
WORKDIR /usr/src
RUN curl -O https://pecl.php.net/get/redis-2.2.7.tgz \
&& tar zxf redis-2.2.7.tgz \
&& cd redis-2.2.7 \
&& /opt/remi/php54/root/usr/bin/phpize \
&& ./configure --with-php-config=/opt/remi/php54/root/usr/bin/php-config \
&& make -j$(nproc) \
&& make install -j$(nproc)

# ---- timezonedb-2015.6（与 php5.3 环境版本一致，pecl 源码编译）----
RUN curl -O https://pecl.php.net/get/timezonedb-2015.6.tgz \
&& tar zxf timezonedb-2015.6.tgz \
&& cd timezonedb-2015.6 \
&& /opt/remi/php54/root/usr/bin/phpize \
&& ./configure --with-php-config=/opt/remi/php54/root/usr/bin/php-config \
&& make -j$(nproc) \
&& make install -j$(nproc)

# ---- 网络诊断工具 ----
RUN yum install -y --setopt=tsflags=nodocs \
    iproute telnet bind-utils curl net-tools \
&& yum clean all

# ---- 用户对齐宿主机 /srv/http/php 属主 1000:982，容器内同名 www-data ----
RUN groupmod -g 982 users \
&& useradd -u 1000 -g 982 -M -s /sbin/nologin www-data

# ---- php.ini 对齐 php5.3（redis / timezonedb / max_input_vars / 时区；default_charset 置空保持 5.3 行为）----
WORKDIR /opt/remi/php54/root/etc
RUN sed -i '$a extension=redis.so' php.ini \
&& sed -i '$a extension=timezonedb.so' php.ini \
&& sed -i '$a max_input_vars = 10000' php.ini \
&& sed -i 's/^;*date.timezone =.*/date.timezone = Asia\/Shanghai/' php.ini \
&& sed -i 's/^default_charset = "UTF-8"/default_charset =/' php.ini \
&& rm -f php.d/opcache.ini

# ---- php-fpm 对齐 php5.3（0.0.0.0:9000 / www-data / dynamic pm）----
RUN sed -i 's/^user = apache/user = www-data/' php-fpm.d/www.conf \
&& sed -i 's/^listen.allowed_clients =.*/;listen.allowed_clients =/' php-fpm.d/www.conf \
&& sed -i 's/^group = apache/group = users/' php-fpm.d/www.conf \
&& sed -i 's|^listen = .*|listen = 0.0.0.0:9000|' php-fpm.d/www.conf \
&& sed -i 's/^pm.max_children =.*/pm.max_children = 8/' php-fpm.d/www.conf \
&& sed -i 's/^pm.start_servers =.*/pm.start_servers = 2/' php-fpm.d/www.conf \
&& sed -i 's/^pm.min_spare_servers =.*/pm.min_spare_servers = 1/' php-fpm.d/www.conf \
&& sed -i 's/^pm.max_spare_servers =.*/pm.max_spare_servers = 3/' php-fpm.d/www.conf \
&& printf 'pm.status_path = /fpm-status\n' >> php-fpm.d/www.conf

RUN mkdir -p /var/www/html

# ---- 全局命令（remi SCL 路径不在默认 PATH，符号链接到 /usr/local/bin）----
RUN ln -s /opt/remi/php54/root/usr/bin/php /usr/local/bin/php \
&& ln -s /opt/remi/php54/root/usr/sbin/php-fpm /usr/local/bin/php-fpm \
&& ln -s /opt/remi/php54/root/usr/bin/phpize /usr/local/bin/phpize \
&& ln -s /opt/remi/php54/root/usr/bin/php-config /usr/local/bin/php-config \
&& ln -s /opt/remi/php54/root/usr/bin/pear /usr/local/bin/pear \
&& ln -s /opt/remi/php54/root/usr/bin/pecl /usr/local/bin/pecl

# ---- 宽松权限（测试/开发镜像定位）：日志与 session 目录放开写 ----
RUN mkdir -p /opt/remi/php54/root/var/lib/php/session \
&& chmod -R 777 /opt/remi/php54/root/var/log/php-fpm /opt/remi/php54/root/var/lib/php

COPY init.sh /init.sh
RUN chmod +x /init.sh

CMD ["/init.sh"]
