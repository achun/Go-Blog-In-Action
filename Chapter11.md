Install
=======
第一个控制器是针对路由 `/install/` 的.
TypePress 是一个博客系统, 绑定到某个 blog 帐户的内部 id (自增数值类型)的页面作为首页是完全可行的.
作为支持站群的博客系统, 绑定顶级域名是必要的. 这样就可以为每个用户设定独立的子域名. 当然预留一些域名也是必要的, 这可以通过配置文件进行设定.
自动建立数据库可以为安装提供很大的方便, 也需要实现.
这就是 install 必须要做的几件事了.

TypePress使用了基于[gosexy/db][0] 修改版本 [achun/db][1]

貌似 gosexy 在考虑更新些和修改版本类似的特性, 如果以后能使用gosexy的话, TypePress 会优先考虑.

此 package 支持`MongoDB`, `MySQL`, `PostgreSQL` 和`SQLite3`. 写此章的时候 TypePress 只完成了`mysql.sql`.

如果要更换数据库驱动, 这需要手工修改 `main.go`, import 对应的驱动. TypePress 把 `main.go` 也目录化管理, 您新建一个目录做自己的 `main.go` 是最佳的方式.

子路由
=====
在 [InitInstall][2] 中您可以看到 [gorilla/mux][3] 路由强大的一面.
也许您注意到 TypePress 把很多不需要导出的函数和变量都做了首字母大写的导出处理, 对于不能大写的 `init` 函数添加另外的函数完成导出.
这样做完全是为了 `go doc` 的方便, 对于开发者来说通过 `go doc` 了解内部实现方法是最方便的.

您也许注意到 TypePress 采用函数的形式完成控制器, 没有用 struct 形式. 不要忘记 Go 语言之下函数是有类型的, 用函数的方法同样也是OOP的方法.

作者在和一位开发者 [北京]老田 交流时, 老田说:
> [北京]老田 15:03:31

> 很多人用go还是模拟其他语言的写法而已

> [北京]老田 15:03:42 

> 东西做出来了，但还不golang化

他的话让作者意识到 TypePress 开发中实践 golang 化的重要性.

meta
====
您可以在 [meta][4] 中看到所有的数据表字段定义, 以及对应的验证方法. 采用的是纯手写代码, 看上去那么生硬笨拙.
你也许注意到, 所有的字段命名都是唯一的, 不管在哪个表里面, 只要字段名一致, 那一定表示的是同一类数据.
虽然这样的命名看上去有些长, 写代码要多敲几个键, 但这不会造成歧义.

事实上这样写很简单直观, 一目了然, 而且容易修改.
需要注意的是, meta 中的验证仅限于字段的值, 不考虑记录完整性问题, 那由 models 负责.

字段名大小写的问题. 因为 [gosexy/db][0] 是支持多种数据库的, 开发者有可能用大小写敏感的数据库, 所以 TypePress 的代码中将严格区分大小写. 如果您发现有地方违背了, 请提交[issues][10], 非常感谢.

models
======
[models][5] 也是纯手写代码. 设定了所有的记录级别操作完整性检查, 字段完整性调用 meta 的验证器.

models 不考虑操作权限问题, 这由控制器负责.

事实上 medels 包装了 `CURD` 操作, 通过[CurdCond][6]进行数据约束, 那其实就是几个 `[]string`. 这些数据对不同的 `CURD` 操作约束意义是不同的.

可以预见这种方法满足不了所有的需求. 遇到新状况的时候再想办法解决吧.

meta, models, controllers 就是这样各负其责配合完成业务需求.

事件大杂烩
=========
事件驱动或者事件模型的概念已经很普遍了, 事件机制的价值无可非议. TypePress 采用了最生硬笨拙的实现方法. 为不同的事件建立单独的全局变量, 并且采用`slice`, 而不是`map`. 分开独立的变量就是为了避开使用 `map`. 方法生硬但很有效. 这其实就是一组 [On/Fire][11] 作为前缀的函数. 注册事件用 `OnXxxx`, `FireXxxx` 其实已经在对应的地方写过了, 一般情况下不需要调用.

预计这种方法, 在未来也会应用到插件中去. 事实上写日志啥的就使用 `OnXxxx` 来注册. 原先 TypePress 好几处地方都写下了 `LogDebug(...)` 这样代码, 随着事件机制的完成, 那些代码都移除了. 只是在
```go
func FireEvent(code int, r *http.Request, i ...interface{}) bool {
	for _, f := range events {
		if !f(code, r, i...) {
			return false
		}
	}
	if code != 200 && len(events) == 0 { // 这里
		LogDebug(code, r, i...)
	}
	return true
}
```
增加了一个 `LogDebug` 方便遇到错误时进行调试. 配套的 [HandlerMux_ServeHTTP][12] 中的 `defer` 

```go
defer func() {
	if e := recover(); e != nil {
		http.Error(wr, "500 Internal Server Error", 500)
		_, file, line, _ := runtime.Caller(4) // 这里
		FireEvent(500, r, "PanicOnHandlerRouter", file, line)
	} else {
		FireEvent(200, r, "deferHandlerRouter")
	}
}()
```
获取了 `panic` 出错的代码位置, 并传递给 `FireEvent`. 记录日志就可以通过 `OnEvent` 去注册了, 这次更新忘了实现了. 下次补上.

TypePress 的这种做法不是严格的事件模型, 更像一个大杂烩.

Ajax技巧
========
如果你查看 [InitInstall][0] 的代码你会发现最后一行是
```go
w.Write([]byte("/"))
```
这句代码的本意是要让客户端进行 `Redirect` 重定向. 因为前端使用了Ajax的提交(糟糕, 应该先做个是否是Ajax的判断), 直接使用 `Redirect` 是不行的.

常见方法有

- 输出 JavaScript 脚本
- 输出 JSON 告诉客户端做什么

其实没有这么麻烦, 我们分析一下输出的数据情况

1. JSON 通常是被`{}`,`[]`包裹
2. HTML 要求以`<`开头是合理的
3. 重定向 `http` 或者 '/' 开头是最常见的
4. 直接执行 JavaScript 代码 可以用个小技巧 `;` 打头, 这符合 JavaScript 语法标准
5. 文本信息, 用于提示, 上述都不是的话,那可以当作文本信息了

好了, 你发现了,这些信息全部都是有特征, 可以通过几个打头字符进行识别. TypePress 提供的 js 文件中使用了这些技巧.



碎碎念
=====
完成第一个控制器的时候, 时逢 [BootStrap 3.0][9] 发布, 让作者遗憾的是:

- 仍然采用px作为尺度单位. pt 更符合时代发展.
- 网格仍然沿用`float:left`的方法. [PureCSS][8] 用了新的方法, 当然这需要现代浏览器支持.

可惜 [PureCSS][8] 也没有采用 pt做为单位. 为什么作者会有这样的想法, 解释起来很费劲.
作者曾写了 [静夜思][7] 这个框架, 目前还不完善, 作者没有足够的精力把她用于 TypePress. 以后再做吧.



[0]: https://github.com/gosexy/db
[1]: https://github.com/achun/db
[2]: http://gowalker.org/github.com/achun/typepress/src/controllers#InitInstall
[3]: https://github.com/gorilla/mux
[4]: http://gowalker.org/github.com/achun/typepress/src/meta
[5]: http://gowalker.org/github.com/achun/typepress/src/models
[6]: http://gowalker.org/github.com/achun/typepress/src/models#CurdCond
[7]: http://achun.github.io/JingYes/doc/
[8]: http://purecss.io
[9]: http://getbootstrap.com/
[10]: https://github.com/achun/typepress/issues
[11]: http://gowalker.org/github.com/achun/typepress/src/global#_index
[12]: http://gowalker.org/github.com/achun/typepress/src/controllers#HandlerMux_ServeHTTP