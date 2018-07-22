---
title: 认识与学习BASH之正规表示法与延伸正规表示法
date: 2017-12-29 15:55:29
categories:
- 书籍
- 《鸟哥的Linux私房菜基础篇（第三版）》
tags:
- 书籍
- BASH
- Shell
---
## 一、正规表示法
正规表示法 (Regular Expression, RE, 或称为常规表示法)是透过一些特殊字符的排列，用以【搜寻/取代/删除】一列或多列文字字符串，简单的说，正规表示法就是用在字符串的处理上面的一项【表示式】。正规表示法并不是一个工具程序，而是一个字符串处理的标准依据，如果您想要以正规表示法的方式处理字符串，就得要使用支持正规表示法的工具程序才行，这类的工具程序很多，例如 vi, sed, awk 等等。

### 1、什么是正规表示法
简单的说，正规表示法就是处理字符串的方法，他是以行为单位来进行字符串的处理行为，正规表示法透过一些特殊符号的辅助，可以让使用者轻易的达到【搜寻/删除/取代】某特定字符串的处理程序。
<!--more-->
举个系统常见的例子，假如你发现系统在开机的时候，出现一个关于 mail 程序的错误，而开机过程的相关程序都是在 /etc/init.d/ 底下，也就是说，在该目录底下的某个档案内具有 mail 这个关键词，你想要将该档案捉出来进行查询修改的动作。此时你怎么找出来含有这个关键词的档案？你当然可以一个档案一个档案的开启，然后去搜寻 mail 这个关键词，只是.....该目录底下的档案可能不止100 个。如果了解正规表示法的相关技巧，那么只要一行指令就找出来：`【grep 'mail' /etc/init.d/*】`那个 grep 就是支持正规表示法的工具程序之一。

谈到这里就得要进一步说明，正规表示法基本上是一种【表示法】，只要工具程序支持这种表示法，那么该工具程序就可以用来作为正规表示法的字符串处理之用。例如 vi, grep, awk ,sed 等等工具，因为她们有支持正规表示法，所以，这些工具就可以使用正规表示法的特殊字符来进行字符串的处理。但例如 cp, ls 等指令并未支持正规表示法，所以就只能使用 bash 自己本身的通配符而已。

### 2、正规表示法与Shell在Linux当中的角色定位：
我们谈到的这个正规表示法，就有点像是数学的九九表一样，是 Linux 基础当中的基础，虽然也是最难的部分，不过，如果学成了之后，一定是【大大的有帮助】的。这就好像是金庸小说里面的学武难关：任督二脉！ 打通任督二脉之后，武功立刻成倍成长！所以，不论是对于系统的认识与系统的管理部分，他都有很棒的辅助。

### 3、延伸的正规表示法：
正规表示法的字符串表示方式依照不同的严谨度而分为： 基础正规表示法与延伸正规表示法。延伸型正规表示法除了简单的一组字符串处理之外，还可以作群组的字符串处理，例如进行搜寻VBird 或 netman 或 lman 的搜寻，注意，是【或(or)】而不是【和(and)】的处理，此时就需要延伸正规表示法的帮助，藉由特殊的【 ( 】与【 | 】等字符的协助，就能够达到这样的目的。不过，我们在这里主要介绍最基础的基础正规表示法。
**（注：有一点要向大家报告说明清楚，那就是：【正规表示法与通配符是完全不一样的东西！】这很重要。因为【通配符 (wildcard) 代表的是 bash 操作接口的一个功能】，但正规表示法则是一种字符串处理的表示方式，这两者要分的很清楚才行。)**


### 4、通过grep实例演示来学习正规表示法：
新建regular_express.txt文档及测试内容：
>"Open Source" is a good mechanism to develop programs.
apple is my favorite food.
Football game is not use feet only.
this dress doesn't fit me.
However, this dress is about $ 3183 dollars.
GNU is free air not free beer.
Her hair is very beauty.
I can't finish the test.
Oh! The soup taste good.
motorcycle is cheap than car.
This window is clear.
the symbol '*' is represented as start.
Oh!	My god!
The gd software is a library for drafting programs.
You are the best is mean you are the no. 1.
The world <Happy> is the same with "glad".
I like dog.
google is the best tools for search keyword.
goooooogle yes!
go! go! Let's go.
\# I am VBird

共有22行，最底下一行为空白行！现在开始我们一个案例一个案例的来介绍。

#### 例题一：搜寻特定字符串
假设我们要从刚刚的档案当中取得 the 这个特定字符串，最简单的方式就是这样：
```
[root@www ~]# grep -n 'the' regular_express.txt
8:I can't finish the test.
12:the symbol '*' is represented as start.
15:You are the best is mean you are the no. 1.
16:The world <Happy> is the same with "glad".
18:google is the best tools for search keyword.
```
那如果想要【反向选择】呢？也就是说，当该行没有 'the' 这个字符串时才显示在屏幕上，那就直接使用：
`[root@www ~]# grep -vn 'the' regular_express.txt`

接下来，如果你想要取得不论大小写的 the 这个字符串，则：
`[root@www ~]# grep -in 'the' regular_express.txt`

#### 例题二、利用中括号 [] 来搜寻集合字符
如果我想要搜寻 test 或 taste 这两个单字时，可以发现到，其实她们有共通的 't?st' 存在。这个时候，我可以这样来搜寻：
```
[root@www ~]# grep -n 't[ae]st' regular_express.txt
8:I can't finish the test.
9:Oh! The soup taste good.
```
其实 [] 里面不论有几个字符，他都谨代表某【一个】字符，所以，上面的例子说明了，我需要的字符串是【tast】或【test】两个字符串而已。

而如果想要搜寻到有 oo 的字符时，则使用：
```
[root@www ~]# grep -n 'oo' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!
```

但是，如果我不想要 oo 前面有 g 的话呢？此时，可以利用在集合字符的反向选择 [^] 来达成：
```
[root@www ~]# grep -n '[^g]oo' regular_express.txt
2:apple is my favorite food.
3:Football game is not use feet only.
18:google is the best tools for search keyword.
19:goooooogle yes!
```

假设我 oo 前面不想要有小写字符，所以，我可以这样写 [^abcd....z]oo ，但是这样似乎不怎么方便，由于小写字符的 ASCII 上编码的顺序是连续的，因此，我们可以将之简化为底下这样：
```
[root@www ~]# grep -n '[^a-z]oo' regular_express.txt
3:Football game is not use feet only.
```
也就是说，当我们在一组集合字符中，如果该字符组是连续的，例如大写英文/小写英文/数字等等，就可以使用`[a-z],[A-Z],[0-9]`等方式来书写，那么如果我们的要求字符串是数字与英文呢？就将他全部写在一起，变成：`[a-zA-Z0-9]`。

#### 例题三、行首与行尾字符 ^ $
我们在例题一当中，可以查询到一行字符串里面有 the 的，那如果我想要让 the 只在行首列出呢？这个时候就得要使用制表符了。我们可以这样做：
```
[root@www ~]# grep -n '^the' regular_express.txt
12:the symbol '*' is represented as start.
```
那如果我不想要开头是英文字母，则可以是这样：
```
[root@www ~]# grep -n '^[^a-zA-Z]' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
21:# I am VBird
```
注意到了吧？那个 ^ 符号，在字符集合符号(括号[])之内与之外是不同的！在 [] 内代表【反向选择】，在 [] 之外则代表定位在行首的意义！

如果我想要找出来，行尾结束为小数点 (.) 的那一行，该如何处理：
```
[root@www ~]# grep -n '\.$' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
4:this dress doesn't fit me.
10:motorcycle is cheap than car.
...
```
特别注意到，因为小数点具有其他意义(底下会介绍)，所以必须要使用跳脱字符`(\)`来加以解除其特殊意义！

那么如果我想要找出来，哪一行是【空白行】，也就是说，该行并没有输入任何数据，该如何搜寻？
```
[root@www ~]# grep -n '^$' regular_express.txt
22:
```
因为只有行首跟行尾 (^$)，所以，这样就可以找出空白行了。

假设你已经知道在一个程序脚本(shell script)或者是配置文件当中，空白行与开头为 # 的那一行是批注，因此如果你要将资料列出给别人参考时，可以将这些数据省略掉以节省保贵的纸张，那么你可以怎么作呢？我们以`/etc/syslog.conf` 这个档案来作范例，你可以自行参考一下输出的结果：
```
[root@www ~]# cat -n /etc/syslog.conf
# 在 CentOS 中，结果可以发现有 26 行的输出，很多空白行与 # 开头
[root@www ~]# grep -v '^$' /etc/syslog.conf | grep -v '^#'
# 结果仅有 7 行，其中第一个【 -v '^$' 】代表【不要空白行】，
# 第二个【 -v '^#' 】代表【不要开头是 # 的那行】
```

#### 例题四、任意一个字符 . 与重复字符 *
我们知道通配符 * 可以用来代表任意(0 或多个)字符，但是正规表示法并不是通配符，两者之间是不相同的。至于正规表示法当中的【 . 】则代表【绝对有一个任意字符】的意思。这两个符号在正规表示法的意义如下：

	. (小数点)：代表【一定有一个任意字符】的意思；
	* (星星号)：代表【重复前一个 0 到无穷多次】的意思，为组合形态

我们直接做个练习吧。假设我需要找出 g??d 的字符串，亦即共有四个字符，起头是 g 而结束是 d ，我可以这样做：
```
[root@www ~]# grep -n 'g..d' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
9:Oh! The soup taste good.
16:The world <Happy> is the same with "glad".
```
因为 \* 代表的是【重复 0 个或多个前面的 RE 字符】的意义，因此，【o\*】代表的是：【拥有空字符或一个 o 以上的字符】，特别注意，因为允许空字符(就是有没有字符都可以的意思)，因此，`【 grep -n 'o*' regular_express.txt 】`将会把所有的数据都打印出到屏幕上。因此，当我们需要【至少两个 o 以上的字符串】时，就需要 ooo\* ，亦即是：
```
[root@www ~]# grep -n 'ooo*' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!
```

#### 例题五、限定连续 RE 字符范围 {}
在上个例题当中，我们可以利用 . 与 RE 字符及 \* 来设定 0 个到无限多个重复字符，那如果我想要限制一个范围区间内的重复字符数呢？举例来说，我想要找出两个到五个 o 的连续字符串，该如何作？这时候就得要使用到限定范围的字符 {} 了。但因为 { 与 } 的符号在 shell 是有特殊意义的，因此，我们必须要使用跳脱字符 \ 来让他失去特殊意义才行。至于 {} 的语法是这样的，假设我要找到两个 o 的字符串，可以是：
```
[root@www ~]# grep -n 'o\{2\}' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!
```
这样看似乎与 ooo\* 的字符没有什么差异？因为第 19 行有多个 o 依旧也出现了。那么换个搜寻的字符串，假设我们要找出 g 后面接 2 到 5 个 o ，然后再接一个 g 的字符串，他会是这样：
```
[root@www ~]# grep -n 'go\{2,5\}g' regular_express.txt
18:google is the best tools for search keyword.
```

### 5、基础正规表示法字符汇整 (characters)
经过了上面的几个简单的范例，我们可以将基础的正规表示法特殊字符汇整如下：
<style>
table th:first-of-type {
    width: 100px;
}
</style>

RE字符 | 意义与范例
-------|------------
^word   |意义：待搜寻的字符串(word)在行首！<br/> 范例：搜寻行首为 # 开始的那一行，并列出行号 <br/> `grep -n '^#' regular_express.txt`
word$ | 意义：待搜寻的字符串(word)在行尾！ <br/> 范例：将行尾为 ! 的那一行打印出来，并列出行号 <br/> `grep -n '!$' regular_express.txt`
 \.  | 意义：代表【一定有一个任意字符】的字符！ <br/> 范例：搜寻的字符串可以是 (eve) (eae) (eee) (e e)， 但不能仅有 (ee) ！亦即 e 与 e 中间【一定】仅有一个字符，而空格符也是字符！ <br/> `grep -n 'e.e' regular_express.txt`
 \\  | 意义：跳脱字符，将特殊符号的特殊意义去除！ <br/> 范例：搜寻含有单引号 ' 的那一行！ <br/> `grep -n \' regular_express.txt`
  \* | 意义：重复零个到无穷多个的前一个 RE 字符 <br/> 范例：找出含有 (es) (ess) (esss) 等等的字符串，注意，因为 * 可以是 0 个，所以 es 也是符合带搜寻字符串。另外，因为 * 为重复【前一个 RE 字符】的符号，因此，在 * 之前必须要紧接着一个 RE 字符。例如任意字符则为【.*】！ <br/> `grep -n 'ess*' regular_express.txt`
[list] | 意义：字符集合的 RE 字符，里面列出想要撷取的字符！ <br/> 范例：搜寻含有 (gl) 或 (gd) 的那一行，需要特别留意的是，在 [] 当中【谨代表一个待搜寻的字符】，例如【 a[afl]y 】代表搜寻的字符串可以是 aay, afy, aly 即[afl] 代表 a 或 f 或 l 的意思！ <br/> `grep -n 'g[ld]' regular_express.txt`
[n1-n2] | 意义：字符集合的 RE 字符，里面列出想要撷取的字符范围！ <br/> 范例：搜寻含有任意数字的那一行！需特别留意，在字符集合 [] 中的减号 - 是有特殊意义的，他代表两个字符之间的所有连续字符！但这个连续与否与 ASCII 编码有关，因此，你的编码需要设定正确(在 bash 当中，需要确定 LANG 与 LANGUAGE 的变量是否正确！) 例如所有大写字符则为 [A-Z] <br/> `grep -n '[0-9]' regular_express.txt`
[^list] | 意义：字符集合的 RE 字符，里面列出不要的字符串或范围！ <br/> 范例：搜寻的字符串可以是 (oog) (ood) 但不能是 (oot) ，那个 ^ 在 [] 内时，代表的意义是【反向选择】的意思。 例如，我不要大写字符，则为 [^A-Z]。但是，需要特别注意的是，如果以 `grep -n [^A-Z] regular_express.txt`来搜寻，即发现该档案内的所有行都被列出，为什么？因为这个 [^A-Z] 是【非大写字符】的意思，因为每一行均有非大写字符，例如第一行的 "Open Source" 就有 p,e,n,o....等等的小写字 <br/> `grep -n 'oo[^t]' regular_express.txt`
\\{n,m\\} | 意义：连续 n 到 m 个的【前一个 RE 字符】 <br/> 意义：若为 \\{n\\} 则是连续 n 个的前一个 RE 字符， <br/> 意义：若是 \\{n,\\} 则是连续 n 个以上的前一个 RE 字符！ 范例：在 g 与 g 之间有2 个到 3 个的 o 存在的字符串，亦即 (goog)(gooog) <br/> `grep -n 'go\\{2,3\\}g' regular_express.txt`

再次强调：【正规表示法的特殊字符】与一般在指令列输入指令的【通配符】并不相同，例如，在通配符当中的 \* 代表的是【 0 ~ 无限多个字符】的意思，但是在正规表示法当中， \* 则是【重复 0 到无穷多个的前一个 RE 字符】的意思。使用的意义并不相同，不要搞混了。


## 二、延伸正规表示法
事实上，一般读者只要了解基础型的正规表示法大概就已经相当足够了，不过，某些时刻为了要简化整个指令操作，了解一下使用范围更广的延伸型正规表示法的表示式会更方便。举个简单的例子，我们要去除空白行与行首为 # 的行列，使用的是
`grep -v '^$' regular_express.txt | grep -v '^#'`
需要使用到管线命令来搜寻两次！那么如果使用延伸型的正规表示法，我们可以简化为：
`egrep -v '^$|^#' regular_express.txt`
延伸型正规表示法可以透过群组功能【 | 】来进行一次搜寻！那个在单引号内的管线意义为【或 or】。此外，grep 预设仅支持基础正规表示法，如果要使用延伸型正规表示法，你可以使用 grep -E ，不过更建议直接使用 egrep ！直接区分指令比较好记忆！其实 egrep 与 grep -E 是类似命令别名的关系。

RE字符 | 意义与范例
-------|---------------
 \+    | 意义：重复【一个或一个以上】的前一个 RE 字符 <br> 范例：搜寻 (god) (good) (goood)... 等等的字符串。那个 o+ 代表【一个以上的 o 】所以，底下的执行成果会将第 1, 9, 13 行列出来。 <br> `egrep -n 'go+d' regular_express.txt`
 ?     | 意义：【零个或一个】的前一个 RE 字符 <br> 范例：搜寻 (gd) (god) 这两个字符串。 那个 o? 代表【空的或 1 个 o 】所以，上面的执行成果会将第 13, 14 行列出来。有没有发现到，这两个案例( 'go+d' 与 'go?d' )的结果集合与 'go*d' 相同？想想看，这是为什么！ <br> `egrep -n 'go?d' regular_express.txt`
  &#124;   | 意义：用或( or )的方式找出数个字符串 <br> 范例：搜寻 gd 或 good 这两个字符串，注意，是【或】！ 所以，第 1,9,14 这三行都可以被打印出来。那如果还想要找出 dog 呢？ <br> `egrep -n 'gd&#124;good' regular_express.txt` <br> `egrep -n 'gd&#124;good&#124;dog' regular_express.txt`
 ()    | 意义：找出【群组】字符串 <br> 范例：搜寻 (glad) 或 (good) 这两个字符串，因为 g 与 d 是重复的，所以，我就可以将 la 与 oo 列于 ( ) 当中，并以 &#124; 来分隔开来，就可以。 <br> `egrep -n 'g(la&#124;oo)d' regular_express.txt`
 ()+   | 意义：多个重复群组的判别 <br> 范例：将【AxyzxyzxyzxyzC】用 echo 叫出，然后再使用如下的方法搜寻一下！ <br> echo 'AxyzxyzxyzxyzC' &#124; egrep 'A(xyz)+C' <br> 上面的例子意思是说，我要找开头是 A 结尾是 C ，中间有一个以上的 "xyz" 字符串的意思
