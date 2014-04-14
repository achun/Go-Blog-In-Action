服务器裸奔
==========

在 go 语言下要跑起一个HTTP服务器是很容易的.
```go
package main

import "net/http"

func main() {
	http.ListenAndServe(":8080", http.FileServer(http.Dir("/usr/share/doc")))
}
```
这就行了,一个静态文件服务器就跑起来了. TypePress 下的代码是这样的
```go
package main

import "github.com/typepress/server"

func main() {
    server.Simple()
}
```
这个服务器没有设置任何 Route, 只返回 404, 裸奔的服务器. 这只是表面, 下面来列举下这个 Simple Server 背后都做了哪些工作

## 配置基本参数
有三种方法设置服务器基本参数:

 - 通过命令行参数 --help 可以获得帮助列表
 - os.Getenv 获取 应用通过 os.Setenv 设置参数
 - 从 TOML 文件读取 [TOML][1] 文件支持已经默认加入

## 基本功能
有些基本的功能是一个框架需要提供的

 - 安全关闭机制 shutdown 总用 kill 是不安全的. 得益于 [manners][2]
 - i18n 接口 [i18n][3] 接口非常轻量, 当 fmt.Sprintf 使就行
 - 自定义信号 完全采用 os.Signal 接口, 安全关闭信号就是基于这个
 - 延迟初始化 有些初始化工作需要在 main 执行时调用
 - 子路由 按 http method 划分的子路由, 主路由只能由 main 函数调用
 - 角色控制 字符串角色命名, 自动转化为 [accessflags][4] 支持的 interger
 - 日志支持 引入 [typepress/log][5], 支持 file 分割, email发送
 - 数据库接口支持 引入[typepress/db][6], 即便不需要也不必担心, 这是个轻量接口
 - [core][7] 全局可访问的对象和 [types][8] 类型
 - **基于 Martini Injector 的设计 这是最最重要的**

这样列举起来, 貌似这个 Simple Server 貌似已经不轻量了. 不! 他依然是轻量的, 因为这些接口设计的很轻量, 当你不用他们的时候, 他们不会产生过多的消耗. 这些接口的代码都很短, 引入他们, 怎加不了多少代码空间.
应该可以看出仅仅是这些基础的功能已经形成了一个服务器框架.

这些都已经准备好了. 哦还有模板, 这个东西不打算默认引入, 各种口味难调.

## 模块化
这些很多都是独立的 package, 可以单独使用. 从 [typepress][9] org 可以看出, 模块以独立的 rep 出现.
typepress 特别注意降低依赖, 写成独立 rep 是最基本的方法.


  [1]: https://github.com/mojombo/toml
  [2]: github.com/braintree/manners
  [3]: https://github.com/typepress/i18n
  [4]: https://github.com/typepress/accessflags
  [5]: https://github.com/typepress/log
  [6]: https://github.com/typepress/db
  [7]: https://github.com/typepress/core
  [8]: https://github.com/typepress/types
  [9]: https://github.com/typepress