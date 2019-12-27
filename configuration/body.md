# 请求body和响应body限制

当应用服务的人群很大、功能很杂时，很有可能会有出现请求body或响应body内容很多的情况，ModSecurity如果全部进行处理则对性能有很大的影响，可以对处理body的大小做一个限制，这里介绍ModSecurity中配置的几个指令。

## 请求body

+ SecRequestBodyAccess, 配置是否处理请求body
  + 语法: SecRequestBodyAccess On|Off
+ SecRequestBodyLimit, 配置ModSecurity接受缓冲的最大请求body大小
  + 语法: SecRequestBodyLimit LIMIT_IN_BYTES
+ SecRequestBodyNoFilesLimit, 除了请求中传输文件的大小外，ModSecurity接受缓冲的最大请求body大小
  + 语法: SecRequestBodyNofilesLimit NUMBER_IN_BYTES
+ SecRequestBodyLimitAction, 当请求body超过限制时如何处理
  + 语法: SecRequestBodyLimitAction Reject|ProcessPartial
  + 设置为Reject表示超过限制后拒绝该请求（返回413状态码）
  + 设置为ProcessPartial表示只处理部分body，超过限制的部分直接通过

## 响应body

+ SecResponseBodyAccess, 配置是否处理响应body
  + 语法: SecResponseBodyAccess On|Off
+ SecResponseBodyMimeType, 配置处理哪种MIME类型的响应body
  + 语法: SecResponseBodyMimeType MIMETYPE MIMETYPE ...
  + 例子: SecResponseBodyMimeType text/plain text/html text/xml
+ SecResponseBodyLimit, 配置ModSecurity接受缓冲的最大响应body大小
  + 语法: SecResponseBodyLimit LIMIT_IN_BYTES
+ SecResponseBodyLimitAction, 当响应body超过限制时如何处理
  + 语法: SecResponseBodyLimitAction Reject|ProcessPartial
