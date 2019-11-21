# OWASP Core Rule Set

ModSecurity 3.0安装成功后，下一步就是安装ModSecurity规则集，规则集里定义了攻击模式以及应对策略。OWASP Core Rule Set（CRS）是一个社区维护的开源项目，包含阻止攻击的通用规则，包括SQL注入、跨站脚本攻击等，还集成了Project Honeypot以及检测机器人和扫描工具的规则。

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

## Anomaly Scoring Detection Mode (异常评分检测模型)

CRS 3.x 默认使用了Anamoly Scoring Detection Mode（异常评分检测模型），区别于旧版本默认使用的Traditional Detection Mode（或者叫IDS/IPS Mode，传统检测模型）。传统检测模型中，所有规则都是自包含的最基本操作模式，规则之间没有智能共享，每个规则也没有之前任何规则匹配的信息，在这种模式下，如果一个规则触发，将执行当前规则上指定的防御性操作和日志操作。

异常评分检测模型实现了协作检测与延迟拦截，通过将检测功能和拦截功能分离更改了规则的逻辑。这些规则并不会直接触发防御性动作，而是增加异常分数。此外，每个规则还将存储匹配的元数据，例如规则ID、攻击类别、匹配的位置和匹配的数据，以便进行日志记录。

使用哪种模型的配置在crs-setup.conf中；
```
# Default (Anamoly Mode)
SecDefaultAction "phase:2,pass,log"

# Updated To Enable Traditional Mode
SecDefaultAction "phase:2,deny,status:403,log"
```

CRS使用可配置的异常评分模型来对攻击进行打分，每触发到定义的规则，就会相应增加攻击分数，如果该分数超过配置的异常阈值，该事务会被阻挡。异常级别如下：

+ Critical, 异常评分5分，表示可能的应用程序攻击，主要由93x和94x类文件定义
+ Error, 异常评分4分，表示可能的数据泄露，主要由95x类文件定义
+ Warning, 异常评分3分，表示可能的恶意客户端，主要由91x类文件定义
+ Notice, 异常评分2分，表示可能违反协议的攻击，主要由92x类文件定义

默认情况下CRS会阻挡所有评分为5或更高的入站流量，这意味着任何触发Critical规则的入站流量都会被丢弃，触发三个或更多的Notice规则的入站流量也会被丢弃。


