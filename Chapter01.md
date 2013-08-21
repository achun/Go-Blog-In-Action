用工具链开发
============
Blog 系统属于非常典型的 WEB 应用. 通常开发者都会首先考虑选择一个 WEB 框架.
这里我们打算用另外一种方式: 工具链进行开发. 用独立的 Go package 通过组合完成开发.

为什么
-----
这不需要什么理由. 软件开发的过程就是通过分治解决问题的. 把大问题分解成数个小的, 相关度很低(解耦)的问题并解决. 这些小问题被解决实现的代码, 多数可以复用到其他开发中. Go 语言中有 package 的概念, 但通常不提项目(虽然项目是客观存在的), 笔者认为就是在倡导代码的重用性. WEB 框架并没有违背重用性. 相反 WEB 框架更具生产力. 这里非要选用工具链方式实现一个 Blog 系统是一种尝试. 并非反对使用框架. 此时笔者也不知道是否会失败.

尝试不同的方法, 程序员就是干这个的.

工具链选择与修改
--------------
工具选择

* 配置文件操作
	- [pelletier/go-toml][0] 修改版本 [achun/go-toml][1]
* http Server
	- [pelletier/manners][4] 支持 ShutDown
* http Request 路由
	- [gorilla/mux][6] 一个强大的 http Request 路由分派器
* SQL 数据库 CURD 操作
	- [gosexy/db][9] 修改版本 [achun/db][10]
* 日志支持
	- [uniqush/log][17] 妙不可言的日志实现, 修改版本 [achun/log][18]
* sessions
	- [gorilla/sessions][19] 内建支持`cookies`,`filesystem`,可扩展
* template 模板
	- 好吧除了官方包暂时没有更合适的选择
* 提交验证
	- 这个问题比较复杂, 后续会讨论具体实现.

### go-toml
[TOML][2] 是一种类似 ini, 比 ini 更好的配置语言. 比较详细的介绍可以参考笔者修改版本说明 [GO-TOML README_CN][3].
笔者修改的版本也 pull 给了原作者, 或许是作者没有看到, 或许是**笔者蹩脚的英文给搞砸了**, 没有得到任何回复.
笔者希望开源项目, 大家共同维护一个版本, 而不是搞独立的分支.(反例马上就会出现)

### manners
服务器运行中可能因各种原因需要 reload, 或者 restart. 这就需要安全的 Close 端口监听, 甚至重启进程.
Go 官方的 `net/http` 包是不会提供这种应用级别支持的. [manners][4] 可以安全的完成 Close 端口监听, 也就是 ShutDown. 当然其他配套工作会随应用复杂度而变得很麻烦. 安全的 ShutDown 是基础保障. 预留这个设计以备后用. 参见 [manners 优雅的关闭 HTTP server][5].

### mux

功能不完全列举, 详细请看 [mux doc][8]

* 支持正则
* 前缀路由 PathPrefix
* 子路由
* URL parameter
* Host 路由
* StrictSlash
* 组合路由, 各种路由组合到一起不可思议的安逸

mux 提供的功能太全面了, 全部都会用到.

### db
首先仍然选择 SQL 数据库做存储, 而不是 NoSQL 数据库. 这也不需要什么理由. 总要选一个的.
提到 CURD 操作, 不得不说 ORM, 这是很常见的选择. 笔者认为 ORM 没有搞错什么, 只是 ORM 干的太多了. 我们不应该把精力放到大而全的 ORM 上并试图最大限度的替代 SQL. 很明显有些问题还是要靠 SQL 解决. 学习 SQL 的成本远比写一个 ORM 或者学习好一个复杂的 ORM 要低的多.
[gosexy/db][9] 的功能也有 ORM 的部分, 但是官方都不以 ORM 冠名.
[官方文档][16]用了数据库抽象集合包装器的提法.

```
The gosexy/db package is a database abstraction and a collection of wrappers for popular third party SQL and No-SQL database drivers.
```

[gosexy/db][9] 本身写的很好, 相同的接口提供了四种数据库支持 [MongoDB][11], [MySQL][12], [PostgreSQL][13] 和 [SQLite3][14]. 甚至包括 NoSQL 数据库 [MongoDB][11].
选择 [gosexy/db][9] 是因为其接口写的非常好, [官方文档][16] 非常详细.
**遗憾的是修改版本无法与其兼容, 参见修改说明 [README_CN][15].**

### log
一般来说实现一个 log 包并不复杂, 对官方提供的 log 包进行少许包装即可完成. 选择 [uniqush/log][17] 是因为其提供了 MultiLogger. 这是其他 log 包所不具备的. 先看几个定义
```go
type multiLogger struct {
    loggers []Logger
}
func NewLogger(writer io.Writer, prefix string, logLevel int) Logger {/*...*/}
```
`Logger` 不言而喻就是一个 log 接口, 这不必费笔墨讲解. MultiLogger 是一个 Logger 切片. 一般的一个应用一个 log 这是很常规的用法. MultiLogger 有什么必要么? 还记得很多 httpd 服务都把访问日志和错误日志分开保存么?
是了, MultiLogger 可以把不同级别的日志分别输出处理. 并且输出对象要求是 io.Writer 接口实现. 比如根据日志级别不同分别保存到不同文件. 

io.Writer 还有更妙的用法.

[uniqush/log][17] 使用的输出是 io.Writer 接口. 不是 *os.File 之类的具体 struct 实例. 这就可以产生很多变化.
假设你的应用要求, 如果超过某个日志级别需要立刻通知到维护人员, 发 email 或者短信. 可以实现一个 io.Writer 接口用于发邮件,一个文件日志 io.Writer 接口, 然后和日志级别一起作为构建函数
```go
func NewLogger(writer io.Writer, prefix string, logLevel int) Logger
```
的参数. 得到一个 logMail, 一个 logFile . 然后生成一个
```go
log:=MultiLogger(logMail, logFile)
```
好了, 用变量 log 记录日志吧, 不同的日志级别文件也写了, 及时的 email 也发送了.
修改版本的 [achun/log][18] 只做了一些小的调整. 同样 pull 给了原作者, 遗憾的是...或许又是**笔者蹩脚的英文给搞砸了**. 

修改版本只是变更了两项

1. 把变量 `logLevelToName` 改为 `LogLevelToName `, 导出以便改变Level名称.比如原来`logLevelToName[LOGLEVEL_WARN] = "[Warning]"`,你可以改为`[W]`
2. 给Logger增加 Flags 字段, 以便设置 log 格式.默认为官方包 `log.LstdFlags`

当然这不是必须的.
### session
[gorilla/session][19] 本身很简单. 这里已经提及 [gorillatoolkit][20] 两个工具了. 事实是不止两个, 这两个工具内部还使用了 [gorillatoolkit][20] 其他的工具. 
session 综合了其他几个工具 `context`, `securecookie` 提供 cookies 和 filesystem 存储支持. 当然也预留了 `Store` 接口

```go
type Store interface {
    Get(r *http.Request, name string) (*Session, error)
    New(r *http.Request, name string) (*Session, error)
    Save(r *http.Request, w http.ResponseWriter, s *Session) error
}
```
这为扩展提供了可能. [gorilla/session][19] 说明中列举的实现.

 * [github.com/srinathgs/couchbasestore][21] - store sessions in Couchbase
 * [github.com/boj/redistore][22] - store sessions in Redis


船小好调头
=========
看完上面罗罗嗦嗦的解说, 你感觉到工具链的好处了吧! 是的, 用工具链的好处就是船小好调头. 如果你用完整的框架, 当你发现框架的局限不足, 或者你暂时没有理解清楚, 没能找到框架下的解决方法. 基本上你是不会去弄个自己的分支的. 因为你的分支不一定会被原作者接受, 而现实是你还要在意原作者的更新, 这样你会很尴尬. 用工具链不必担心这个, 你可以随意分支, 按需求改写, 更或者是中途换掉一个工具. 这相对容易很多.

[0]: https://github.com/pelletier/go-toml
[1]: https://github.com/achun/go-toml
[2]: https://github.com/mojombo/toml
[3]: https://github.com/achun/go-toml/blob/master/README_CN.md
[4]: https://github.com/braintree/manners
[5]: http://my.oschina.net/achun/blog/150211
[6]: https://github.com/gorilla/sessions
[7]: https://github.com/gorilla/mux
[8]: http://www.gorillatoolkit.org/pkg/mux
[9]: https://github.com/gosexy/db
[10]: https://github.com/achun/db
[11]: http://mongodb.org
[12]: http://mysql.com
[13]: http://postgresql.org
[14]: http://sqlite.com
[15]: https://github.com/achun/db/blob/github/README_CN.md
[16]: https://menteslibres.net/gosexy/db
[17]: https://github.com/uniqush/log
[18]: https://github.com/achun/log
[19]: https://github.com/gorilla/sessions
[20]: http://www.gorillatoolkit.org/pkg/
[21]: https://github.com/srinathgs/couchbasestore
[22]: https://github.com/boj/redistore