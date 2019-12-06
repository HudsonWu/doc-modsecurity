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

包含原始文件名的集合。仅用在检查`multipart/form-data`的请求中。从请求body中提取出文件后才可用。

```
SecRule FILES "@rx \.conf$" "id:17"
```
