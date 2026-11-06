# php-docker-image

## 镜像推送到阿里云 ACR

registry 固定地址（实例 ID + 地域，公开命名空间 chriswongxp，拉取免登录）：

```
crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com
```

推送前需登录（密码为阿里云容器镜像服务的固定访问密码）：

```
docker login --username=<阿里云账号全名> crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com
```

### php8.5

```
docker tag chriswongxp/php8.5:latest crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.5:latest
docker push crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.5:latest
```

### php7.4

```
docker tag chriswongxp/php7.4:latest crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php7.4:latest
docker push crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php7.4:latest
```

### php5.4

```
docker tag chriswongxp/php5.4:latest crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php5.4:latest
docker push crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php5.4:latest
```

其他服务器拉取直接用长名（详见各版本目录下的 Dockerfile.md）：

```
docker pull crpi-6sveuztmrd6gt9gg.cn-chengdu.personal.cr.aliyuncs.com/chriswongxp/php8.5:latest
```
