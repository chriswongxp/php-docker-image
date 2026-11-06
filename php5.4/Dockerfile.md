# 镜像分发（阿里云 ACR，固定地址，公开仓库免登录拉取）

registry 地址（实例 ID+地域，开通后不变，无需再登阿里云后台查询）：
crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com

其他服务器部署（直接用长名，不做 tag 短名）：
docker pull crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php5.4:latest
docker run -d -e TZ="Asia/Shanghai" --name php5.4 --restart=always --network=static --ip 172.18.0.7 -v /srv/http/php:/var/www/html crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php5.4:latest

（--network/--ip/--volume 按各机器实际情况调整；验证三件事：php -v 是 5.4.45、date 是北京时间、nginx 机器发真实请求 200）

# Run container example（本机构建运行）:
docker run -d -e TZ="Asia/Shanghai" --name php5.4 --restart=always --network=static --ip 172.18.0.7 -v /srv/http/php:/var/www/html chriswongxp/php5.4

# 环境说明

- PHP 5.4.45（Remi SCL php54，centos:7 基础镜像），cli 路径 /opt/remi/php54/root/usr/bin/php，fpm 路径 /opt/remi/php54/root/usr/sbin/php-fpm，监听 0.0.0.0:9000（本机 nginx upstream fastcgi_pass）
- 用途：购途网等 php5.3+CI2 项目升级 PHP 5.4 的本地验证环境（生产为三方商用 PHP 5.4，定制修复 PHP2Shell）
- 配置目录：/opt/remi/php54/root/etc/php.ini、php-fpm.d/www.conf、php.d/

# 与 php5.3 容器（chriswongxp/php5.3）的对齐

- redis-2.2.7 pecl 源码编译（同版本）
- timezonedb-2015.6 pecl 源码编译（同版本）
- php.ini：max_input_vars = 10000、date.timezone = Asia/Shanghai（5.4 不认 PRC，见坑）、default_charset 置空（保持 5.3 行为，htmlspecialchars 默认 ISO-8859-1 与 5.3 逐字节一致）
- OPcache 禁用（remi 的 pecl-zendopcache 的 opcache.ini 已删，对齐 php5.3 无 OPcache）
- php-fpm：user www-data(1000) group users(982)，对齐宿主机 /srv/http/php 属主，避免挂载目录权限问题；pm dynamic 8/2/1/3、pm.status_path=/fpm-status（同 php5.3）
- 扩展与 php5.3 的 diff：仅缺 SQLite(v2)（remi 未打包 ext/sqlite 旧接口；业务 dbdriver=mysql，sqlite_* 仅第三方库可选驱动：lotusphp/PExcel缓存后端/CI2 sqlite 驱动，实际不走）。多出 calendar/gmp/mhash/mysqlnd/pcntl/readline/shmop/sysv*/wddx/xsl（remi 默认编译，无害）
- 网络诊断工具：ip（iproute 包，注意 centos7 包名不是 iproute2）、telnet、nslookup（bind-utils）、curl、ping、netstat（net-tools）

# yum 源（centos7 已 EOL）

- base/updates: mirrors.aliyun.com/centos-vault/7.9.2009
- remi-php54/remi-safe: mirrors.aliyun.com/remi/enterprise/7
- epel: mirrors.aliyun.com/epel/7（libc-client/libmcrypt/libzstd 依赖在此）

# 构建踩坑

- 官方 php:5.4 镜像 manifest 损坏（Docker Hub 与国内镜像源均拉取失败），webdevops/php-5.4 等第三方镜像拉取超时不可用，故 centos:7 + Remi php54 RPM + pecl 编译自建
- date.timezone = PRC 在 5.4 下报 "Invalid date.timezone value" 回退 UTC（5.3 认），必须用 Asia/Shanghai
- centos7 的 ip 命令包名是 iproute（Debian 系才叫 iproute2）
- fpm pool group 不能写 www-data（那是用户名，组是 users/982），否则 "cannot get gid for group 'www-data'" 启动失败
- groupadd users 报组已存在，用 groupmod -g 982 users
