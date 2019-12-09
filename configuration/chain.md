# chain

将当前规则与紧随其后的规则连接起来，创建一个规则链，链式规则允许更复杂的处理逻辑。

规则链的匹配需要所有规则都匹配，破坏性动作（disruptive actions）、执行阶段（execution phases）、元数据动作（metadata actions，包括id、rev、msg、tag、severity、logdata）、skip以及skipAfter动作只能在第一个规则中指定。

```
# 没有Content-Length头部的POST请求将被拒绝
SecRule REQUEST_METHOD "^POST$" phase:1,chain,t:none,id:105
  SecRule &REQUEST_HEADERS:Content-Length "@eq 0" t:none
```
