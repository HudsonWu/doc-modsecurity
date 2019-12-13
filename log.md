# 日志管理

ModSecurity有两种类型的日志，一是审计日志（audit log），记录被阻挡事务的详细日志以及为什么被阻挡；二是调试日志（debug log），会记录ModSecurity操作日志的详细信息。

## 审计日志

审计日志是ModSecurity的主要日志，它记录所有的攻击，包括可能发生的攻击。审计日志被分成几个小节，下面列出了不同节代表的内容：

+ A, 审计日志头部，强制记录
+ B, 请求headers
+ C, 请求body
+ D, 保留
+ E, 响应body
+ F, 响应headers
+ G, 保留
+ H, trailer, 包含额外的数据
+ I, 压缩的请求body, 不包括文件
+ J, 上传文件信息
+ K, 匹配到的规则列表
+ Z, 审计日志尾部，强制记录

### 审计日志配置文件modsecurity.conf

+ SecAuditEngine
  + Off, 禁止审计日志
  + On, 记录所有事务
  + RelevantOnly, 只记录触发Warning/error的事务，
    或者匹配SecAuditLogRelevantStatus指令中声明的状态码的事务
+ SecAuditLogRelevantStatus, 如果SecAuditEngine被设置为RelevantOnly，这个指令控制哪种HTTP响应状态码会被日志记录。
  这个指令的值是基于正则表达式的，如"^(?:5|4(?!04))"表示记录除了404外的4xx和5xx的响应
+ SecAuditLogParts, 控制哪些小节被记录到日志中，例如取值为"ABIJDEFHZ"
  
## 调试日志

调试日志默认是关闭的，调试日志也是在modsecurity.conf文件中配置的，有两个配置指令：
+ SecDebugLog, 日志记录路径，如`SecDebugLog /var/log/modsec_debug.log`
+ SecDebugLogLevel, 日志记录等级（0-9）， 如`SecDebugLogLevel 4`

日志等级取值含义：
  + 0, 不记录调试日志
  + 1, errors，只记录错误（请求被拦截）的调试日志
  + 2, warnings，警告
  + 3, notices，通知
  + 4, 事务处理的细节
  + 5, 包括每条信息的处理信息
  + 9, 记录所有日志，包括详细的调试信息

级别在1-3的消息会复制到web服务器的错误日志中。大量的日志记录会对性能产生负面影响，如果非常关心性能，可以设置级别为0。日志记录级别的最佳值是3，不建议在生产环境中使用更高的日志记录级别。
