# Flow Actions, 流操作

## chain, 规则链

将当前规则与紧随其后的规则连接起来，创建一个规则链，链式规则允许更复杂的处理逻辑。

规则链的匹配需要所有规则都匹配。破坏性操作（disruptive actions）、执行阶段（execution phases）、元数据操作（metadata actions，包括id、rev、msg、tag、severity、logdata）、skip以及skipAfter操作只能在第一个规则中指定。

```
# 没有Content-Length头部的POST请求将被拒绝
SecRule REQUEST_METHOD "^POST$" phase:1,chain,t:none,id:105
  SecRule &REQUEST_HEADERS:Content-Length "@eq 0" t:none
```

下面的指令可用于规则链；
  + SecAction
  + SecRule
  + SecRuleScript

特殊的规则控制在规则链中操作的使用：
  + 任何影响规则流的操作，例如破坏性操作、skip、skipAfter，只能定义在第一条规则，且整个链匹配时才会执行。
  + 非破坏性操作可在任何规则中定义，包含该操作的规则匹配到时就会执行，不需要整个链匹配。
  + 元数据操作只能定义在第一条规则中。

## skip

成功匹配后跳过一个或多个规则（或规则链）。

skip操作只在当前处理阶段有效。

```
SecRule REMOTE_ADDR "^127\.0\.0\.1$" "phase:1,skip:1,id:141"

# 当REMOTE_ADDR值为127.0.0.1时，这条规则会跳过
SecRule &REQUEST_HEADERS:Accept "@eq 0" "phase:1,id:142,deny,msg:'Request Missing an Accept Header'"
```

## skipAfter

成功匹配后跳过一个或多个规则（或规则链），跳到指定ID规则或者`SecMarker`指令位置后的第一条规则继续执行。

skipAfter操作只在当前处理阶段有效。

```
SecRule REMOTE_ADDR "^127\.0\.0\.1$" "phase:1,id:143,skipAfter:IGNORE_LOCALHOST" 
SecRule &REQUEST_HEADERS:Accept "@eq 0" "phase:1,deny,id:144,msg:'Request Missing an Accept Header'" 
SecMarker IGNORE_LOCALHOST
```
