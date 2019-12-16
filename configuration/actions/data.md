# Data Action

## status

指定与deny操作和redirect操作一起使用的响应状态码。

```
SecDefaultAction "phase:1,log,deny,id:145,status:403"
```

## xmlns

配置一个XML命名空间（namespace），它将用于XPath表达式的执行。

```
SecRule REQUEST_HEADERS:Content-Type "text/xml" "phase:1,id:147,pass,ctl:requestBodyProcessor=XML,ctl:requestBodyAccess=On, \
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
SecRule XML:/soap:Envelope/soap:Body/q1:getInput/id() "123" phase:2,deny,id:148
```
