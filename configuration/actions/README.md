# Actions, 操作

+ [破坏性操作 (Disruptive actions)](disruptive.md)
  + 很多情况下意味着拦截事务
  + allow操作也属于此类别，但它的作用与拦截相反
  + 每个规则只能有一个破坏性操作，如果有多个，则最后一个会生效
  + 规则链(chain)中破坏性操作只能出现在第一个规则中
  + 如果SecRuleEngine设置为DetectionOnly，破坏性操作不会被执行
  + 如果想要使用allow action去创建异常/白名单规则(exception/whitelisting rules)，
    应该添加`ctl:ruleEngine=On`操作去执行
+ [非破坏性操作 (Non-disruptive actions)](non-disruptive.md)
  + 这类操作不会也不能影响规则的处理过程
  + 例如，设置一个变量或者变更变量的值
  + 非破坏性操作可以出现在任何规则中，包括规则链的每条规则
+ [流操作 (Flow actions)](flow.md)
  + 这些操作影响规则流，例如skip、skipAfter
+ [元数据操作 (Meta-data actions)](metadata.md)
  + 用于提供规则的更多信息，如id、rev、severity、msg
+ [数据操作 (Data actions)](data.md)
  + 不是真正的操作，只是作为容器保存其他操作使用到的数据
  + 例如，status操作保存了拦截时用到的status数据
  
