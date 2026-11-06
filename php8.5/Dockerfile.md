# 镜像分发（阿里云 ACR，固定地址，公开仓库免登录拉取）

registry 地址（实例 ID+地域，开通后不变，无需再登阿里云后台查询）：
crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com

其他服务器部署（直接用长名，不做 tag 短名）：
docker pull crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.5:latest
docker run -d -e TZ="Asia/Shanghai" --name php8.5 --restart=always --network=static --ip 172.18.0.8 -v /srv/http/php:/var/www/html crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.5:latest

（--network/--ip/--volume 按各机器实际情况调整；验证三件事：php -v 是 8.5.12、date 是北京时间、nginx 机器发真实请求 200）

# Run container example（本机构建运行）:
docker run -d -e TZ="Asia/Shanghai" --name php8.5 --restart=always --network=static --ip 172.18.0.8 -v /srv/http/php:/var/www/html chriswongxp/php8.5

# 环境说明

- PHP 8.5.12（官方 php:8.5-fpm，Debian 13 trixie 基础镜像），cli/fpm 路径 /usr/local/bin/php、/usr/local/sbin/php-fpm，监听 0.0.0.0:9000（官方镜像 zz-docker.conf 默认，本机 nginx upstream fastcgi_pass）
- 用途：项目升级 PHP 8.5 的开发/测试验证环境（解决 PHP2Shell 漏洞的升级路线）
- 配置：/usr/local/etc/php/php.ini（php.ini-development + 追加 5 行）、/usr/local/etc/php/conf.d/*.ini

# 与 php7.4 容器（chriswongxp/php7.4）的依赖对齐

apt 包（trixie 包名差异 3 处，其余同 php7.4 清单）：
- 同名照搬：zip libmcrypt-dev apt-utils build-essential git curl libzip-dev libevent-dev libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev
- 包名差异（bullseye→trixie）：libcurl4 → libcurl4t64、libfreetype6-dev → libfreetype-dev、dnsutils → bind9-dnsutils

扩展（编译安装清单与 php7.4 一致）：
- docker-php-ext-install：ldap pdo_mysql mysqli zip intl pcntl sockets bcmath gd（gd --with-freetype --with-jpeg）
- pecl：redis-6.3.0（php7.4 为 5.3.7，随 PHP 8.5 升）、xdebug-3.5.3（php7.4 为 3.1.6）、event-3.2.0（php7.4 为 3.0.8）
- 基础镜像自带：curl mbstring dom xml tokenizer pdo_sqlite sodium 等（同 php7.4 基线）
- composer 2.11.1（getcomposer.org installer）
- php.ini 对齐 php7.4 五项：xdebug.var_display_max_children=128 / max_data=1024 / max_depth=15、max_input_vars = 10000；date.timezone 用 Asia/Shanghai（php7.4 写 PRC，同一时区换规范名，也避开 php5.4 那次 PRC 不识别的坑）

网络诊断工具（对齐 php5.4）：ip（iproute2）、telnet、nslookup（bind9-dnsutils）、curl、ping（iputils-ping）、netstat（net-tools）

# 构建踩坑

- Debian 13 (trixie) apt 源是 deb822 格式（/etc/apt/sources.list.d/debian.sources），不是 bullseye 的 /etc/apt/sources.list；腾讯源改写要按 Types/URIs/Suites/Components/Signed-By 写
- pecl event 加载报 "undefined symbol: socket_ce"：conf.d 按文件名字母序加载，docker-php-ext-event.ini 排在 docker-php-ext-sockets.ini 前面，PHP 8.5 下 event.so 加载时强解析 sockets 的 socket_ce 即失败（php7.4 同序却能加载，老版本行为宽松）。修法：把 event 的 ini 改名为 zzz-docker-php-ext-event.ini 置后
- trixie 包名迁移：libcurl4→libcurl4t64、libfreetype6-dev→libfreetype-dev、dnsutils→bind9-dnsutils（dnsutils 过渡包已移除）
- 时区：Asia/Shanghai 规范名（PRC 是 tzdata backward 别名，新版本别再用）
