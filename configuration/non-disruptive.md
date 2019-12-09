# Non-disruptive action group

## t

用来指定转换管道，这个管道在规则匹配之前转换规则中用到的每一个变量值。在SecRule中指定的转换函数都将被添加到SecDefaultAction中指定的前一个转换函数中，建议在规则中始终使用t:none，这可以防止它们依赖于默认配置。

```
SecRule ARGS "(asfunction|javascript|vbscript|data|mocha|livescript):" "id:146,t:none,t:htmlEntityDecode,t:lowercase,t:removeNulls,t:removeWhitespace"
```

