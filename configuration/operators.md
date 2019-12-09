# Operators, 操作符

## rx

执行正则表达式匹配，默认操作符，没有明确指定操作符的规则使用@rx。

```
# 检测Nikto
SecRule REQUEST_HEADERS:User-Agent "@rx nikto" phase:1,id:173,t:lowercase

# 使用大小写不敏感模式检测nikto
SecRule REQUEST_HEADERS:User-Agent "@rx (?i)nikto" phase:1,id:174,t:none

# 默认使用@rx
SecRule REQUEST_HEADERS:User-Agent "(?i)nikto" "id:175"
```

正则表达式是交给[PCRE](http://www.pcre.org)处理的，ModSecurity使用下面的配置编译：
+ 整个输入作为单行，即使有换行符
+ 所有匹配都是大小写敏感的，如果希望执行不区分大小写的匹配，可以使用小写转换函数或强制不区分大小写匹配（在模式前加上(?i)修饰符）
+ 编译时设置了PCRE_DOTALL和PCRE_DOLLAR_ENDONLY，意味着单个点将匹配任何字符，包括换行符，而`$`不会匹配结尾的换行符

## pm

根据所需的输入值对提供的词组执行不区分大小写的匹配，这个操作符使用基于集合的匹配算法（Aho-Corasick），这意味着它将并行匹配任意数量的关键字。当需要匹配大量关键字时，该操作符的性能比正则表达式好得多。

从ModSecurity v2.6.0开始，该操作符支持snort/suricata内容样式，如"@pm A|42|C|44|F"。

```
# 通过查看user agent标识来检测可疑的客户端
SecRule REQUEST_HEADERS:User-Agent "@pm WebZIP WebCopier Webster WebStripper ... SiteSnagger ProWebWalker CheeseBot" "id:166"
```

## pmFromFile, pmf

这个操作符和@pm相同，不同之处在于它接受一个文件列表作为参数。

```
# 检测可疑客户端
SecRule REQUEST_HEADERS:User-Agent "@pmFromFile /path/to/blacklist1 blacklist2" "id:167"

# 准备自定义REMOTE_ADDR变量
SecAction "phase:1,id:168,nolog,pass,setvar:tx.REMOTE_ADDR=/%{REMOTE_ADDR}/"
# 检查REMOTE_ADDR是否在黑名单中
SecRule TX:REMOTE_ADDR "@pmFromFile blacklist.txt" "phase:1,id:169,deny,msg:'BlackListed IP address'"
```

blacklist.txt文件例子：
```
# ip-blacklist.txt contents:
# NOTE: All IPs must be prefixed/suffixed with "/" as the rules
#   will add in this character as a boundary to ensure
#   the entire IP is matched.
# SecAction "phase:1,id:170,pass,nolog,setvar:tx.remote_addr='/%{REMOTE_ADDR}/'"
/1.2.3.4/ 
/5.6.7.8/
```

读取的文件（后面称为短语文件，phrase files）要求：
+ 文件每行必须包含一个词语，行标记的结束符（包括LF和CRLF）将被删除、任何开头和结尾的空格将被删除，空行和注释行（以#开头）将被忽略。
+ 为了更简便地使用短语文件，可以使用文件的相对路径（使用包含规则的文件所在路径作为前缀）。
+ 匹配时不会检测边界，可能会出现误报，比如ip地址匹配，语句`1.2.3.4`将匹配`1.2.3.40`或`1.2.3.41`，为了避免误报，可以在规则和文件中自定义边界，比如使用`/1.2.3.4/`。

