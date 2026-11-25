# 镜像分发（阿里云 ACR，固定地址，公开仓库免登录拉取）

registry 地址（实例 ID+地域，开通后不变，无需再登阿里云后台查询）：
crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com

其他服务器部署（直接用长名，不做 tag 短名）：
docker pull crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.3:latest
docker run -d -e TZ="Asia/Shanghai" --name php8.3 --restart=always --network=static --ip 172.18.0.7 -v /srv/http/php:/var/www/html crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.3:latest

（--network/--ip/--volume 按各机器实际情况调整；验证四件事：php -v 是 8.3.x、date 是北京时间、composer --version 是 2.11.2、nginx 机器发真实请求 200）

# Run container example（本机构建运行）:
docker run -d -e TZ="Asia/Shanghai" --name php8.3 --restart=always --network=static --ip 172.18.0.7 -v /srv/http/php:/var/www/html chriswongxp/php8.3

# 环境说明

- PHP 8.3.37（官方 php:8.3-fpm，Debian 13 trixie 基础镜像），cli/fpm 路径 /usr/local/bin/php、/usr/local/sbin/php-fpm，监听 0.0.0.0:9000（官方镜像 zz-docker.conf 默认，本机 nginx upstream fastcgi_pass）
- Composer 2.11.2（getcomposer.org installer，2026-11-23 构建时为最新版，与 packagist composer/composer 最新 release 一致）
- 用途：项目升级 PHP 8.3 的开发/测试验证环境
- 配置：/usr/local/etc/php/php.ini（php.ini-development + 追加 5 行）、/usr/local/etc/php/conf.d/*.ini

# IP 规划（本机 static 网络 172.18.0.0/16）

| 容器 | IP |
|------|----|
| php7.4 | 172.18.0.2 |
| php5.4 | 172.18.0.3 |
| php5.6 | 172.18.0.4 |
| php7.2 | 172.18.0.5 |
| php7.0 | 172.18.0.6 |
| php8.3 | 172.18.0.7 |
| php8.5 | 172.18.0.8 |

# 与 php8.5 容器的依赖对齐

基础镜像同为 Debian 13 (trixie)，apt 包名清单与 php8.5 **完全一致**：
- libcurl4t64、libfreetype-dev、bind9-dnsutils（trixie 包名，非 bullseye 的 libcurl4 / libfreetype6-dev / dnsutils）
- 其余同名照搬：zip libmcrypt-dev apt-utils build-essential git curl libzip-dev libevent-dev libsqlite3-dev libsqlite3-0 mariadb-client zlib1g-dev libicu-dev libjpeg62-turbo-dev libpng-dev libldap2-dev libssl-dev libmagickwand-dev libmagickcore-dev

扩展（编译安装清单与 php8.5 一致）：
- docker-php-ext-install：ldap pdo_mysql mysqli zip intl pcntl sockets bcmath gd（gd --with-freetype --with-jpeg）
- pecl：redis-6.3.0、xdebug-3.5.3、event-3.2.0（与 php8.5 同版本）
- 基础镜像自带：curl mbstring dom xml tokenizer pdo_sqlite sodium 等
- php.ini 对齐 php8.5 五项：xdebug.var_display_max_children=128 / max_data=1024 / max_depth=15、max_input_vars = 10000、date.timezone=Asia/Shanghai

网络诊断工具（对齐 php5.4）：ip（iproute2）、telnet、nslookup（bind9-dnsutils）、curl、ping（iputils-ping）、netstat（net-tools）

# 构建踩坑

- **pecl 安装扩展报 `Connection to 'ssl://pecl.php.net:443' failed: Connection timed out`**：pecl 走 PHP streams 解析 pecl.php.net 时拿到 A + AAAA 两条记录，直连 IPv6 超时（宿主机与容器内 `curl` 都正常，因为 curl 有 IPv4 回退）。**修法：改用 `curl` 下载 tgz 后 `pecl install /tmp/xxx.tgz` 本地安装**，实测本地安装不再访问网络。不硬编码 IP，pecl.php.net 换 IP 也不受影响。php8.5 那份 Dockerfile 用的仍是 `pecl install redis-6.3.0` 在线装，若哪天重建同样超时，按此法改。
- Dockerfile 的 `RUN` 续行里**不能插 `#` 注释**：本机 docker 用 legacy builder（构建日志有 deprecation 警告），续行中的注释行可能把命令切断。注释一律放 `RUN` 块外面。
- xdebug 3.5.3 的 package.xml 约束为 `PHP 8.0.0 – 8.5.99`，8.3 在范围内（已核对，非猜测）。
- event 须在 sockets 之后加载：conf.d 按文件名字母序加载，`docker-php-ext-event.ini` 排在 `docker-php-ext-sockets.ini` 前面会报 `undefined symbol: socket_ce`。修法同 php8.5：改名为 `zzz-docker-php-ext-event.ini` 置后（8.3 下是否必然触发未单独验证，保留该改名是零风险的预防措施）。
- 时区统一用 Asia/Shanghai 规范名（PRC 是 tzdata backward 别名，新版本别再用）。
