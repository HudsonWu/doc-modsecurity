# 应用规则的阶段 (Processing Phases)

ModSecurity将规则应用在请求循环的5个阶段：
  + Request headers (REQUEST_HEADERS)
    + 此阶段的规则在请求头读取成功后会立即处理，此时请求的Body还没有读取
    + 适合此阶段的规则：request body读取之前、是否缓冲请求body、如何处理请求body
  + Request body (REQUEST_BODY)
    + 这是通用的输入分析阶段，大多数面向应用的规则应该放在这里
    + 为了可以访问request body阶段的数据，必须设置SecRequestBodyAccess为On
    + ModSecurity支持三种编码类型
      + application/x-www-form-urlencoded, 用来传输表单数据
      + multipart/form-data, 用来文件传输
      + text/xml, 用来传递xml数据
  + Response headers (RESPONSE_HEADERS)
    + 此阶段发生在响应头发送到客户端之前
    + 适合此阶段的规则：响应头发送到客户端之前观察响应、是否缓冲响应body
  + Response body (RESPONSE_BODY)
    + 这是通用的输出分析阶段
    + 此时，对响应body运行规则，前提是缓冲了body
    + 为了可以访问response body阶段的数据，必须设置SecResponseBodyAccess为On
  + Logging (LOGGING)
    + 此阶段在日志记录发生之前运行，此阶段的规则只能影响日志记录的方式

下图显示了标准的请求循环：
![request cycel](/images/phase.jpg)

+ 每个阶段可用的数据是累积的，也就是说当一个阶段一个阶段往下执行时，可用的数据是越来越多的
+ 规则是根据阶段执行的，即使两个规则在配置文件中相邻，如果不是同一阶段，是不会一个接一个地执行的
+ Logging阶段比较特殊，无论前面的阶段执行情况如何，Logging阶段都会在最后执行

阶段的定义，可以直接在规则中使用phase action或者使用SecDefaultAction指令：
```
SecDefaultAction "log,pass,phase:2,id:4"
SecRule REQUEST_HEADERS:Host "!^$" "deny,phase:1,id:5"
```

