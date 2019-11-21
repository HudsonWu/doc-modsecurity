# 开源nginx下modsecurity安装指南

+ [nginx安装](http://nginx.org/en/linux_packages.html#mainline)

Nginx 1.11.5之后，不再需要完整编译到Nginx二进制文件中，可以独立编译动态模块。接下来的ModSecurity安装假设你已经安装了Nginx，且版本不低于1.11.5。

1. 安装依赖包

```sh
$ apt-get install -y apt-utils autoconf automake build-essential \
git libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre++-dev \
libtool libxml2-dev libyajl-dev pkgconf wget zlib1g-dev
```

2. 下载和编译libmodsecurity

```sh
$ git clone --depth 1 -b v3/master --single-branch \
https://github.com/SpiderLabs/ModSecurity

$ cd ModSecurity
$ git submodule init
$ git submodule update
$ ./build.sh
$ ./configure
$ make
$ make install
```

编译过程如果出现如下报错，可以忽略：
```
fatal: No names found, cannot describe anything.
```

3. 下载Nginx Connector并作为Nginx的动态模块进行编译

```sh
$ git clone --depth 1 https://github.com/SpiderLabs/Modsecurity-nginx.git

# 查看nginx版本
$ nginx -v
nginx version: nginx/1.13.7

# 下载对应版本的nginx源代码
# 即使编译动态模块, 也需要完整的源代码
$ wget http://nginx.org/download/nginx-1.13.7.tar.gz
$ tar zxvf nginx-1.13.7.tar.gz

# 编译动态模块
$ cd nginx-1.13.7
$ ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
$ make modules
$ cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
```

4. 启用Nginx ModSecurity Connector动态模块

在/etc/nginx/nginx.conf文件的最外层添加下面的配置
```
load_module modules/ngx_http_modsecurity_module.so;
```

## ModSecurity使用举例

准备工作：
```sh
# 创建ModSecurity配置目录
$ mkdir /etc/nginx/modsec

# 下载推荐配置
$ cd /etc/nginx/modsec
$ wget https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended
$ mv modsecurity.conf-recommended modsecurity.conf
```

/etc/nginx/modsec/modsecurity.conf
```
# SecRuleEngine DetectionOnly
SecRuleEngine On
```

/etc/nginx/modsec/main.conf
```
# Include the recommended configuration
Include /etc/nginx/modsec/modsecurity.conf
# A test rule
SecRule ARGS:testparam "@contains test" "id:1234,deny,log,status:403"
```

/etc/nginx/conf.d/proxy.conf
```
server {
  listen 80;

  modsecurity on;
  modsecurity_rules_file /etc/nginx/modsec/main.conf;

  location / {
   proxy_pass http://localhost:8085;
   proxy_set_header Host $host;
  }
}
```

验证命令
```
$ curl -D - http://localhost/foo?testparam=thisisatestofmodsecurity
HTTP/1.1 403 Forbidden
Server: nginx/1.11.10
Date: Wed, 3 May 2017 09:00:48 GMT
Content-Type: text/html
Content-Length: 170
Connection: keep-alive
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr/><center>nginx/1.11.10</center>
</body>
</html>
```
