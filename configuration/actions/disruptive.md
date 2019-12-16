# Disruptive Actions, 破坏性操作

## allow

成功匹配后停止规则处理并允许事务继续。

Modsecurity 2.5之前，allow操作只影响当前阶段(phase)，从v2.5.0开始，allow操作有更细粒度的控制：
  + 只有allow自身时，将影响整个事务，匹配后停止处理当前阶段，同时跳过其他阶段（除了logging阶段） 
  + 如果为`allow:phase`，匹配后停止处理当前阶段，其他阶段不影响
  + 如果为`allow:request`，匹配后停止处理当前阶段，下一阶段直接跳到`RESPONSE_HEADERS`

```
# 对指定ip允许不受限制的访问
SecRule REMOTE_ADDR "^192\.168\.1\.100$" phase:1,id:95,nolog,allow

# 匹配后不处理request, 但要处理response
SecAction phase:1,allow:request,id:96

# 匹配后不处理整个事务(request和response)
SecAction phase:1,allow,id:97

# 允许response通过
SecAction phase:3,allow,id:98
```

## block

执行前面`SecDefaultAction`指令定义的破坏性操作。

这个操作本质上是一个占位符(placeholder)，被规则编写者用来请求一个拦截操作，但没有指定如何拦截。其思想是这样的决策最好留给用户来管理，同时允许用户在需要时覆盖。

```
# 指定怎样拦截
SecDefaultAction phase:2,deny,id:101,status:403,log,auditlog

# 检测想要拦截的攻击
SecRule ARGS attack1 phase:2,block,id:102

# 检测只想发出警告的攻击
SecRule ARGS attack2 phase:2,pass,id:103
```

可以使用`SecRuleUpdateActionById`指令覆盖默认拦截规则，在以下三种场景下有用：
  + 如果一个规则直接硬编码拦截，而你希望它使用你定义的策略
  + 如果一个规则被编写为拦截，而你想要它只是发出警告
  + 如果一个规则被编写为只发出警告，而你想要它被拦截

```
# 指定怎样拦截
SecDefaultAction phase:2,deny,status:403,log,auditlog,id:104

# 检测攻击并拦截
SecRule ARGS attack1 phase:2,id:1,deny

# 改变规则拦截方式为你定义的策略
SecRuleUpdateActionById 1 block
```

## deny

停止规则处理并拦截事务。

```
SecRule REQUEST_HEADERS:User-Agent "nikto" "log,deny,id:107,msg:'Nikto Scanners Identified'"
```

## drop

通过发送一个FIN包，立即关闭TCP连接。

```
SecAction phase:1,id:109,initcol:ip=%{REMOTE_ADDR},nolog
SecRule ARGS:login "!^$" "nolog,phase:1,id:110,setvar:ip.auth_attempt=+1,deprecatevar:ip.auth_attempt=25/120"
SecRule IP:AUTH_ATTEMPT "@gt 25" "log,drop,phase:1,id:111,msg:'Possible Brute Force Attack'"
```

这个操作在响应暴力攻击(Brute Force)和拒绝服务攻击(Denial of Service)时非常有用。因为在这两种情况下，你都希望最小化网络带宽和返回给客户机的数据。
