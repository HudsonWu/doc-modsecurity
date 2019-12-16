# Non-disruptive action group

## append

需要启用`SecContentInjection`指令，将作为参数给出的文本附加到响应body的末尾，附加时不会进行内容类型（Content type）检查，这意味着使用任何内容注入操作之前，你需要检查下响应的内容类型是否适合注入。

```
SecRule RESPONSE_CONTENT_TYPE  "^text/html" "nolog,id:99,pass,append:'<hr>Footer'"
```

## auditlog

将事务记录到审计日志中。

```
SecRule REMOTE_ADDR "^192\.168\.1\.100$" auditlog,phase:1,id:100,allow
```

## capture

当与正则表达式操作符(@rx)一起使用时，capture操作会将正则捕获到的数据复制到事务变量集合中。

在成功的模式匹配中最多复制10个捕获，每个捕获的名称由0到9的数字组成。TX.0变量总是包含正则匹配的整个区域，所有其他变量都包含捕获的值，其顺序和捕获括号在正则中出现的顺序相同。

```
SecRule REQUEST_BODY "^username=(\w{25,})" phase:2,capture,t:none,chain,id:105
  SecRule TX:1 "(?:(?:a(dmin|nonymous)))"
```

## ctl

事务执行过程中改变modsecurity配置(transient)，任何改变之影响执行该操作的事务，默认配置以及并行运行的其他事务不受影响。

```
# 将Content-Type为"text/xml"的请求解析为XML
SecRule REQUEST_CONTENT_TYPE ^text/xml "nolog,pass,id:106,ctl:requestBodyProcessor=XML"

# 列出指定规则的用户参数
SecRule REQUEST_URI "@beginsWith /index.php" "phase:1,t:none,pass, \
  nolog,ctl:ruleRemoveTargetById:981260;ARGS:user"
```

支持以下配置项：
  + auditEngine
  + auditLogParts
  + debugLogLevel
  + forceRequestBodyVariable
  + requestBodyAccess
  + requestBodyLimit
  + requestBodyProcessor
  + responseBodyAccess
  + responseBodyLimit
  + ruleEngine
  + ruleRemoveById
  + ruleRemoveByMsg
  + ruleRemoveByTag
  + ruleRemoveTargetById
  + ruleRemoveTargetByMsg
  + ruleRemoveTargetByTag
  + hashEngine
  + hashEnforcement

除了`requestBodyProcessor`和`forceRequestBodyVariable`设置之外，每个配置项都对应一个配置指令，其用法是相同的。

`requestBodyProcessor`项允许你配置请求体处理器，默认情况下，ModSecurity分别使用`URLENCODED`和`MULTIPART`处理器来处理`application/x-www-form-urlencoded`和`multipart/form-data`类型的数据。`JSON`和`XML`处理器也受支持，但必须明确声明。

如果解析过程中发生错误，请求体处理器不会中断事务，它们将设置变量`REQBODY_PROCESSOR_ERROR`和`REQBODY_PROCESSOR_ERROR_MSG`。应该在`REQUEST_BODY`阶段检查这些变量并采取适当的操作。`forceRequestBodyVariables`项允许你配置`REQUEST_BODY`变量，以便在没有配置请求体处理器时设置它，这允许检查位置类型的请求体。

## expirevar

配置一个集合变量，使其在给定时间段后过期（以秒为单位）。

```
SecRule REQUEST_COOKIES:JSESSIONID "!^$" "nolog,phase:1,id:114,pass,setsid:%{REQUEST_COOKIES:JSESSIONID}"
SecRule REQUEST_URI "^/cgi-bin/script\.pl" "phase:2,id:115,t:none,t:lowercase,t:normalizePath,log,allow,setvar:session.suspicious=1,expirevar:session.suspicious=3600,phase:1"
```

## deprecatevar

数值随时间递减，只适用于存储在持久存储器中的变量。

```
# 每300秒减少60
SecAction phase:5,id:108,nolog,pass,deprecatevar:SESSION.score=60/300
```

计数器的值总是正的，与expirevar不同，deprecate操作必须在每个请求上执行。

## initcol

通过从存储中加载数据或在内存中创建新集合来初始化指定的持久集合。

```
SecAction phase:1,id:116,nolog,pass,initcol:ip=%{REMOTE_ADDR}
```

在执行initcol操作时，集合按需加载到内存中，只有在事务处理过程中对集合进行了更改时，才会将其持久化。

## multiMatch

如果启用，ModSecurity将对每个目标执行多个操作符调用，在每次执行反规避(anti-evasion)转换之前和之后。

```
SecRule ARGS "attack" "phase:1,log,deny,id:119,t:removeNulls,t:lowercase,multiMatch"
```

通常，每个规则只检查变量一次，并且在完成所有转换函数后才检查变量。使用multiMatch后，在每个改变输入的转换函数之前和之后，都会根据操作符检查变量。

## t

用来指定转换管道，这个管道在规则匹配之前转换规则中用到的每一个变量值。在SecRule中指定的转换函数都将被添加到SecDefaultAction中指定的前一个转换函数中，建议在规则中始终使用t:none，这可以防止它们依赖于默认配置。

```
SecRule ARGS "(asfunction|javascript|vbscript|data|mocha|livescript):" "id:146,t:none,t:htmlEntityDecode,t:lowercase,t:removeNulls,t:removeWhitespace"
```

