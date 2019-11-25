# 安装和启用OWASP CRS

1. 下载和解压最新的OWASP CRS规则
```sh
# wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.0.0.tar.gz
# tar -xzvf v3.0.0.tar.gz
# mv owasp-modsecurity-crs-3.0.0 /usr/local
# cd /usr/local/owasp-modsecurity-crs-3.0.0
# sudo cp crs-setup.conf.example crs-setup.conf
```

2. 在ModSecurity主配置文件（/etc/nginx/modsec/main.conf）中引入CRS规则
```
# Include the recommended configuration
Include /etc/nginx/modsec/modsecurity.conf
# OWASP CRS v3 rules
Include /usr/local/owasp-modsecurity-crs-3.0.0/crs-setup.conf
Include /usr/local/owasp-modsecurity-crs-3.0.0/rules/*.conf
```

3. 重新加载nginx配置
```sh
# nginx -s reload
```

## 测试CRS规则

### 测试工具Nikto

我们使用Nikto扫描工具去生成恶意请求，包括对易受攻击的文件是否存在的探测、跨站脚本和其他类型的攻击，该工具还报告了哪些请求被应用程序接收，从而发现程序中的潜在漏洞。

运行下面的命令获取Nikto代码并对web应用进行攻击：
```sh
# git clone https://github.com/sullo/nikto
# cd nikto

# perl program/nikto.pl -h http://your.app.com

# curl -H "User-Agent: Nikto" http://your.app.com
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.15.12</center>
</body>
</html>
```

可以在启用CRS规则前执行一次攻击命令，启用后再执行一次攻击命令，对比下区别。

