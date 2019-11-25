# 生产环境中部署ModSecurity

## 调优以最小化误报

误报(False Positives)是web应用防火墙(WAF)广泛使用最大的威胁，下面的调优基于Dr.Christian Folini的推荐，会减少误报。基本思想是以blocking模式运行ModSecurity，将异常评分设置为比较高的值，等熟练应用后，慢慢地降低这个值。

```
# 1. 在modsecurity.conf中设置以blocking模式运行
SecRuleEngine On

# 2. 确保审计日志处于开启状态

# 3. 在crs-setup.conf中将异常评分值设置的大一些
SecAction \
  "id:900110,\
  phase:1,\
  nolog,\
  pass,\
  t:none,\
  setvar:tx.inbound_anomaly_score_threshold=1000,\
  setvar:tx.outbound_anomaly_score_threshold=1000"

# 4. 监控审计日志的误报信息，可以使用下面的指令移除误报的规则
SecRemoveRuleByID rule-id

# 5. 如果运行一段时间后没有误报，降低一半的异常评分值，然后监控和处理误报，直到异常评分值到达默认值
```

## 禁止审计日志

审计日志默认启用，不过在生产环境中建议禁止审计日志，一是影响性能，二是磁盘消耗。禁止审计日志需要在modsecurity.conf文件中配置：
```
SecAuditEngine off
```

## 静态内容不启用ModSecurity

对于通过web服务器提供的静态内容，运行ModSecurity没有意义，可以使用location指令分别配置：
```
server {
  listen 80;
  
  location / {
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/main.conf;
    proxy_pass http://localhost:8085;
    proxy_set_header Host $host;
  }

  location ~ \.(git|jpg|png|jpeg|svg)$ {
    root /data/images;
  }
}
```

## 使用nginx减缓DDOS攻击以及速率限制的功能

ModSecurity动态模块当前并不支持CRS的减缓DDOS攻击的规则（REQUEST-912-DOS-PROTECTION.conf），但Nginx内置了速率限制和减缓DDOS的功能，下面的配置限制了每个独立IP每秒10个请求：
```
# limit_req_zone指令定义了速率限制的参数
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
  listen 80;

  location / {
    # limit_req在当前上下文启用速率限制
    limit_req zone=mylimit;

    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/main.conf;
    proxy_pass http://localhost:8085;
    proxy_set_header Host $host;
  }
}
```
