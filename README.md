# ModSecurity 3.0 and Nginx

ModSecurity是一个开源、跨平台的web应用防火墙(WAF)，由Trustwave's SpiderLabs维护。它有一个健壮的基于事件的编程语言，提供了对web应用程序一系列攻击的保护，而且还可以进行HTTP流量监控、日志记录和实时分析。

ModSecurity 3.0是一个全新的版本，摆脱了apache的阴影，更加独立开放，支持很多web服务器，包括apache、nginx、iis。ModSecurity 3.0将核心功能放入了独立引擎`libmodsecurity`中，和web服务器等通过特定组件的连接器通信，Nginx而言，连接器叫做`Nginx Connector`。


![ModSecurity 3.0 架构](/images/architecture.png)

Nginx的ModSecurity动态模块包括`libmodsecurity`和`Nginx Connector`。

## 快速导航

+ [ModSecurity安装指南](mod-install.md)
+ [ModSecurity配置指令介绍](/configuration/)
+ [日志配置管理](log.md)
+ [OWASP核心规则集](/crs/)
+ [k8s中的ingress controller配置](ingress.md)


## 引用

+ [nginx官方提供的阅读材料: 《ModSecurity 3.0 and NGINX: Quick Start Guide》](https://www.nginx.com/resources/library/modsecurity-3-nginx-quick-start-guide)
+ [ModSecurity文档](https://github.com/SpiderLabs/ModSecurity/wiki)
+ [OWASP Core Rule Set规则代码](https://github.com/SpiderLabs/owasp-modsecurity-crs)
+ [OWASP-CRS-Documentation, 文档介绍](https://github.com/SpiderLabs/OWASP-CRS-Documentation)
