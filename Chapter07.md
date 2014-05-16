# ABNF

[BNF][1] 是巴科斯范式, 英语：Backus Normal Form 的缩写, 也被称作 巴科斯-诺尔范式, 英语: Backus–Naur Form. Backus 和 Naur 是两位作者的名字. 必须承认这是一项伟大的发明, BNF 开创了描述计算机语言语法的符号集形式. 如果您还不了解 BNF, 需要先 [Google][2] 一下.  随时间推移逐渐衍生出一些扩展版本, 这里直接列举几条 ABNF [RFC5234][3] 定义的文法规则.

```ABNF
; 注释以分号开始
name     = elements         ; name 规则名, elements 是一个或多个规则名, 本例只是示意
command  = "command string" ; 字符串在一对双引号中, 大小写不敏感
rulename = %d97 %d98 %d99   ; 值表示 "abc", 大小写敏感, 中间的空格是 ABNF 规则定界符
foo      = %x61             ; a, 上面的 "%d" 开头表示用十进制, "%x" 表示用十六进制
bar      = %x62             ; b, "%x62" 等同 "%d98"
CRLF     = %d13.10          ; 值点连接, 等同 "%d13 %d10" 或者 "%x0D %x0A"

mumble   = foo bar foo      ; Concatenation 级联, 上面定义了 foo, bar, 等同区分大小写的 "aba"

ruleset  = alt1 / alt2      ; Alternative 替代, 匹配一个就行, 以 "/" 分界
ruleset  =/ alt3            ; 增量替代以 "=/" 指示
ruleset  = alt1 / alt2 / alt3 ; 等同于上面两行
ruleset  =  alt1 /          ; 你也可以分成多行写
            alt2 /
            alt3

DIGIT    = %x30-39          ; Value range 值范围, 用 "-" 连接, 等同下面
DIGIT    = "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9"

charline = %x0D.0A %x20-7E %x0D.0A ; 值点连接和值范围组合的例子

seqGroup = elem (foo / bar) blat ; Grouping 分组, 一对圆括号包裹, 与下面的含义完全不同
seqGroup = elem foo / bar blat   ; 这是两个替代, 上面是三个级联

integer  = 1*DIGIT            ; Repetition 重复, 1 至 无穷 个 DIGIT
some     = *1DIGIT            ; 0 至 1 个 DIGIT
someAs   = 0*1DIGIT           ; 等同上一行
year     = 1*4DIGIT           ; 1 至 4 个 DIGIT
foo      = *bar               ; 0 至 无穷 个 bar
baz      = 3foo               ; 3 次重复 foo, 等同 3*3foo

number   = 1*DIGIT ["." 1*DIGIT]    ; Optional 可选规则, 用中括号包裹, 等同下面两种写法
number   = 1*DIGIT  *1("." 1*DIGIT)
number   = 1*DIGIT 0*1("." 1*DIGIT)
foobar   = <foo bar is foo bar> baz ; 用尖括号括起来, 一个规则名就可以使用空格这些
                                    ; ASCII 可显示符号了, "<" *(%x20-3D / %x3F-7E) ">"
```
上面的描述用的也是 ABNF, 事实上这些文字就源自 RFC5234 规范.
级联规则就是一个顺序匹配的序列, 好比 Seq 顺序规则或者叫 And 规则. 替代好比 Or 规则或者叫 Any 规则.

# 四则运算表达式

现在我们尝试一下四则运算表达式的 ABNF 写法. 我们从人脑运算方式逐步推演出正确的写法.
周知四则运算会包含数字和运算符还有括号.
```ABNF
; 错误写法一
; Expr 表示要解决的问题, 四则运算规则
Expr = Num /               ; Num 表示数字, 仅仅一个数字也可以构成Expr
	   Num   Op   Expr /   ; Op  运算符 
	   "("   Expr  ")" /   ; 括号会改变 Expr 运算优先级
	   Expr  Op   Expr     ; 最复杂的情况

Op   = "+" / "-" / "*" / "/" ; 运算符的定义
Num  = 1*(0-9)               ; 最简单的正整数定义
```
上面的写法模拟人脑做四则运算的习惯, 很明显绝大多数解析器都无法使用这个规则.
因为出现了[左递归][4]. "最复杂的情况" 这一行中 Expr 出现在规则的最左边, 这将导致解析器递归, 造成死循环. 虽然可以把解析器设计的复杂一些, 解决左递归, 但这会使解析器很复杂, 并造成效率低下, 时间复杂度陡增, 所以通常要求写规则时就消除左递归.

继续分析推演. 消除左递归一般通过因式分解(factor)或者引入新的规则条款(terms)解决.
通过 factor 或者 term 解除左递归发生的可能性, 好比多绕几个圈子, 多给解析器几条路, 让解析器绕过死循环的路径.
下面加上了 Repetition 重复规则. 我们先按照人脑思维, 乘法除法优先的顺序来写.

```ABNF
; 错误写法二
Expr   = Term   *Mul / ; Mul 是乘法, *Mul 表示可能有或没有, Term 就是要绕的圈子了.
         Term   *Quo   ; 除法和乘法一样, Term 这个圈子其实表示的还是 Expr.
Term   = Factor *Add / ; 一个圈子明显不行, 再绕个圈子 Factor,  
         Factor *Sub   ; 这两行描述加减法, 逻辑都没错吧, 都是可能有, 也可能没有

Factor = Num /         ; 绕再多圈子总是要回来的, 数字总要有吧
	     "(" Expr ")"  ; 括号的运算总要有吧

Add    = "+" Term      ; 一旦出现运算符, 后面一定会有后续的表达式吧
Sub    = "-" Term
Mul    = "*" Factor
Quo    = "/" Factor
Num     = 1*(0-9)
```

看上去会发生左递归么? 不会, 怎么绕你都不会死循环, 因为 Factor 的第一条规则 Num , 给绕圈圈一个结束的机会. 这个叫终结符. 但是这个写法是错误的.
你可以在脑子里模拟下 `1+2-3`, 到 `-` 号的时候就解析不下去了. `1+2` 被
```ABNF
Term   = Factor *Add
```
匹配了, 但是后面还有`-`号, 被匹配的是加法规则
```
Add    = "+" Term  ; 最后一个又回到 Term
```
但是 Term 无法匹配减号, Term 推演规则中没有以减号开头的. 你说重头来不就行了? 不行, 解析器执行的规则是找到一条路可以一直走下去, 如果走不动了, 就表示这条规则匹配完成了, 或者失败了. 减号来的时候, 如果假设解析器认为 `1+2` 已经走完, 减号来的时候还是要从 Expr 开始, 不能直接从 Sub 开始, 开始只能有一个, 从 Expr 开始推导不出首次就匹配 "-" 号的. 所以 `1+2-3` 没有走完, 解析进行不下去了.

那上面的问题出在哪里呢? 问题在:

**终结符在推导循环中不能首次匹配**

问题的逻辑是: 

**可以穷举开始和结尾, 不能穷举中间过程.**

解决方法是循环或者递归:

**在循环和递归中已经没有明确的开始,头尾相接就没有头尾了,没有头尾也意味能一直绕下去**

综合这三句话, 我们解决问题的方法也就出来了:

    1. 引入Term, Factor 消除左递归
    2. 要给终结符在循环中首次匹配的机会或者说不阻断循环的进行

终结符就是推导循环到了最后, 不包含规则推导中的其他规则名. 再来符号就是新的, 要重头开始.
有终结但无法继续重头开始, 圈子绕不下去了.

继续推演, 我们先确定终结符. 我们用个小技巧, 按优先级合并运算符.

```ABNF
; 正确写法
Expr   = Term   *Sum   ; 继续绕圈子, *Sum 有或者没有, 先写求和是有原因的
Term   = Factor *Mul   ; 再写乘积, *Sum 不匹配, 就尝试乘积
Sum    = SumOp  Term   ; 求和的运算, 有运算符必定要有后续表达式
Mul    = MulOp  Factor ; 乘积的运算,
Factor = Num /         ; 引向终结
         "(" Expr ")"  ; 括号永远都在

Num    = 1*(0-9)      ; 数字, 这可以是独立的终结符
SumOp  = "+" / "-"    ; 加或者减, 可以叫做求和, 小技巧
MulOp  = "*" / "/"    ; 乘或者除, 可以叫做乘积
```

把这两种写法左右排列, 看的更清楚

```ABNF
; 错误写法                ; 正确写法
Expr   = Term   *Mul /    ; Expr   = Term   *Sum   ; 蛇头
	     Term   *Quo      ; Term   = Factor *Mul   ; 蛇头
Term   = Factor *Add /    ; Sum    = SumOp  Term   ; 咬蛇尾
	     Factor *Sub      ; Mul    = MulOp  Factor ; 咬蛇尾

Factor = Num /            ; Factor = Num /
	     "(" Expr ")"     ;          "(" Expr ")" 

Add    = "+" Term         ; SumOp   = "+" /
Sub    = "-" Term         ;           "-"
Mul    = "*" Factor       ; MulOp   = "*" /
Quo    = "/" Factor       ;           "/"
Num     = 1*(0-9)         ; Num     = 1*(0-9)
```

你应该发现了, 主要区别是: 运算符和后续的 Expr 的结合处理方式不同.
左侧的规则是: (数字,运算符,数字) 然后还想找 (数字,运算符,数字).
右侧的规则是: (数字,运算符) 然后继续 (数字,运算符), 最后找到终结.

左侧规划了一条既定的有终结路线, 走不了几步就终结了. 蛇头没咬到蛇尾, 咬到七寸了.
右侧规划了一条蛇头咬蛇尾的循环路线, 循环中所有的规则名都有机会匹配.



# 解析器

面包会有的

  [1]: http://zh.wikipedia.org/wiki/%E6%89%A9%E5%B1%95%E5%B7%B4%E7%A7%91%E6%96%AF%E8%8C%83%E5%BC%8F
  [2]: https://www.google.com.hk/search?q=BNF%20EBNF
  [3]: http://tools.ietf.org/html/rfc5234
  [4]: http://zh.wikipedia.org/wiki/%E5%B7%A6%E9%81%9E%E6%AD%B8