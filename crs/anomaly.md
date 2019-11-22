# Anomaly Scoring Detection Mode, 异常评分检测模型

CRS 3.x 默认使用了Anamoly Scoring Detection Mode（异常评分检测模型），区别于旧版本默认使用的Traditional Detection Mode（或者叫IDS/IPS Mode、Self-contained Mode，传统检测模型）。传统检测模型中，所有规则都是自包含的最基本操作模式，规则之间没有智能共享，每个规则也没有之前任何规则匹配的信息，在这种模式下，如果一个规则触发，将执行当前规则上指定的防御性操作和日志操作。

异常评分检测模型实现了协作检测与延迟拦截，通过将检测功能和拦截功能分离更改了规则的逻辑。这些规则并不会直接触发防御性动作，而是增加异常分数。此外，每个规则还将存储匹配的元数据，例如规则ID、攻击类别、匹配的位置和匹配的数据，以便进行日志记录。

使用哪种模型的配置在crs-setup.conf中；
```
# Default (Anamoly Mode)
SecDefaultAction "phase:1,log,auditlog,pass"
SecDefaultAction "phase:2,log,auditlog,pass"

# Updated To Enable Traditional Mode
SecDefaultAction "phase:1,log,auditlog,deny,status:403"
SecDefaultAction "phase:2,log,auditlog,deny,status:403"
```

## Anomaly Scoring Severity Levels, 异常评分严重级别

CRS使用可配置的异常评分模型来对攻击进行打分，每触发到定义的规则，就会相应增加攻击分数，如果该分数超过配置的异常阈值，对应的事务会被拦截。

每一条规则会有一个关联的严重级别，不同的严重级别对应不同的异常评分，在规则匹配到后进行异常分数的增加。下面是每一个严重级别的默认异常评分：

+ Critical, 异常评分5分，表示可能的应用程序攻击，主要由93x和94x类文件定义
+ Error, 异常评分4分，表示可能的数据泄露，主要由95x类文件定义
+ Warning, 异常评分3分，表示可能的恶意客户端，主要由91x类文件定义
+ Notice, 异常评分2分，表示可能违反协议的攻击，主要由92x类文件定义

可以在crs-setup.conf文件中调整评分数，不过通常没有必要这么做。

```
SecAction \
 "id:900100,\
  phase:1,\
  nolog,\
  pass,\
  t:none,\
  setvar:tx.critical_anomaly_score=5,\
  setvar:tx.error_anomaly_score=4,\
  setvar:tx.warning_anomaly_score=3,\
  setvar:tx.notice_anomaly_score=2"
```

默认情况下CRS会阻挡所有评分为5或更高的入站流量，这意味着任何触发Critical规则的入站流量都会被丢弃，触发三个或更多的Notice规则的入站流量也会被丢弃。可以在crs-setup.conf文件中调整异常评分阈值：

```
SecAction \
 "id:900110,\
  phase:1,\
  nolog,\
  pass,\
  t:none,\
  setvar:tx.inbound_anomaly_score_threshold=5,\
  setvar:tx.outbound_anomaly_score_threshold=4"
```


