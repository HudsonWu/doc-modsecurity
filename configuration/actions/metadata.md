# metadata actions, 元数据操作

## id

分配unique ID给rule或者chain，ModSecurity从2.7开始，id action强制要有，且必须是数值。

```
SecRule &REQUEST_HEADERS:Host "@eq 0" "log,id:60008,severity:2,msg:'Request Missing a Host Header'"
```

下面是id值范围的分配情况：
+ 1-99999: 本地使用，内部使用，不要将此范围用于分发给其他人的规则
+ 10,0000-19,9999: 保留给Oracle发布的规则
+ 20,0000-29,9999: 保留给Comodo发布的规则
+ 30,0000-39,9999: 保留给gotroot.com发布的规则
+ 40,0000-41,9999: 未使用
+ 42,0000-42,9999: 保留给ScallyWhack
+ 43,0000-43,9999: 保留给Flameeyes
+ 44,0000-59,9999: 未使用
+ 60,0000-69,9999: 保留给Akamai
+ 70,0000-79,9999: 保留给Ristic
+ 90,0000-99,9999: 保留给OWASP ModSecurity Core Rule Set项目
+ 100,0000-100,9999: 保留给Redhat安全团队发布的规则
+ 101,0000-199,9999: 未使用
+ 200,0000-299,9999: 保留给Trustwave公司的SpiderLabs研究团队
+ 300,0000-399,9999: 保留给Akamai
+ 400,0000-409,9999: 保留给AviNetworks
+ 410,0000-419,9999: 保留给Fastly
+ 420,0000-429,9999: 保留给CMS-Garden
+ 430,0000-430,0999: 保留给Ensim.hu
+ 430,1000-1999,9999: 未使用
+ 2000,0000-2199,9999: 保留给Trustwave公司的SpiderLabs研究团队
+ 2200,0000以及以上: 未使用

## accuracy

指定误报或漏报的相关准确度级别，该值是一个基于数字刻度的字符串，范围为1-9，9准确度很高，1有很多误报。

```
SecRule REQUEST_FILENAME|ARGS_NAMES|ARGS|XML:/* "\bgetparentfolder\b" \
    "phase:2,ver:'CRS/2.2.4,accuracy:'9',maturity:'9',capture,t:none,t:htmlEntityDecode,t:compressWhiteSpace,t:lowercase,ctl:auditLogParts=+E,block,msg:'Cross-site Scripting (XSS) Attack',id:'958016',tag:'WEB_ATTACK/XSS',tag:'WASCTC/WASC-8',tag:'WASCTC/WASC-22',tag:'OWASP_TOP_10/A2',tag:'OWASP_AppSensor/IE1',tag:'PCI/6.5.1',logdata:'% \
{TX.0}',severity:'2',setvar:'tx.msg=%{rule.msg}',setvar:tx.xss_score=+%{tx.critical_anomaly_score},setvar:tx.anomaly_score=+%{tx.critical_anomaly_score},setvar:tx.%{rule.id}-WEB_ATTACK/XSS-%{matched_var_name}=%{tx.0}"
```

## msg

将自定义消息添加到规则（rule）或者链（chain），该消息将与警报一起被记录。

```
SecRule &REQUEST_HEADERS:Host "@eq 0" "log,id:60008,severity:2,msg:'Request Missing a Host Header'"
```

## tag

给rule或chain分配标签。标签的作用是使事件的自动分类更容易，同一规则可以指定多个标签，使用`/`创建类别层级结构。

```
SecRule REQUEST_FILENAME|ARGS_NAMES|ARGS|XML:/* "\bgetparentfolder\b" \
    "phase:2,rev:'2.1.3',capture,t:none,t:htmlEntityDecode,t:compressWhiteSpace,t:lowercase,ctl:auditLogParts=+E,block,msg:'Cross-site Scripting (XSS) Attack',id:'958016',tag:'WEB_ATTACK/XSS',tag:'WASCTC/WASC-8',tag:'WASCTC/WASC-22',tag:'OWASP_TOP_10/A2',tag:'OWASP_AppSensor/IE1',tag:'PCI/6.5.1',logdata:'% \
{TX.0}',severity:'2',setvar:'tx.msg=%{rule.msg}',setvar:tx.xss_score=+%{tx.critical_anomaly_score},setvar:tx.anomaly_score=+%{tx.critical_anomaly_score},setvar:tx.%{rule.id}-WEB_ATTACK/XSS-%{matched_var_name}=%{tx.0}"
```

## severity

指定规则严重性级别，可以使用数值或文本值指定严重级别，但应该始终使用文本值，因为数值不好记忆。从2.5.0版本开始，数值的使用被禁止了。

```
SecRule REQUEST_METHOD "^PUT$" "id:340002,rev:1,severity:CRITICAL,msg:'Restricted HTTP function'"
```

ModSecurity中的严重程度值遵循syslog的数值范围（0是最严重的），以下数据由OWASP ModSecurity核心规则集使用：
+ 0, EMERGENCY, 由入站攻击（inbound attack）和出站泄露（outbound leakage）的异常评分数据相关性产生
+ 1, ALERT, 由入站攻击（inbound attach）和出站应用层错误（outbound application level error）相关性产生
+ 2, CRITICAL, 异常评分5，是没有相关性的最高严重级别，通常由web攻击规则（40x类文件）生成
+ 3, ERROR, 异常评分4，主要由出站泄露规则（50类文件）产生
+ 4, WARNING, 异常评分3，由恶意客户端规则（35类文件）产生
+ 5, NOTICE, 异常评分2，，由协议策略和异常文件产生
+ 6, INFO
+ 7, DEBUG


## logdata

记录数据片段到警报信息的一部分。

```
SecRule ARGS:p "@rx <script>" "phase:2,id:118,log,pass,logdata:%{MATCHED_VAR}"
```


## rev

指定规则修订版本（revision），与id action一起使用提供规则已更改的指示。

```
SecRule REQUEST_FILENAME|ARGS_NAMES|ARGS|XML:/* "(?:(?:[\;\|\`]\W*?\bcc|\b(wget|curl))\b|\/cc(?:[\'\"\|\;\`\-\s]|$))" \
    "phase:2,rev:'2.1.3',capture,t:none,t:normalizePath,t:lowercase,ctl:auditLogParts=+E,block,msg:'System Command Injection',id:'950907',tag:'WEB_ATTACK/COMMAND_INJECTION',tag:'WASCTC/WASC-31',tag:'OWASP_TOP_10/A1',tag:'PCI/6.5.2',logdata:'%{TX.0}',severity:'2',setvar:'tx.msg=%{rule.msg}',setvar:tx.anomaly_score=+%{tx.critical_anomaly_score},setvar:tx.command_injection_score=+%{tx.critical_anomaly_score},setvar:tx.%{rule.id}-WEB_ATTACK/COMMAND_INJECTION-%{matched_var_name}=%{tx.0},skipAfter:END_COMMAND_INJECTION1"
```

