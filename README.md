# ModSecurity 3.0 and Nginx

和之前的版本不同，ModSecurity 3.0原生支持nginx。ModSecurity 3.0将核心功能放入了独立引擎`libmodsecurity`中，和其他组件的交互通过特定组件的连接器，Nginx而言，连接器叫做`Nginx Connector`。

![ModSecurity 3.0 架构](/images/architecture.png)

Nginx的ModSecurity动态模块包括`libmodsecurity`和`Nginx Connector`。

## 引用

+ [ModSecurity 3.0 and NGINX: Quick Start Guide](https://www.nginx.com/resources/library/modsecurity-3-nginx-quick-start-guide)
+ [ModSecurity源代码](https://github.com/SpiderLabs/ModSecurity)
+ [OWASP Core Rule Set源代码](https://github.com/SpiderLabs/owasp-modsecurity-crs)
