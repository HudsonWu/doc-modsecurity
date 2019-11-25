# ModSecurity 3.0 and Nginx

和之前的版本不同，ModSecurity 3.0原生支持nginx。ModSecurity 3.0将核心功能放入了独立引擎`libmodsecurity`中，和其他组件的交互通过特定组件的连接器，Nginx而言，连接器叫做`Nginx Connector`。

![ModSecurity 3.0 架构](/images/architecture.png)

Nginx的ModSecurity动态模块包括`libmodsecurity`和`Nginx Connector`。

## 快速导航

+ [ModSecurity安装指南](mod-install.md)
+ [日志记录配置](log.md)
+ [OWASP核心规则集](/crs/)
+ [k8s中的ingress controller配置](ingress.md)


## 引用

+ [ModSecurity 3.0 and NGINX: Quick Start Guide](https://www.nginx.com/resources/library/modsecurity-3-nginx-quick-start-guide)
+ [ModSecurity源代码](https://github.com/SpiderLabs/ModSecurity)
+ [OWASP Core Rule Set源代码](https://github.com/SpiderLabs/owasp-modsecurity-crs)
+ [OWASP-CRS-Documentation](https://github.com/SpiderLabs/OWASP-CRS-Documentation)
