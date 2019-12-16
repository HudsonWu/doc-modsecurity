# 变量

## ARGS

+ ARGS是一个集合，可以单独使用，代表所有参数，包括POST payload；也可以跟随一个静态参数或一个正则表达式使用
+ ARGS_NAMES包含所有请求参数名，可以用来搜索希望检查的特定参数名
+ ARGS_GET，类似于ARGS，但只包含查询字符串参数（query string parameters）
+ ARGS_POST，类似于ARGS，但只包含来自POST body的参数

```
# 检查所有的请求参数
SecRule ARGS dirty "id:7"

# 只检查名为p的参数（通常，请求可以包含多个同名参数）
SecRule ARGS:p dirty "id:8"

# 检查所有请求参数，除了名为z的参数
SecRule ARGS|!ARGS:Z dirty "id:9"

# &是一个特殊的运算符，如果请求参数大于0，下面的规则将触发
SecRule &ARGS !^0$ "id:10"

# 检查所有名称以id_开头的参数
SecRule ARGS:/^id_/ dirty "id:11"

# 只列出授权的参数名，这个规则只允许两个参数名：p和a
SecRule ARGS_NAMES "!^(p|a)$" "id:12"
```

## ARGS_COMBINED_SIZE

所有请求参数数据的合并大小，文件被排除在计算之外。

```
# 检测长度超过2500字节的请求
SecRule ARGS_COMBINED_SIZE "@gt 2500" "id:13"
```

## AUTH_TYPE

此变量包含验证身份信息的方式，在反向代理中，如果身份验证是后端服务处理的，此信息不可用。

```
SecRule AUTH_TYPE "Basic" "id:14"
```

## ENV

提供对ModSecurity或其他服务器模块设置的环境变量访问的集合，需要一个参数来指定变量的名称。

```
# 设置环境变量
SecRule REQUEST_FILENAME "printenv" \
"phase:2,id:15,pass,setenv:tag=suspicious"

# 检查环境变量
SecRule ENV:tag "suspicious" "id:16"

# 从另外一个模块(mod_ssl)读取环境变量
SecRule TX:ANOMALY_SCORE "@gt 0" "phase:5,id:16,msg:'%{env.ssl_cipher}'"
```

## FILES

+ FILES, 初始文件名的集合。仅用在检查`multipart/form-data`的请求中，而且从请求body中提取出文件后才可用
+ FILES_COMBINED_SIZE, 请求body中传输文件的总大小，仅用在检查`multipart/form-data`请求中
+ FILES_NAMES, 包含用于文件上传的表单字段列表，仅用在检查`multipart/form-data`请求中

```
SecRule FILES "@rx \.conf$" "id:17"

SecRule FILES_COMBINED_SIZE "@gt 100000" "id:18"

SecRule FILES_NAMES "^upfile$" "id:19"
```

## FULL_REQUEST

+ FULL_REQUEST, 包含完整的请求，请求行、请求头、请求body（SecRequestBodyAccess被设置为On时可用）
+ FULL_REQUEST_LENGTH, FULL_REQUEST可用的字节数

```
SecRule FULL_REQUEST "User-Agent: ModSecurity Regression Tests" "id:21"

SecRule FULL_REQUEST_LENGTH "@eq 205" "id:21"
```

## GEO

GEO是由最后一个@geoLookup操作产生的结果的集合，该集合可用于匹配从IP地址或主机名查找的地理字段。

字段：
+ COUNTRY_CODE, 两个字符的国家代码，如：US、GB等
+ COUNTRY_CODE3, 最多三个字符的国家代码
+ COUNTRY_NAME, 完整的国家名
+ COUNTRY_CONTINENT, 国家所在洲的两个字符代码，如EU
+ REGION, 两个字符的区域代码，美国而言是州，中国而言是省
+ CITY, 如果database支持的话，代表城市名
+ POSTAL_CODE, 如果database支持的话，代表邮政编码
+ LATITUDE, 如果database支持的话，代表纬度
+ LANGITUDE, 如果database支持的话，代表经度

```
SecGeoLookupDb /usr/local/geo/data/GeoLiteCity.dat
...
SecRule REMOTE_ADDR "@geoLookup" "chain,id:22,drop,msg:'Non-GB IP address'"
SecRule GEO:COUNTRY_CODE "!@streq GB"
```

## HIGHEST_SEVERITY

该变量包含匹配到的所有最严重的规则，严重级别使用数字表示，可以用@lt等比较操作符一起使用。严重级别越高，值越低，值为255表示没有设置严重级别。

```
SecRule HIGHEST_SEVERITY "@le 2" "phase:2,id:23,deny,status:500,msg:'severity %{HIGHEST_SEVERITY}'"
```

## INBOUND_DATA_ERROR

当请求body超过SecRequestBodyLimit指令设置的大小时，该变量将被设置为1。你的策略应该始终要有一个规则来检查这个变量，可以根据误报率和你的默认策略来决定在触发规则后是拦截还是警告。

```
SecRule INBOUND_DATA_ERROR "@eq 1" "phase:1,id:24,t:none,log,pass,msg:'Request Body Larger than SecRequestBodyLimit Setting'"
```

## MATCHED_VAR

+ MATCHED_VAR, 此变量保存最近匹配的变量的值，类似于TX:0，但所有操作符都自动支持它，且不需要指定捕获操作
+ MATCHED_VARS, 与MATCHED_VAR类似，只是它是当前操作符检查的所有匹配项的集合
+ MATCHED_VAR_NAME, 这个变量保存匹配到的变量的全名
+ MATCHED_VARS_NAME, 与MATCH_VAR_NAME类似，只是它是当前操作符检查的所有匹配项的集合

```
SecRule ARGS pattern "chain,deny,id:25"
  SecRule MATCHED_VAR "further scrutiny"

SecRule ARGS pattern "chain,deny,id:26"
  SecRule MATCHED_VARS "@eq ARGS:param"

SecRule ARGS pattern "chain,deny,id:27"
  SecRule MATCHED_VAR_NAME "@eq ARGS:param"

SecRule ARGS pattern "chain,deny,id:28"
  SecRule MATCHED_VARS_NAME "@eq ARGS:param"
```

## MODSEC_BUILD

此变量保存ModSecurity构建号，此变量用于在使用仅在特定构建中可用的特性之前检查构建号。

```
SecRule MODSEC_BUILD "!@ge 02050102" "skipAfter:12345,id:29"
SecRule ARGS "@pm some key words" "id:12345,deny,status:500"
```

## TX

这是短暂的事务集合，用来存储数据片段、创建事务异常评分等。置于此集合中的变量仅在事务完成之前可用。

```
# 增加事务攻击得分
SecRule ARGS attack "phase:2,id:82,nolog,pass,setvar:TX.score=+5"

# 拦截评分过高的事务
SecRule TX:SCORE "@gt 20" "phase:2,id:83,log,deny"
```

TX集合中有一些变量名有特殊含义，不能自定义：
  + TX:0, 使用@rx或@pm操作符在capture action下匹配到的值
  + TX:1-TX:9, 使用@rx操作符在capture action下匹配到的子表达式值
  + TX:MSC_.*, ModSecurity处理标识
  + MSC_PCRE_LIMITS_EXCEEDED, 如果超过PCRE匹配限制，则设置为非0
