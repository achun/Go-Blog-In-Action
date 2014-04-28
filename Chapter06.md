解析器与舞台剧
==============

好吧, 这是一个单章.

语法解析和编译原理是程序员的基础科目, 笔者却一直没有学好. 看着语法树跳来跳去的圈圈, 脑子里像有一群猴子在蹦跶. 一直想有机会补足这门功课, 为 TOML 写个解析器是个不错的选择. 因此 tom-toml 的解析是纯手工的. 作为一个新手笔者无法用正规准确的文字描述解析器的写法和原理, 因此本章用舞台剧来比喻解析器. 看官权当是看故事, 不必严格追究文法和用词.

本文指的是类 [PEG][1] 的方法, 这里有一篇翻译 [解析表达文法][2]. 简单的说 PEG 下的一切都是可确定的, 无二义性, 上下文无关, 无回溯的(线性时间). 这让我想到了舞台剧(事实是, 我先写完了 tom-toml 才发现用的是手工 PEG 的方法). 剧本是写好的, 场景, 台词, 演员, 结果都是固定的. 那么让我给你讲个大导演 TOM 导演一出舞台剧的故事.

## 汤姆的故事

吉它湖大剧院要办一场舞台剧, 这个任务当然是由大导演汤姆来做, 不然还有谁呢!
这天是周五下午三点, 汤姆接到了剧院下达的任务, 演一场舞台剧. 看完任务, 汤姆怨念骤起:

    啥事儿都找我, 舞台剧反反复复都演了多少年了, 弄啥勒!
    不中, 黑喽还得去斗地主咧, 快下班儿了, 时间紧任务急, 俩钟头弄完它.
    杰森, 把咱勒临时演员都叫来, 有活儿了.

助理杰森抬起头

    老板儿, 撒子事儿么? 你快说, 上次那个剧本的大括弧还没有写完呢.

汤姆

    你胡扯啥, 不是叫你叫临时演员都来么!

杰森

    没得问题! 100 分钟完成.

 汤姆

    拉到吧, 1 分钟, 只给你 1 分钟.

杰森扭头大喊

    汤姆要发福利了....

殷特, 付乐得, 司琼在剧院门口已经蹲守三个月了, 不然还能去哪儿呢! 临时演员的职业操守就是等活儿和追讨工资, 蹲守是基本功. 杰森的声音让三人眼睛一亮, 下一秒就出现在汤姆面前, 此刻杰森的脖子还处于180度状态, 迷茫的望着门口.

    我给恁仨说, 现在有个急活儿, 我直接念台词你们谁能演就言一声儿
    
汤姆掰了掰手指头

    1

殷特和付乐得同时出声

    我认识1, 我能演

司琼一脸漠然, 不懂啊. 汤姆傻眼了, 心说我正琢磨第一句说啥勒, 这俩伙当台词了, 算了, 我也懒的想了, 那就 1 吧, 这...

    2

殷特和付乐得那个高兴啊

    2 我也认识, 我能演

汤姆有点不高兴了, 这俩显摆啥咧, 认识个1,2就这德性

    点
    
殷特茫然了, 付乐得心里高兴啊

    点我也会, 我演吧

汤姆受不了了

    12. 你都认识, 你学问不低啊, 不用说 12.3 你都认识了?
    
付乐得一拍胸脯

    没问题, 12.3 我认识, 12.34567 我也能演

汤姆拿了张纸, 画个圈儿

    中了, 这个角色给你了, 这是场次安排分剧本, 好好练练, 走吧
    
付乐得接分剧本扭头走了

    咱接着走台词啊, 下个台词是...
    "

司琼其实是他们三个学历最高的了, 研究生啊, 可惜书读的太多, 脑子就傻了, 只知道照本宣科, 从小就知道剧本台词都是以 " 开始的, 不然那就不是台词啊, 终于让琼斯等到了

    汤姆导演, 这个是台词, 我会

汤姆一头黑线, 感情 12.3 就不是台词! 怒了

    你啥意思, 我前面说了那么多, 那都不叫台词! 你@#%$#^%$&$%#@%$@#$

琼斯一声不吭, 支起耳朵, 紧锁眉头把汤姆的每一个字都记下

    你以为你会了可多, 你不就认识个"

琼斯眉头一展

    嗯, 汤姆导演, 你说的台词真好, 可标准了, 我都记下了, 保证演好

汤姆张大嘴巴足足90分钟才楞过神来

    中, 就这吧, 给你拿好分剧本, 赶恁咧走吧

琼斯双手接过剧本, 鞠躬, 走人. 殷特急了

    导演给我也安排个活儿吧, 我都认识 0123456789 呢

汤姆随口说

    那你就报开场倒计时吧

汤姆分了剧本, 忽略头才转回90度的杰森, 低头看表. 5点, 汤姆嘴角一扬走出了办公室.

## PEG

PEG 的解析过程就像舞台剧, 固定的台词(待解析的文本), 固定的演员(token), 固定场景下有固定的演员和台词, 并且固定的转场. token 判定函数对得到的字符逐个判断, 例如当顺序流入

    1234.567

直到字符 "4" 时, ItsInteger 和 ItsFloat 都认为可能认识这个 token, 都反馈 "可能是", SMaybe 状态. 出现了 ".", ItsInteger 返回"不认识" SNot 状态, ItsFloat 继续返回 SMaybe. 后续的字符都完毕 ItsFloat 都可识别, 都返回 SMaybe. 最后 ItsFloat 拿到 EOF 后确认认识, 返回 "确定是" SYes 状态. 识别 token 就是这么一个过程.

那么, 整个的解析流程就像舞台剧的场景, 每个场景是清楚会出现哪些 token 的. 以 TOML 语法为例, 开始场景命名为 stageEmpty, 可允许出现的 token 包括:

    EOF            空文本也是允许的
    Whitespace     白字符
    NewLine        新行 LF,CR,LFCR,CRLF
    Comment        # 注释
    TableName      [tableName]
    ArrayOfTables  [[arrayOfTableName]]
    Key            键名

注: 上面的次序有效率问题, 甚至是必须的次序才能实现或简化代码. 周知开始场景和结束场景是相同的, EOF 出现在stageEmpty 中是理所当然的. 如果没有 token 被匹配, 那一定是语法错误. 如果匹配, 就进入下一个场景, 每个场景都有固定的 token 列表, 循环这个过程直到重回开始场景识别到 EOF. token 和场景变化可以这样描述

stageEmpty

     EOF            -> stageEnd
     Whitespace     -> stageEmpty
     NewLine        -> stageEmpty
     Comment        -> stageEmpty
     TableName      -> stageEmpty
     ArrayOfTables  -> stageEmpty
     Key            -> stageEqual
 
stageEqual

    Whitespace     -> stageEqual
    Equal          -> stageValue

stageValue

    Whitespace     -> stageEqual
    ArrayLeftBrack -> stageArray
    String         -> stageEmpty
    Boolean        -> stageEmpty
    Integer        -> stageEmpty
    Float          -> stageEmpty
    Datetime       -> stageEmpty

stageArray

    Whitespace     -> stageEqual
    ArrayLeftBrack -> stageArrayWho
    ArrayRightBrack-> stageArrayWho
    String         -> stageStringArray
    Boolean        -> stageBooleanArray
    Integer        -> stageIntegerArray
    Float          -> stageFloatArray
    Datetime       -> stageDatetimeArray

stageStringArray

    Whitespace     -> stageStringArrayComma
    String         -> stageStringArray
    ArrayRightBrack-> stageArrayPop

stageStringArrayComma

    Whitespace     -> stageStringArrayComma
    Comma          -> stageStringArray
    ArrayRightBrack-> stageArrayPop

以此类推, 其中

    为便于阅读, 上述定义省略部分新行和注释, 这不会影响理解.
    Array 是可嵌套的, stageArrayWho 有多种实现方法, 需要专门的篇幅描述. 本文不讨论. 
    stageStringArray 也受嵌套影响, 肯定不能这么简单就得到 stageXxxxArray. 本文不讨论.
    如果某个 token 在解析时做不到验证完整性, 可以放到生成 Toml 时再检查.

**注: 在本新手眼里 Array 的嵌套被当作左递归的一种, 理论上 PEG 要求消除左递归文法, 先手工硬编码解决这问题吧.**

完全手工构造场景变化表是比较痛苦的, 可以把 token 匹配和文法合法性检查分开, 减省 stage 的数量. 比如 stageStringArrayComma 就可以减省, 留给其他代码处理.

你会发现不同语言实现的 PEG, 在表达式文法和用词上甚至不一致. PEG 确实没有规定确切文法用词, PEG 关注的是解析中的逻辑关系.


  [1]: http://en.wikipedia.org/wiki/Parsing_expression_grammar
  [2]: http://article.yeeyan.org/compare/35225