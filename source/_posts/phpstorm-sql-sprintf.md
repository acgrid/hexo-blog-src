title: PHPStorm SQL字符串与sprintf格式化
date: 2016-01-21 10:51:41
tags:
- SQL
- PHPStorm
- sprintf
- 自定义
- 效率
---
# 问题
PHPStorm打开一个老项目，在SQL字符串上全是警告，要求选择正确的SQL dialect，然而选择以后语法要求更加严格了，警告都成了错误。比如这张图：

{% asset_img problem.png 血淋淋的SQL %}

PHPStorm版本为 10.0.3。

# 已有方案
[这里](http://stackoverflow.com/questions/25529608/avoid-syntax-error-warnings-when-using-string-interpolation-in-sql-query-generat)的解决办法。

如果确实已有的SQL太坑了，连人都看不下去又不想重构，关闭检查也算是一种办法。

然而试过PHPStorm添加数据库以后，自动补全功能又让人欲罢不能。

而且看起来PHPStorm也提供了User Parameter的支持，看起来就是他了。

# 波折

当我兴冲冲的把`\%\w+`加入列表以后……并没有什么卵用，把它放在首位，也没有变化。

{% asset_img setting.png 勾选的设定 %}

怎么回事？虽然不太相信，将自带的规则删除后，错误消失了，明明下面带百分号的那条还限定了Java，没想到真的有冲突。

为了支持类似于`%.2F`这样的格式化说明符，我把正则改为`\%(\.)?\w+`。至于为什么不是`%.2f`，大写F是非区域化，永远使用点作为小数点，不然当区域设定的小数点是逗号的时候就杯具了。

最后发现UPDATE后面的%s仍然报错，既然不符合语法，就把%s换成普通的字符串连接，这下终于不报错了。

{% asset_img resolved.png 成功洗白 %}

# 总结

找到解决方法后经过修改，1500多个SQL总算不报错了，有的因为连表名都是动态的所以出现无法解析的警告，至少比以前看起来好多了。

使用`sprintf`来格式化SQL现在来看也是权宜之计，新的项目应该都是用数据对象抽象层或者用准备语句。讲道理，Raw SQL带来的迷之13格的代价就是可理解性和可测试性的损失啊。