# k8s中的ingress controller配置

## 在ConfigMap中全局配置

```
# 开启modsecurity模块
enable-modsecurity: "true"

# 启用owasp规则
enable-owasp-modsecurity-crs: "true"
```

## 在ingress资源的annotation中配置

```
# 开启modsecurity模块
nginx.ingress.kubernetes.io/enable-modsecurity: "true"

# 启用owasp规则
nginx.ingress.kubernetes.io/enable-owasp-core-rules: "true"

# 自定义规则
# 如果同时开启了owasp-core-rules, 只有自定义规则会生效
nginx.ingress.kubernetes.io/modsecurity-snippet: |
SecRuleEngine On
SecRequestBodyAccess On
SecAuditLogParts ABIJDEFHZ
SecAuditLog /var/log/modsec_audit.log
SecDebugLog /tmp/modsec_debug.log
SecRule REQUEST_HEADERS:User-Agent \"fern-scanner\" \"log,deny,id:107,status:403,msg:\'Fern Scanner Identified\'\"

# 使用include语句使owasp-core-rules也生效
nginx.ingress.kubernetes.io/modsecurity-snippet: |
Include /etc/nginx/owasp-modsecurity-crs/nginx-modsecurity.conf
Include /etc/nginx/modsecurity/modsecurity.conf
```
