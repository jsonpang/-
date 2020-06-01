# Promql查询

Promeql

PromQL遵循与Go相同的转义规则。在单引号或双引号中，反斜杠开始转义序列，转义序列后面可以跟a、b、f、n、r、t、v或\。可以使用八进制（nnn）或十六进制（xnn、unnnn和unnnnnn）提供特定字符。
在backticks中不处理转义。与Go不同，Prometheus不会在backticks中丢弃新行。
```
=：选择与提供的字符串完全相等的标签。
!=：选择不等于提供的字符串的标签。
=~：选择正则表达式与提供的字符串匹配的标签。
!~：选择与提供的字符串不匹配的正则表达式标签。
```
范围向量选择器

范围向量文本的工作方式与即时向量文本类似，只是它们从当前瞬间选择一个样本范围。在语法上，范围持续时间被附加在向量选择器末尾的方括号（[]）中，以指定应为每个结果范围向量元素获取时间值的时间间隔。

持续时间指定为一个数字，紧接着是下列单位之一：
```

s - seconds
m - minutes
h - hours
d - days
w - weeks
y - years
```
正则使用re2的语法
https://github.com/google/re2/wiki/Syntax