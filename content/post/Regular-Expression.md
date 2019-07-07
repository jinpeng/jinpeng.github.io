---
title: Regular Expression
date: 2018-05-09 08:37:21
tags:
---

1966年，基于pattern的早期NLP系统ELIZA，可以和人进行简单的对话，并且有人认为ELIZA真的可以理解他。在ELIZA和后来的一些给予pattern的chatbots系统中，以及其他很多NLP任务中，正则表达式（regular expression）都是一个基本的但重要的工具。例如：信息抽取（information extraction），包括现在的很多爬虫，需要采集特定信息，也会使用正则表达式与XPath、CSS selector等一起使用，从爬取的网页里抽取信息；还有一些日历应用，可以从email、短信、IM消息、社交网络等文本中抽取时间、地点、事件等信息，自动添加到日历中。又例如：文本正则化（text normalization），需要对文本进行tokenization，对于中文要分词，对于英文也需要识别被空格分隔的几部分组成的词，还需要把缩写展开，识别表情符号（emoji）或者twitter的hashtag等；文本正则化还包括词形还原（lemmatization）或者简化版词形还原——词干提取（stemming），后者常用于搜索索引。

正则表达式（regular expression）是用于描述字符串集合的一种代数符号（algebraic notations），常用于在文档集中检索符合特定模式的文本。比如：Unix命令grep就是使用正则表达式进行文本检索的常用工具。

#### 基本模式

简单字符序列，例如：/Bufferfly/，匹配完整的字符串，匹配是大小写敏感的。

方括号[]是一种disjunction，例如：/[Ww]/匹配的是大写或者小写w；/[0123456789]/匹配的是任意单个数字（digit）。

使用范围（range）可以简化一些定义明确序列的字符disjunction，例如：/[0-9]/相当于/[0123456789]/，/[A-D]/相当于/[ABCD]/。

否定符号（^），在方括号内，当^紧跟在左方括号后面时，表示否定（negation），匹配的是与后面的字符串不相同的字符。例如：/\[^A-Z\]/表示非大写字母。但当^不是紧跟在左方括号后面时，只表示^字符本身。

通配符（.），表示任意字符，当要表示period字符本身时，需要前置转义符号（\），例如：/\[\.A\]/表示period或者大写A。

可选符号（?）是一种计数器，表示它前面的字符是可选的，出现的次数是0或者1。例如：/colou?r/可以匹配color或者colour两种拼写方法，其中的?修饰的是字符u。

Kleene *（读作"cleany star"）也是计数器，表示它前面的字符出现次数是0或者n，n是正整数。例如：/oo\*h/可以匹配oh, ooh, oooh...

Kleene + （读作"cleany plus"）也是计数器，表示它前面的字符出现次数是1或者n，n是正整数。例如：/[0-9]+/匹配数字序列。

锚点（anchor）用于定位字符串，^表示行首，$表示行尾，\b匹配词边界，\B匹配无边界。例如：/^The dong\\.\$/完全匹配字符串"The dog."的行；/\bthe\b/匹配"the"，但不匹配"other"。其中词的含义是编程语言的含义，由数字、字母和下划线组成的序列，而不是英语中的单词。比如：/\b99\b/匹配"There are 99 bottles."和"\$99"，但不匹配"There are 299 bottles."

#### Disjunction, Grouping and Precednece

Disjunction，除了方括号，pipe符号（|）是另一种disjunction符号。例如：/cat|dog/匹配包含"cat"或者"dog"的字符串。

优先级（precedence），作为代数符号，正则表达式也有不同的优先级。比如：简单字符序列优先级高于pipe符号，因此/cat|dog/匹配的是包含"cat"或者"dog"的字符串，而不是包含"catog"或者"cadog"的字符串。使用括号（parenthesis）可以改变优先级，比如：/gupp(y|ies)/可以匹配"guppy"或者"guppies"。对于计数器，其优先级也低于简单序列，因此有时候也需要使用括号来改变优先级，比如：/(Column [0-9]+ \*)\*/。

操作符优先层级（operator precedence hierarchy）:

| name                  | symbol   |
| :-------------------- | :------- |
| Parenthesis           | ()       |
| Counters              | * + ? {} |
| Sequences and anchors | the ^ $  |
| Disjunction           | \|       |

贪婪（greedy）和不贪婪（non-greedy），正则表达式匹配有多种可能性，特别是有Kleene计数器的情况下，匹配最长的字符串称为贪婪模式，不贪婪模式则匹配最短的字符串。在Kleene star和Kleene plus后面加?，可以表示不贪婪匹配：*?或者+?。

#### Precision and recall

编写正则表达式主要解决不改被匹配的字符串（False Positive）和漏掉没匹配的字符串（False Negative）两类问题，提高precision和recall。
$$
precision = \frac{True Positive}{True Positive + True Negative} \\
recall  = \frac{True Positive}{True Positive + False Negative}
$$

#### 更多操作符

常用范围（range）别名：

| RE   | Expansion   | Match        | First Matches     |
| ---- | ----------- | ------------ | ----------------- |
| \d   | [0-9]       | 任意数字     | Party of <u>5</u> |
| \D   | [^0-9]      | 任意非数字   | <u>B</u>lue moon  |
| \w   | [a-zA-Z0-9] | 字母或数字   | <u>D</u>aiyu      |
| \W   | [^\w]       | 非字母或数字 | Aha<u>!</u>       |
| \s   | [ \r\t\n\f] | 白空格       |                   |
| \S   | [^\s]       | 非白空格     |                   |

计数器（counter）：

| RE     | Match     |
| ------ | --------- |
| *      | 0或者多个 |
| +      | 1或者多个 |
| ?      | 0或者1个  |
| {n}    | n个       |
| {n, m} | n到m个    |
| {n, }  | 至少n个   |

特殊字符:

| RE   | Match     | First Matches                               |
| ---- | --------- | ------------------------------------------- |
| \\*  | *字符     | K<u>*</u>A                                  |
| \\.  | .字符     | Dr<u>.</u> Livingston                       |
| \?   | 问号      | Why don't they com  and lend a hand<u>?</u> |
| \n   | 新行符号  |                                             |
| \t   | tab制表符 |                                             |

#### 正则表达式替换和捕获组

使用vim或者sed等工具替换，例如：替换字符串s/colour/color/ ，给所有数字增加尖括号s/([0-9]+)/<\1>/

捕获组（capture group），使用括号（parenthesis）定义和保存模式（pattern）称为捕获组。被捕获的结果保存在寄存器（register）内，使用 \n 来引用，其中n是第n个捕获组。

/The (.\*)er they (.\*), the \1er we \2/ 匹配 "The faster they ran, the faster we ran."

非捕获组（non-capturing group），如果只想改变优先级，不想捕获保存进寄存器，可以使用非捕获组：(?: pattern)。

#### Lookahead assertions

某些情况下，需要往前看有没有那些模式存在，但不移动匹配光标，这时可以使用lookahead assertions。

(?= pattern)返回true如果pattern存在，但是宽度为0，(?! pattern)返回true如果pattern不存在，但是宽度也为0。



