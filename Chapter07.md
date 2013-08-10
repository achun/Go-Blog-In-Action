global
======
Global 用于提供服务器运行期全局化的对象和方法. 这是一个很常见的技术方案.
在 Go 语言可以通过 `package global` 来做. 全局的当然是要被其他包`import`的. 因此要注意`package global`中`import`的包不能造成循环`import`.
[global][0] 当前提供了这几个对象

* 管理配置 Conf `*toml.TomlTree` 
* 日志管理 Log `MultiLogger` 
* http路由 Mux `*mux.Router`
* 可卸载监听 GracefulListen `*manners.GracefulListener`
* 站点根目录 DocRoot `string`
* 安全的ShutDown [OnShutDown][3]

toml
====
前文说过选用 TOML 格式作为配置文件. 这也是作者第一次使用 TOML. 在当前的版本下有几点体会

* 虽然类型是 `TomlTree`,但是读取配置数据的时候只能从根对象读取, 这个具体实现有关
* 由于`Get`等方法返回的都是`interface{}`, 所以要用到 `type assertion`

[LoadConfig][5] 大量使用`type assertion`的写法可能会让使用者不喜. 作者感觉可以接受.

main
====
用`braintree/manners`作为可`ShutDown`的服务器. [main][4] 只负责启动/重启/停止服务器, 所以`main`函数反而是最简单的. `import`一些包, 使其生效, 简单的维护下服务器的状态就可以了. 你会发现这个项目是分好多`package`完成, 这就是 Go 的风格.

静态文件
=======
对于`裸奔的服务器`, 完成静态文件访问是最基本的. 路由分派器`gorilla/mux`没有提供静态文件支持, 这很正常, 这不是路由负责的, 要由应用自己完成.TypePress采用这样的路由方式

> 如果所有的路由匹配都失败, 那么就会执行`gorilla/mux`的`NotFoundHandler`. TypePress 用这个进行静态文件请求. 实现一个[静态文件][1]服务器其实是很简单, 加上静态gzip支持也要不了几行代码. 

log
===
文件日志是最常见的日志方案. TypePress选用的`log`包提供了很好的机制,但是没有提供文件日志. 没关系, 自己实现[filewriter][2]. 主要做了2件事

* 给写文件加并发互斥锁
* 注册到[OnShutDown][3],以便正常的关闭日志文件

GracefulListen
==============
当`ShutDown`的时候应该关闭监听. 所以就有了全局的 [GracefulListen][0], `ShutDown`的行为应该有控制负责. 当前还没有具体实现.

注意
====
因还在开发中, 所以 GitHub 源码中没有上传 `DocRoot` 目录, 也没有建立 `log` 目录. 如果你想试运行, 可以自建.

[0]: http://gowalker.org/github.com/achun/typepress/global#_variables
[1]: http://gowalker.org/github.com/achun/typepress/controllers#StaticFile
[2]: http://gowalker.org/github.com/achun/typepress/global#NewFileWriter
[3]: http://gowalker.org/github.com/achun/typepress/global#OnShutDown
[4]: https://github.com/achun/typepress/blob/master/main.go#L1
[5]: http://gowalker.org/github.com/achun/typepress/global#LoadConfig