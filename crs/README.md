# OWASP Core Rule Set, OWASP核心规则集

ModSecurity 3.0安装成功后，下一步就是安装ModSecurity规则集，规则集里定义了攻击模式以及应对策略。OWASP Core Rule Set（CRS）是一个社区维护的开源项目，包含拦截攻击的通用规则，包括SQL注入、跨站脚本攻击等，还集成了Project Honeypot以及机器人检测和工具扫描的规则。

OWASP Core Rule Set最初由 Ofer Shezaf在2006年发布，目前使用Apache Software License version 2 (ASLv2)授权，由Christian Folini领导的10名开发人员维护。CRS是一个通用的黑名单规则集，由已知用于攻击的代码段组成。CRS的3.0版本（通常称为CRS3）在2016年11月发布，它将误报数量减少了90%以上，推荐使用CRS3。

## CRS文件和目录结构

[CRS源代码](https://github.com/SpiderLabs/owasp-modsecurity-crs)

+ crs-setup.conf.example, CRS主配置文件，定义了异常评分阈值（anomaly scoring thresholds）、paranoia级别以及其他ModSecurity重要配置项
+ rules/, 规则目录
  + 90x 类文件, 排除误报
  + 91x 类文件, 检测恶意客户端，如扫描工具和机器人
  + 92x 类文件, 违反协议
  + 95x 类文件, 检测流出流量中的数据泄漏，暂不支持NGINX和NGINX Plus
  + .data 文件, 规则中用到的数据，比如crawlers-user-agents.data文件中包含一个扫描器会用到的User-Agent值的列表，这个文件被规则文件REQUEST-913-SCANNER-DETECTION.conf用来辨别扫描器和机器人

## 快速导航

+ [安装OWASP核心规则集](install.md)
+ [异常评分模型](anomaly.md)
+ [生产环境启用ModSecurity的建议](prod.md)
