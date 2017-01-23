# jQuery选择器和选取方法

我们已经使用了带有简单Css选择器的jQuery选取函数:$()。现在是时候深入了解jQuery选择器语法，以及一些提取和扩充选中元素集的方法了。

## 一、jQuery选择器

在CSS3选择器标淮草案定义的选择器语法中，jQuery支持相当完整的一套子集，同时还添加了一些非标准但很有用的伪类。注意:本节讲述的是 jQuery选择器。其中有不少选择器(但不是全部)可以在CSS样式表中使用。选择器语法有三层结构。你肯定已经见过选择器中最简单的形式。”#te st”选取id属性为”test”的元素。”blockquote”选取文档中的所有<blockquote>元素，而”div.note” 则选取所有class属性为”note”的<blockquote>元素。简单选择器可以组合成“组合选择器”，比如 “div.note>p”和“blockquote i”，只要用组合字符做分隔符就行。简单选择器和组合选择器还可以分组成逗号分隔的列表。这种选择器组是传递给$()函数最常见的形式。在解释组合选择器 和选择器组之前，我们必须先了解简单选择器的语法。

### 1、简单选择器

简单选择器的开头部分(显式或隐式地)是标签类型声明。例如，如果只对<P>元素感兴趣，简单选择器可以用“P”开头。如果选取的元素和标签名无关，则可以使用通配符“*”号来代替。如果选择器没有以标签名或通配符开头，则隐式含有一个通配符。

标签名或通配符指定了备选文档元素的一个初始集。在简单选择器中，标签类型声明之后的部分由零个或多个过滤器组成。过滤器从左到右应用，和书写顺序一致，其中每一个都会缩小选中元素集。下表列举了jQuery支持的过滤器。

jQuery选择过滤器
过滤器 | 含义
 #id | 匹配id属性为id的元素。在有效的}ITML文档中，永远不会出现多个元素拥有相同的ID，因此该过滤器通常作为独立选择器来使用
.class | 匹配class属性(是一串被解析成用空格分隔的单词列表)含有class单词的所有元素
[attr] | 匹配拥有attr属性(和值无关)的所有元素
[attr=val] | 匹配拥有attr属性且值为val的所有元素
[attr!=val] | 匹配没有attr属性、或attr属性的值不为val的所有元素((jQuery的扩展)
[attr^=val] | 匹配attr属性值以val开头的元素
[attr$=val] | 匹配attr属性值以val结尾的元素
[attr*=val] | 匹配attr属性值含有val的元素
[attr~=val] | 当其attr属性解释为一个由空格分隔的单词列表时，匹配其中包含单词val的元素。因此选择器“div.note”与“div [class~=note]”相同
[attr|=val] | 匹配attr属性值以val开头且其后没有其他字符，或其他字符是以连字符开头的元素
:animated | 匹配正在动画中的元素，该动画是由jQuery产生的
:button | 匹配<button type=”button”>和<input type=”button”>元素(jQuery的扩展)
:checkbox | 匹配<input type=”checkbox”>元素( jQuery的扩展)，当显式带有input标签前缀”input:checkbox”时，该过滤器更高效
:checked | 匹配选中的input元素
:contains(text) | 匹配含有指定text文本的元素(jQuery的扩展)。该过滤器中的圆括号确定了文本的范围—无须添加引号。被过滤的元素的文本是由textContent或innerText属性来决定的—这是原始文档文本，不带标签和注释
:disabled | 匹配禁用的元素
:empty | 匹配没有子节点、没有文本内容的元素
:enabled | 匹配没有禁用的元素
:eq(n) | 匹配基于文档顺序、序号从0开始的选中列表中的第n个元素(jQuery的扩展)
:even | 匹配列表中偶数序号的元素。由于第一个元素的序号是0，因此实际上选中的是第1个、第3个、第5个等元素(jQuery的扩展)
:file | 匹配<input type=”file”>元素(jQuery的扩展)
:first | 匹配列表中的第一个元素。和“:eq(0)”相同(jQuery的扩展)
:first-child | 匹配的元素是其父节点的第一个子元素。注意:这与“:first”不同
:gt(n) | 匹配基于文档顺序、序号从0开始的选中列表中序号大于n的元素( jQuery的扩展)
:has(sel) | 匹配的元素拥有匹配内嵌选择器sel的子孙元素
:header | 匹配所有头元素:<h1>, <h2>, <h3>, <h4>, <h5>或<h6> (jQuery的扩展)
:hidden | 匹配所有在屏幕上不可见的元素:大体上可以认为这些元素的offsetWidth和offsetHeight为0
:image | 匹配<input type=”image”>元素。注意该过滤器不会匹配<img>元素( jQuery的扩展)
:input | 匹配用户输入元素:<input>, <textarea>, <select>和<button>( jQuery的扩展)
:last | 匹配选中列表中的最后一个元素(( jQuery的扩展)
:last-child | 匹配的元素是其父节点的最后一个子元素。注意:这与“:last”不同
:lt(n) | 匹配基于文档顺序、序号从0开始的选中列表中序号小于n的元素( jQuery的扩展)
:not(sel) | 匹配的元素不匹配内嵌选择器sel
:nth(n) | 与“:eq(n)”相同(jQuery的扩展)
:nth-child(n) | 匹配的元素是其父节点的第n个子元素。。可以是数值、单词even,单词odd或计算公式。 使用“:nth-child(even)”来选取那些在其父节点的子元素中排行第2或第4等序号的元素。使用“:nth-child(odd)”来选取那 些在其父节点的子元素中排行第1、第3等序号的元素。更常见的情况是，n是xn或x n+y这种计算公式，其中x和y是整数，n是字面量n。因此可以用nth-child(3n+1)来选取第1个、第4个、第7个等元素。注意该过滤器的序号是从1开始的，因此如果一个元素是其父节点的第一个子元素，会认为它是奇数元素，匹配的是3n+1，而不是3n。要和“:even以及“:odd”过滤器区分开来，后者匹配的序号是从0开始的。
:odd | 匹配列表中奇数(从0开始)序号的元素。注意序号为1和3的元素分别是第2个和第4个匹配元素( jQuery的扩展)
:only-child | 匹配那些是其父节点唯一子节点的元素
:parent | 匹配是父节点的元素，这与“:empty”相反(jQuery的扩展)
:password | 匹配<input type=”password”>元素(jQuery的扩展)
:radio | 匹配<input type=”radio”>元素( j Query的扩展)
:reset | 匹配<input type=”reset”>和<button type=”reset”>元素(jQuery的扩展)
:selected | 匹配选中的<option>元素。使用“:checked”来选取选中的复选框和单选框(jQuery的扩展)
:submit | 匹配<input type=”submit”>和<button type=”submit”>元素(jQuery的扩展)
:text | 匹配<input type=”text”>元素(jQuery的扩展)
:visible | 匹配所有当前可见的元素:大体上可以认为这些元素的offsetWidth和offsetHeight的值不为0，这和“:hidden”相反
