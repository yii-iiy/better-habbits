# 代码格式规范

*文档在 GFDL Latest 下发放*

## 注释和文档

### 中英文混用场景

中文和英文/数字之间务必有半角空格符，除非有标点符号间隔。

- `你好，Jimmy 小朋友` 规范
- `你好，Jimmy小朋友` 不规范

英文标点要在标点后有一个空格，除非位于段落末尾。

- `吃饭啦！Tom, Jerry and Jimmy！` 规范
- `吃饭啦！Tom,Jerry and Jimmy！` 不规范

## 代码格式

对机器而言（功能和性能上）没什么区别的代码，在人类表达的层面上可以有多种表达方式。

在这里，只要机器执行起来区别不大，这样的不同代码之间互相称为具有“执行等价性”。

在此情况下，在编写代码时认真选择一个最合适的表达，就成为了一件需要去做的事情。这是此章要讨论的。

### 管道

是否应使用管道，应取决于函数的设计。

- 如果函数被设计为头一个参数相比其余参数有特殊地位的、或一个调用中头参具特殊地位的，调用时应当使用管道。
- 如果函数被设计为所有参数地位等价的、或一个调用中要强调其每个参数地位等价（而不是应强调某参数特殊地位）的，调用时应不使用管道。
- 其余情况，可为代码整洁性和便于理解来服务。比如，如果减少逗号的使用很容易且能用于避免一些可能的造成错误的几率，管道要优先被考虑；如果管道滥用造成代码同样失去重点，则不应使用。

#### 一般

比如，调用中参数是并列关系的地位：

~~~ r
paste0 (
	"aaaa", 
	"_", "bbbb", 
	"-", "cccc")
~~~

或者将参数看作并列比不看作并列更有利于读者理解：

~~~ r
somefunc (
	setting1, 
	setting2, 
	setting3)
~~~

此时不应用管道如 `setting1 %>% somefunc (setting2, setting3)` ，这会降低可读性。

但如果是这样：

~~~ r
somefunc (
	setting_list_object, 
	setting1, 
	setting2, 
	setting3)
~~~

后续的 `setting1` `setting2` 等待都是配置如何将函数应用于 `setting_list_object` 这个主角一般的对象身上，那么下面这样写就会可读性更好：

~~~ r
setting_list_object %>% somefunc (
	setting1, 
	setting2, 
	setting3)
~~~

如果这个函数的调用完整是这样的：

~~~ r
obj <- somefunc (
	obj, 
	setting1, 
	setting2, 
	setting3)
~~~

使用管道可以直观地写作这样：

~~~ r
obj %<>% somefunc (
	setting1, 
	setting2, 
	setting3)
~~~

它能直观地表示“以参数 `setting1` `setting2` `setting3` 为配置将函数 `somefunc` 应用于对象 `obj` 上并将应用后的结果覆盖原值”这样的意思。

再比如：

~~~ r
paste0 (
	"aaaa", 
	"_", "bbbb", 
	"-", "cccc")
~~~

如果 `aaaa` 所表达的事物**在你的业务逻辑中**有较后面的 `bbbb` `cccc` 而言更有特殊地位，比如 `aaaa` 是名称而后面的只是后缀，那么此时，更好可读性的表达就应该是把 `aaaa` 拎出来写：

~~~ r
"aaaa" %>% paste0 (
	"_", "bbbb", 
	"-", "cccc")
~~~

以强调**合乎业务逻辑结构**的含义：“以 `bbbb` 和 `cccc` 为后缀，将拼接操作应用于我们业务逻辑中的此小段代码的主体 `aaaa` 上面”。

甚至，如果你的业务逻辑是给名称 `bbbb` 加前后缀，有可读性的写法就得是这样写：

~~~ r
"bbbb" %>% paste0 (
	"aaaa", "_", 
	., 
	"-", "cccc")
~~~

正所谓，*“最好的可读性就是一眼就从代码看见业务逻辑本身而不需要注释描述太多”*。

但还是那句话，如果 `aaaa` `bbbb` `cccc` 关系并列，比如它们是一个表的三个字段现在需要拼接起来，那么，使用管道就是**降低可读性**的行为了，就还是要这样才对：

~~~ r
paste0 (
	"aaaa", 
	"_", "bbbb", 
	"-", "cccc")
~~~

或者更应该这样写，如果 `_` 和 `-` 有不同层次含义的话：

~~~ r
paste0 (
	"aaaa", "_", "bbbb", 
	"-", "cccc")
~~~

#### 调用链

一个数据对象经过一系列转换逻辑的场景（**这种场景在绝大多数特别是（任何格式的）数据的处理的编程中都会有而不仅仅用 DPLYR 时会有**），管道好过中间值。

这是管道调用链的表达效果：

~~~ r
any_data_object %>% 
	transer1 (option1 = 1, option2 = T) %>% 
	transer2 (option1 = T) %>% 
	transer3 () %>% 
	transer4 (para1, para2, para3, .) %>% 
	{.} ;
~~~

只是经过 4 个转换，看起来并不算多。

下面是不用管道的两种情况。

嵌套调用：

~~~ r
transer4 (
	para1, 
	para2, 
	para3, 
	transer3 (
		transer2 (
			transer1 (
				any_data_object, 
				option1 = 1, 
				option2 = T), 
			option1 = T)))
~~~

管道本身就是为了为嵌套调用提供更好的展现。这个代码的可读性就不用说了。哪怕不提它奇怪的形状，括号也会让人眼晕（即便有括号着色（有着色反而更眼晕分不清层次）），而且要理解它的正确阅读顺序是要从后往前倒着来的。

嵌套调用显然不会是个好点子。那赋中间值不就能解决这个问题了吗？

赋中间值调用：

~~~ r
middle1 <- transer1 (any_data_object, option1 = 1, option2 = T)
middle2 <- transer2 (middle1, option1 = T)
res <- transer4 (para1, para2, para3, transer3 (middle2))
~~~

如此，嵌套调用只是最低限度地用了一下，看起来至少形状没那么奇怪了。

那么，要理解这个代码，需要如何梳理呢？

首先，这段代码要处理什么，并不那么明显。 `any_data_object` 和 `option1` 处于同层级，这次调用的意图不那么能看出来。为了确定意图，你不得不进入 `transer1` 的定义。而如果你在 `transer1` 中的编码也是这个风格，你可能就不得不再次进入其中某个调用的定义。现在，你体验到了试图捋清楚到处都是设计模式的代码的人的痛苦：你跳来跳去，又似乎跳到原来的地方，仿佛进入了一个迷宫，然后迷路了。

如果是你自己构建的这个迷宫那你更惨，因为不会再有人能给你更多的提示了。

等待，提示？用注释似乎是个好办法。然后代码变成了这样：

~~~ r
...

# ------------------------

# middle1 只是个中间变量，不用担心，下面它只会被用一次。
# transer1 是用来转换 any_data_object 的，另外两个参数只是一些选项，你可以根据需要来改变。
middle1 <- transer1 (any_data_object, option1 = 1, option2 = T)
# 只有这里用到了 middle1 ，并且 transer2 的目的也只是转换 middle1 为 middle2 而已，除此外再没有别的含义了。
middle2 <- transer2 (middle1, option1 = T)
# 这里有 transer3 和 transer4 两个调用，别看错了，是 transer3 先发生 transer4 后发生。
# transer3 的目的只是用来应用于 middle2 然后再传给 transer4 的。
# 注意，是 transer3 应用于 middle2 然后再传给 transer4 ，别看错了。
# 另外， middle2 也没再被使用，不用在下面找了。
res <- transer4 (para1, para2, para3, transer3 (middle2))
# 哦对了，这三段代码其实是一个整体。写成这样可能看不出来，但它们确实是合在一起做一件事的，这里记一下。
# （再上下加个划线用注释强调它们是一个整体）

# ------------------------

...
~~~

但麻烦仍然存在。注释能使用的自然语言只有一种，如果注释很多，用两种自然语言都会显得鸠占鹊巢。

而且，不知你有没有发现，像上面这样阅读注释能捋清楚的，其实就是最开始管道代码已经能够表达了的。管道代码不需要这么多注释，代码本身就能体现相应的含义。

换句话说，不论赋中间值还是嵌套版本，人理解这些代码的过程，也是要在脑内绘制出一个数据变化的流程图出来。然而，这个流程图和使用管道的例子是同构的。

再换句话说，好的代码表达，就是能够与思路本身同构、且再没有多余信息点了的代码表达。

在这里， `middle` 系列变量就是多余表达、需要从后往前看、从里往外看的嵌套（即便只有一层）调用就是不同构。要知道，这是示例，所以你能从我写的 `1` `2` `3` 里仍能看出顺序，但实际情况不是这样的，一个函数能被抽象出来一定是因为它可以用在这里**并且**它还可以用在那里，你没法在定义的时候预言所有可能的调用情况。

另外，一个极坏的点子就是，看到自己的赋中间值或者嵌套调用（或者都有）的代码看起来过于繁琐，就一股脑给它丢到一个函数定义的 `{}` 中间，以为藏起来就可以万事大吉了，但这并不会消灭糟糕的表达，正如上面在“你不得不进入”那里提到过的。




### 缩进

缩进可以用制表符或空格，但同一文件（或项目）内应当统一。

~~~ r
add1 = 
function (x) 
	x + 1 ;
~~~

对齐要用空格，不要用缩进。

~~~ r
add1 =        # xxxx
function (x)  # xxxx
	x + 1 ;   # xxxx
~~~

原本的一行代码如果被换行符截断，原有的空格不应删除。但一行命令的最后不要有空格。

~~~ r
add1 = 
function (x) 
	x + 1 ;

id = \ (a) a ;
~~~

考虑到任何人都可能要用较小显示器工作，因而尽量不要像这样缩进：

~~~ r
some_long_long_named_function (some_long_long_named_call_or_val1, 
                               some_long_long_named_call_or_val2, 
                               some_long_long_named_call_or_val3)
~~~

甚至这样（用某些编辑器如 RSTUDIO 的编辑器如果懒于手动修复格式化就可能出现这样的缩进）：

~~~ r
some_long_long_named_function1 (some_long_long_named_call_or_val1, 
                                some_long_long_named_function2 (some_long_long_named_call_or_val1, some_long_long_named_call_or_val2 + 
                                                                                                   some_long_long_named_call_or_val3), some_long_long_named_call_or_val3)
~~~

对于上述情况，建议这样：

~~~ r
some_long_long_named_function1 (some_long_long_named_call_or_val1, 
	some_long_long_named_function2 (some_long_long_named_call_or_val1, 
		some_long_long_named_call_or_val2 + 
			some_long_long_named_call_or_val3), 
	some_long_long_named_call_or_val3)
~~~

或者这样：

~~~ r
some_long_long_named_function1 (
	some_long_long_named_call_or_val1, 
	some_long_long_named_function2 (
		some_long_long_named_call_or_val1, 
		some_long_long_named_call_or_val2 + 
			some_long_long_named_call_or_val3), 
	some_long_long_named_call_or_val3)
~~~

为了避免不同层次的东西被缩进在一个层级从而导致容易混淆，应当灵活处理，尽可能优雅地更换其它合适的格式化方案以使它们错开：

~~~ r
some_long_long_named_function = 
function (
	
	some_long_long_name1, 
	some_long_long_name2, 
	some_long_long_name3) 
	
	some_long_long_name1 (
		some_long_long_name2) + 
		some_long_long_name3 ;

some_long_long_name1 %>% f (
	some_long_long_name2, 
	some_long_long_name3) %>% 
	f2
~~~

上述就不是好的缩进，因为不同层次的在同一层次。下面是对上述两种情况的变动：

~~~ r
some_long_long_named_function = 
function (
		
		some_long_long_name1, 
		some_long_long_name2, 
		some_long_long_name3
		
	) 
	
	some_long_long_name2 %>% 
		{some_long_long_name1 (.) + 
			some_long_long_name3} %>% 
	{.} ;

some_long_long_name1 %>% 
	f (
		some_long_long_name2, 
		some_long_long_name3) %>% 
	f2
~~~


### 关于语言的便利性

语言或库有时会提供一些便利出来，这会让表达的方式可以变多。

比如这个：

~~~ r
(\ (a) (\ (b) (\ (c) a + b + c) (c = a * b)) (b = a + 1)) (a = 3)
~~~

或者写成这样可能可读性稍微好些：

~~~ r
(\ (a) 
(\ (b) 
(\ (c) 
	
	a + b + c) 

(c = a * b)) 
(b = a + 1)) 
(a = 3)
~~~

—— 因为此处的**目的**就是做三个赋值操作**而已了**，所以这样去缩进最能体现意图（业务逻辑）。

而语言提供的代码块功能 `{}` 则允许将多段表达式变为一段，从而上面的代码可以像这样写：

~~~ r
{
	a = 3
	b = a + 1
	c = a * b

	a + b + c
}
~~~

它和 `(\ (a) (\ (b) (\ (c) a + b + c) (c = a * b)) (b = a + 1)) (a = 3)` 是等效的。

因此，类似情况，完全可以用大括号化简写法。

但由于该功能（允许将多段表达式变为一段）可以做到的事情过于宽泛，因而使得它能被用来做一些不好的事情。

就比如前面提到的管道链：

~~~ r
any_data_object %>% 
	transer1 (option1 = 1, option2 = T) %>% 
	transer2 (option1 = T) %>% 
	transer3 () %>% 
	transer4 (para1, para2, para3, .) %>% 
	{.} ;
~~~

被写成这样：

~~~ r
# middle1 只是个中间变量，不用担心，下面它只会被用一次。
# transer1 是用来转换 any_data_object 的，另外两个参数只是一些选项，你可以根据需要来改变。
middle1 <- transer1 (any_data_object, option1 = 1, option2 = T)
# 只有这里用到了 middle1 ，并且 transer2 的目的也只是转换 middle1 为 middle2 而已，除此外再没有别的含义了。
middle2 <- transer2 (middle1, option1 = T)
# 这里有 transer3 和 transer4 两个调用，别看错了，是 transer3 先发生 transer4 后发生。
# transer3 的目的只是用来应用于 middle2 然后再传给 transer4 的。
# 注意，是 transer3 应用于 middle2 然后再传给 transer4 ，别看错了。
# 另外， middle2 也没再被使用，不用在下面找了。
res <- transer4 (para1, para2, para3, transer3 (middle2))
~~~

然后再，不是用注释而是用大括号来把它们变为整体：

~~~ r
{
	# middle1 只是个中间变量，不用担心，下面它只会被用一次。
	# transer1 是用来转换 any_data_object 的，另外两个参数只是一些选项，你可以根据需要来改变。
	middle1 <- transer1 (any_data_object, option1 = 1, option2 = T)
	# 只有这里用到了 middle1 ，并且 transer2 的目的也只是转换 middle1 为 middle2 而已，除此外再没有别的含义了。
	middle2 <- transer2 (middle1, option1 = T)
	# 这里有 transer3 和 transer4 两个调用，别看错了，是 transer3 先发生 transer4 后发生。
	# transer3 的目的只是用来应用于 middle2 然后再传给 transer4 的。
	# 注意，是 transer3 应用于 middle2 然后再传给 transer4 ，别看错了。
	# 另外， middle2 也没再被使用，不用在下面找了。
	res <- transer4 (para1, para2, para3, transer3 (middle2))

	res
}
~~~

这当然会比用注释划分界限好一些，但其实还是差不多一样乱的。还是要有协助理解代码的注释 —— 或者，没有这些注释，但这会让读者遇到前文提到过的困境。


# License

GFDL and AGPL Latest version
