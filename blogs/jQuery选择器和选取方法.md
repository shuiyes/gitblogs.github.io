# jQuery选择器和选取方法

我们已经使用了带有简单Css选择器的jQuery选取函数:$()。现在是时候深入了解jQuery选择器语法，以及一些提取和扩充选中元素集的方法了。

## 一、jQuery选择器

在CSS3选择器标淮草案定义的选择器语法中，jQuery支持相当完整的一套子集，同时还添加了一些非标准但很有用的伪类。注意:本节讲述的是 jQuery选择器。其中有不少选择器(但不是全部)可以在CSS样式表中使用。选择器语法有三层结构。你肯定已经见过选择器中最简单的形式。”#te st”选取id属性为”test”的元素。”blockquote”选取文档中的所有 blockquote 元素，而”div.note” 则选取所有class属性为”note”的 blockquote 元素。简单选择器可以组合成“组合选择器”，比如 “div.note>p”和“blockquote i”，只要用组合字符做分隔符就行。简单选择器和组合选择器还可以分组成逗号分隔的列表。这种选择器组是传递给$()函数最常见的形式。在解释组合选择器 和选择器组之前，我们必须先了解简单选择器的语法。

### 1、简单选择器

简单选择器的开头部分(显式或隐式地)是标签类型声明。例如，如果只对 P>元素感兴趣，简单选择器可以用“P”开头。如果选取的元素和标签名无关，则可以使用通配符“*”号来代替。如果选择器没有以标签名或通配符开头，则隐式含有一个通配符。

标签名或通配符指定了备选文档元素的一个初始集。在简单选择器中，标签类型声明之后的部分由零个或多个过滤器组成。过滤器从左到右应用，和书写顺序一致，其中每一个都会缩小选中元素集。下表列举了jQuery支持的过滤器。

jQuery选择过滤器

 过滤器 | 含义 
 --- | ---
 #id | 匹配id属性为id的元素。在有效的}ITML文档中，永远不会出现多个元素拥有相同的ID，因此该过滤器通常作为独立选择器来使用
 .class | 匹配class属性(是一串被解析成用空格分隔的单词列表)含有class单词的所有元素
 [attr] | 匹配拥有attr属性(和值无关)的所有元素
 [attr=val] | 匹配拥有attr属性且值为val的所有元素
 [attr!=val] | 匹配没有attr属性、或attr属性的值不为val的所有元素((jQuery的扩展)
 [attr^=val] | 匹配attr属性值以val开头的元素
 [attr$=val] | 匹配attr属性值以val结尾的元素
 [attr*=val] | 匹配attr属性值含有val的元素
 [attr~=val] | 当其attr属性解释为一个由空格分隔的单词列表时，匹配其中包含单词val的元素。因此选择器“div.note”与“div [class~=note]”相同
 [attr\|=val] | 匹配attr属性值以val开头且其后没有其他字符，或其他字符是以连字符开头的元素
 :animated | 匹配正在动画中的元素，该动画是由jQuery产生的
 :button | 匹配 button type=”button”>和 input type=”button”>元素(jQuery的扩展)
 :checkbox | 匹配 input type=”checkbox”>元素( jQuery的扩展)，当显式带有input标签前缀”input:checkbox”时，该过滤器更高效
 :checked | 匹配选中的input元素
 :contains(text) | 匹配含有指定text文本的元素(jQuery的扩展)。该过滤器中的圆括号确定了文本的范围—无须添加引号。被过滤的元素的文本是由textContent或innerText属性来决定的—这是原始文档文本，不带标签和注释
 :disabled | 匹配禁用的元素
 :empty | 匹配没有子节点、没有文本内容的元素
 :enabled | 匹配没有禁用的元素
 :eq(n) | 匹配基于文档顺序、序号从0开始的选中列表中的第n个元素(jQuery的扩展)
 :even | 匹配列表中偶数序号的元素。由于第一个元素的序号是0，因此实际上选中的是第1个、第3个、第5个等元素(jQuery的扩展)
 :file | 匹配 input type=”file”>元素(jQuery的扩展)
 :first | 匹配列表中的第一个元素。和“:eq(0)”相同(jQuery的扩展)
 :first-child | 匹配的元素是其父节点的第一个子元素。注意:这与“:first”不同
 :gt(n) | 匹配基于文档顺序、序号从0开始的选中列表中序号大于n的元素( jQuery的扩展)
 :has(sel) | 匹配的元素拥有匹配内嵌选择器sel的子孙元素
 :header | 匹配所有头元素: h1>, h2>, h3>, h4>, h5>或h6> (jQuery的扩展)
 :hidden | 匹配所有在屏幕上不可见的元素:大体上可以认为这些元素的offsetWidth和offsetHeight为0
 :image | 匹配 input type=”image”>元素。注意该过滤器不会匹配 img>元素( jQuery的扩展)
 :input | 匹配用户输入元素: input>,  textarea>,  select>和 button>( jQuery的扩展)
 :last | 匹配选中列表中的最后一个元素(( jQuery的扩展)
 :last-child | 匹配的元素是其父节点的最后一个子元素。注意:这与“:last”不同
 :lt(n) | 匹配基于文档顺序、序号从0开始的选中列表中序号小于n的元素( jQuery的扩展)
 :not(sel) | 匹配的元素不匹配内嵌选择器sel
 :nth(n) | 与“:eq(n)”相同(jQuery的扩展)
 :nth-child(n) | 匹配的元素是其父节点的第n个子元素。。可以是数值、单词even,单词odd或计算公式。 使用“:nth-child(even)”来选取那些在其父节点的子元素中排行第2或第4等序号的元素。使用“:nth-child(odd)”来选取那 些在其父节点的子元素中排行第1、第3等序号的元素。更常见的情况是，n是xn或x n+y这种计算公式，其中x和y是整数，n是字面量n。因此可以用nth-child(3n+1)来选取第1个、第4个、第7个等元素。注意该过滤器的序号是从1开始的，因此如果一个元素是其父节点的第一个子元素，会认为它是奇数元素，匹配的是3n+1，而不是3n。要和“:even以及“:odd”过滤器区分开来，后者匹配的序号是从0开始的。
 :odd | 匹配列表中奇数(从0开始)序号的元素。注意序号为1和3的元素分别是第2个和第4个匹配元素( jQuery的扩展)
 :only-child | 匹配那些是其父节点唯一子节点的元素
 :parent | 匹配是父节点的元素，这与“:empty”相反(jQuery的扩展)
 :password | 匹配 input type=”password”>元素(jQuery的扩展)
 :radio | 匹配 input type=”radio”>元素( j Query的扩展)
 :reset | 匹配 input type=”reset”>和 button type=”reset”>元素(jQuery的扩展)
 :selected | 匹配选中的 option>元素。使用“:checked”来选取选中的复选框和单选框(jQuery的扩展)
 :submit | 匹配 input type=”submit”>和 button type=”submit”>元素(jQuery的扩展)
 :text | 匹配 input type=”text”>元素(jQuery的扩展)
 :visible | 匹配所有当前可见的元素:大体上可以认为这些元素的offsetWidth和offsetHeight的值不为0，这和“:hidden”相反

注意:表中列举的部分选择器在圆括号中接受参数。例如，下面这个选择器选取的元素在其父节点的子元素中排行第1或第2等，只要它们含有“JavaScript”单词，就不包含元素。

p:nth-child(3n+1): text (JavaScript):not(:has(a))

通常来说，指定标签类型前缀，可以让过滤器的运行更高效。例如，不要简单使用”:radio”来选取单选框按钮，使用“input:radio”会 更好。ID过滤器是个例外，不添加标签前缀时它会更高效。例如，选择器“#address”通常比更明确的“form#address”更高效。

### 2、组合选择器

使用特殊操作符或“组合符”可以将简单选择器组合起来，表达文档树中元素之间的关系。下表列举了jQuery支持的组合选择器。这些组合选择器与CSS3支持的组合选择器是一样的。

下面是组合选择器的一些例子:

```
"blockquote i"              //匹配blockquote>里的  i>元素
"ol > li"                   //li>元素是ol>的直接子元素
"#output+*"                 //id="output"元素后面的兄弟元素
"div.note > h1+p"           //紧跟h1>的P>元素，在div class="note">里面
```

注意组合选择器并不限于组合两个选择器:组合三个甚至更多选择器也是允许的。组合选择器从左到右处理。

### 3、选择器组

传递给$()函数(或在样式表中使用)的选择器就是选择器组，这是一个逗号分隔的列表，由一个或多个简单选择器或组合选择器构成。选择器组匹配的元 素只要匹配该选择器组中的任何一个选择器就行。对我们来说，一个简单选择器也可以认为是一个选择器组。下面是选择器组的一些例子:

```
"h1, h2,h3"             //匹配 h1>,   h2>和  h3>元素
"#p1, #p2, #p3"         //匹配id为p1, p2或p3的元素
"div.note, p.note"      //匹配class="note"的  div>和  P>元素
"body>p,div.note>p"     //  body>和  div class="note">的  P>子元素
```

注意:CSS和jQuery选择器语法允许在简单选择器的某些过滤器中使用圆括号，但并不允许使用圆括号来进行更常见的分组。例如，不能把选择器组或组合选择器放在圆括号中并且当成简单选择器:

```
(h1, h2, h3)+p          //非法
h1+p, h2+p, h3+p        //正确的写法
```

## 二、选取方法

除了$()函数支持的选择器语法，jQuery还定义了一些选取方法。本章中我们已看到过的大部分jQuery方法都是在选中元素上执行某种操作。选取方法不一样:它们会修改选中元素集，对其进行提取、扩充或仅作为新选取操作的起点。

本节描述这些选取方法。你会注意到这些选取方法中的多数提供的功能与选择器语法的功能是一样的。

提取选中元素最简单的方式是按位置提取。first()返回的jQuery对象仅包含选中元素中的第一个，last()返回的jQuery对象则只 包含最后一个元素。更通用的是，eq()方法返回物Query对象只包含指定序号的单个选中元素。(在jQuery 1.4中，负序号也是允许的，会从选区的末尾开始计数。)注意这些方法返回的jQuery对象只含有一个元素。这与常见的数组序号是不一样的，数组序号返 回的单一元素没有经过jQuery包装:

```
var paras=$("p");
paras.first()           //仅选取第一个  p>元素
paras.last()            //仅选取最后一个  P>
paras.eq(1)             //选取第二个  P>
paras.eq(-2)           //选取倒数第二个  P>
paras[1]                //第二个  P>元素自身
```

通过位置提取选区更通用的方法是slice()o jQuery的slice()方法与Array.slice()方法类似:前者接受开始和结束序号(负序号会从结尾处计算)，返回的jQuery对象包含 从开始到结束序号(但不包含结束序号)处的元素集。如果省略结束序号，返回的对象会包含从开始序号起的所有元素:

```
$("p").slice(2,5)       //选取第3个、第4个和第5个  P>元素
$("div").slice(-3)      //选取最后3个  div>元素
```

filter()是通用的选区过滤方法，有3种调用方式:

* 传递选择器字符串给filter()，它会返回一}jQuery对象，仅包含也匹配该选择器的选中元素。
* 传递另一个jQuery对象给filter()，它会返回一个新的jQuery对象，该对象包含这两们Query对象的交集。也可以传递元素数组甚至单一文档元素给filter()。
* 传递判断函数给filter()，会为每一个匹配元素调用该函数，filter()则返回一个jQuery对象，仅包含判断函数为true(或任意真值)的元素。在调用判断函数时，this值为当前元素，参数是元素序号。

```
$("div").filter(".note")        //与$("div.note")一样
$("div").filter($(".note"))     //与$("div.note")一样
$("div").filter(function(idx){return idx%2 == 0})         //与$("div:even")一样
```

not()方法与filter()一样，除了含义与filter()相反。如果传递选择器字符串给not()它会返回一个新的jQuery对象，该 对象只包含不匹配该选择器的元素。如果传递jQuery对象、元素数组或单一元素给not()，它会返回除了显式排除的元素之外的所有选中元素。如果传递 判断函数给not()，该判断函数的调用就与在filter()中一样，只是返回的jQuery对象仅包含那些使得判断函数返回false或其他假值的元 素:

```
$("div").not("#header, #footer");        //除了两个特殊元素之外的所有元素
```

　　在jQuery 1.4中，提取选区的另一种方式是has()方法。如果传入选择器，has()会返回一个新的jQuery对象，仅包含有子孙元素匹配该选择器的选中元素。如果传入文档元素给has()，它会将选中元素集调整为那些是指定元素祖先节点的选中元素:

```
$("p").has("a[href]")         //包含链接的段落
```

add()方法会扩充选区，而不是对其进行过滤或提取。可以将传给$()函数的任何参数(除了函数)照样传给add()方法。add()方法会返回 原来的选中元素，加上传给$()函数的那些参数所选中(或创建)的那些元素。add()会移除重复元素，并对该组合选区进行排序，以便里面的元素按照文档 中的顺序排列:

```
//选取所有  div>和所有  P>元素的等价方式
$("div, p")             //使用选择器组
$("div").add(p)         //给add()传入选择器
$("div").add($("p"))    //给add()传入jQuery对象
var paras = document.getElementsByTagName("p");       //类数组对象
$("div").add(paras);        //给add()传入元素数组
```

### 1.将选中元素集用做上下文

上面描述的filter(). add()、和not()方法会在各自的选中元素集上执行交集、并集和差集运算。jQuery还定义一些其他选取方法可将当前选中元素集作为上下文来使 用。对选中的每一个元素，这些方法会使用该选中元素作为上下文或起始点来得到新的选中元素集，然后返回一个新的jQuery对象，包含所有新的选中元素的 并集。与add()方法类似，会移除重复元素并进行排序，以便元素会按照在文档中出现的顺序排列好。

该类别选取方法中最通用的是find()。它会在每一个当前选中元素的子孙元素中寻找与指定选择器字符串匹配的元素，然后它返回一个新的 jQuery对象来代表所匹配的子孙元素集。注意这些新选中的元素不会并入已存在的选中元素集中。同时注意find()和filter()不 同，filter()不会选中新元素，只是简单地将当前选中的元素集进行缩减:

```
$("div").find("p")            //在中查找元素，与$("div p")相同
```

该类别中的其他方法返回新的jQuery对象，代表当前选中元素集中每一个元素的子元素、兄弟元素或父元素。大部分都接受可选的选择器字符串作为参数。不传入选择器时，它们会返回所有子元素、兄弟元素或父元素。传入选择器时，它们会过滤元素集，仅返回匹配的。

children()方法返回每一个选中元素的直接子元素，可以用可选的选择器参数进行过滤:

```
//寻找id为"header"和"footer"元素的子节点元素中的所有  span>元素
//与$("#header>span, #footer>span")相同
$("#header, #footer").children("span")
```

contents()方法与children()方法类似，不同的是它会返回每一个元素的所有子节点，包括文本节点。如果选中元素集中 有  iframe>元素，contents()还会返回该  iframe>内容的文档对象。注意contents()不接受可选 的选择器字符串参数—因为它返回的文档节点不完全是元素，而选择器字符串仅用来描述元素节点。
next()和prev()方法返回每一个选中元素的下一个和上一个兄弟元素(如果有的话)。如果传入了选择器，会只选中匹配该选择器的兄弟元素:

1
2
$("h1").next("p")      //与$("h1+p")相同
$("h1").prev()         //  h1>元素前面的兄弟元素
nextAll()和prevAll()返回每一个选中元素前面或后面的所有兄弟元素(如果有的话)。siblings()方法则返回每一个选中元素的所有兄弟元素(选中元素本身不是自己的兄弟元素)。如果给这些方法传人选择器，则只会返回匹配的兄弟元素:

```
$("#footer").nextAll("p")       //紧跟#footer元素的所有  P>兄弟元素
$("#footer").prevAll()          //#footer元素前面的所有兄弟元素
```

从jQuery 1.4开始，nextUntil()和prevUntil()方法接受一个选择器参数，会选取选中元素后面或前面的所有兄弟元素，直到找到某个匹配该选择 器的兄弟元素为止。如果省略该选择器，这两个方法的作用就和不带选择器的nextAll()和prevAll()一样。

parent()方法返回每一个选中元素的父节点:

```
$("li").parent()        //列表元素的父节点，比如  u1>和  ol>元素
```

parents()方法返回每一个选中元素的祖先节点(向上直到元素)。parent()和parents()都接受一个可选的选择器字符串参数:

```
$("a[href]").parents("p")            //含有链接的  P>元素
```

parentsUntil()返回每一个选中元素的祖先元素，直到出现匹配指定选择器的第一个祖先元素。closest()方法必须传人一个选择器 字符串，会返回每一个选中元素的祖先元素中匹配该选择器的最近一个祖先元素(如果有的话)。对该方法而言，元素被认为是自身的祖先元素。在jQuery 1.4中，还可以给closest()传入一个祖先元素作为第二个参数，用来阻止jQuery往上查找时超越该指定元素:

```
$("a[href]").closest("div")         //包含链接的最里层  div>
$("a[href]").parentsUntil(":not(div)")      //所有包裹  a>的  div>元素
```

### 2、恢复到之前的选中元素集

为了实现方法的链式调用，很多jQuery对象的方法最后都会返回调用对象。然而本节讲述的方法都返回新的jQuery对象。可以链式调用下去，但必须清晰地意识到，在链式调用的后面所操作的元素集，可能已经不是该链式调用开始时的元素集了。

实际情况还要复杂些。当这里所描述的选取方法在创建或返回一个新的ejQuery对象时，它们会给该对象添加一个到它派生自的旧jQuery对象的 内部引用。这会创建一个jQuery对象的链式表或栈。end()方法用来弹出栈，返回保存的jQuery对象。在链式调用中调用end()会将匹配元素 集还原到之前的状态。考虑以下代码:

```
//寻找所有  div>元素，然后在其中寻找  P>元素
//高亮显示  P>元素，然后给  div>元素添加一个边框
//首先，不使用链式调用
var divs = $("div");
var paras = div.find("p");
paras.addClass("highlight");
divs.css("border", "solid black 1px");
 
//下面展现如何使用链式调用来实现
$("div").find("p").addClass("highlight").end().css("border", "solid black 1px");
//还可以将操作调换顺序来避免调用end()
$("div").css("border", "solid block 1px").find("p").addClass("highlight");
```

如果想手动定义选中元素集，同时保持与end()方法的兼容，可以将新的元素集作为数组或类数组对象传递给push5tack()方法。指定的元素会成为新的选中元素，之前选中的元素集则会压入栈中，之后可以用end()方法还原它们:

```
var sel = $("div");             //选取所有  div>元素
sel.pushStack(document.getElementsByTagName("p"));      //修改为所有  P>元素
sel.end();                      //还原为  div>元素
```

既然我们已经讲解了end()方法及其使用的选区栈，就有最后一个方法需要讲解。andSelf()返回一个新的jQuery对象，包含当前的所有 选中元素，加上之前的所有选中元素(会去除重复的)。andSelf()和add()方法一样，或许“addPrev”是一个更具描述性的名字。作为例 子，考虑上面代码的下述变化:高亮显示  P>元素及其父节点中的  div>元素，然后给这些  div>元素添加边 框:

```
$("div").find("p").andSelf().           //寻找  div》中的  P>，合并起来
addClass("highlight").              //都高亮
end().end().                            //弹出栈两次，返回$("div")
css("border", "solid black 1px");       //给divs添加边框
```

