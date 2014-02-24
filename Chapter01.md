为什么是Martini
===============

在上一版 [Go语言博客实践][1] 中, 作者提到不使用框架来完成一个 Blog 系统. 现在选择 [Martini][2] 作为基础框架确实和 Martini 设计的独特性有关. Martini 的核心 [Injector][3] 实现了[依赖注入][4] ( 参见 [控制反转][5] ).

这里有两篇博客可供参考 [Martini的工作方式][6] 和 [Martini中的Handler][7].
简单的说 Injector 通过 reflect 削弱了合作对象间引用依赖.

对于 Martini 的使用可以简单总结为:

 - Martini 对象方法 Map/MapTo/Use/Handlers/Action 非并发安全, 服务器运行前使用.
 - Router 对象也是非并发安全的, 服务器运行前使用.
 - Context 对象是在 http Request 时动态创建的.
 - 所有要使用的对象必须先 Map/MapTo.
 - 对 http.ResponseWriter 任何的 Write 都会终止 http Request.
 - 善用 Context 对象的 Next 方法会产生奇效.


上一版本因为不能找到 "解耦" 的框架而放弃使用框架. Martini 在 Injector 的支持下为"解耦"提供了可能. 这正是笔者希望的.

Package选择与修改
=================

 - Martini社区 [martini-contrib][8]

    这是一组社区提供的 Martini package, 可能会使用一些. 事实上如果您研究过 Martini 和这些 contrib package 后您会发现一个事实: 真的解耦了.

 - 角色控制 [accessflags][9]

    角色控制是应用中的常见需求, accessflags 基于 Martini 实现了一个通过 interger 标记值控制 Martini.Handler 是否允许访问. 可以用于角色控制.

 - 配置文件操作 [tom-toml][10]

    笔者重新写了一个 TOML 解析器 tom-toml, 参见文章[有关tom-toml的一些事儿][11].

 - 数据库操作 [typepress/db][12]

    [upper.io/db][13] 是 [gosexy/db][14] 的重构版本. 代码质量有很大提高. 但是同样的包路径给社区开发造成了同样的问题. 为了以后方便笔者 fork 了一个 github 版本 [typepress/db][15].
    upper.io/db 为常见的 SQL/NoSQL 数据库提供了统一的调用接口, 这是非常难能可贵的.

 - 日志支持 [typepress/log][16]

    typepress/log 学习了 [uniqush/log ][17] 的一些好想法重新构建的.
    typepress/log 支持日志分割, 并实现了一个 file 日志, 一个 email 日志.

 - template 模板

    可能会有几个备选版本 martini-contrib 中有[render][18], 笔者写有 [template][19].

 - [i18n][20] 国际化支持

    这是一个简洁的 i18n 支持接口, 仿照 fmt.Sprint, fmt.Sprintf 的形式.
    在使用中即便暂时没有国际化支持的需求, 使用 i18n 所带来的消耗也是极小的. 完全可以当作 fmt.Sprint, fmt.Sprintf 两个方法的代理.

依赖注入
========

Martini 的核心就是依赖注入, 解耦支持. 那依据依赖注入的思路, 上述的 package 被替换掉应该不是一件复杂的事情. 随时引入依赖注入也应该很容易. 也许吧. 实践中我会关注这个事情.


  [1]: https://github.com/achun/Go-Blog-In-Action/tree/master
  [2]: https://github.com/codegangsta/martini "Martini"
  [3]: https://github.com/codegangsta/inject "inject"
  [4]: http://en.wikipedia.org/wiki/Dependency_injection "Dependency injection"
  [5]: http://zh.wikipedia.org/wiki/控制反转 "控制反转"
  [6]: http://my.oschina.net/achun/blog/192912 "Martini的工作方式"
  [7]: http://my.oschina.net/achun/blog/197546 "Martini中的Handler"
  [8]: https://github.com/martini-contrib "martini-contrib"
  [9]: https://github.com/typepress/accessflags
  [10]: https://github.com/achun/tom-toml
  [11]: http://my.oschina.net/achun/blog/196953 "有关tom-toml的一些事儿"
  [12]: https://github.com/typepress/db
  [13]: https://github.com/upper/db
  [14]: https://github.com/gosexy/db
  [15]: https://github.com/typepress/db
  [16]: https://github.com/typepress/log
  [17]: https://github.com/uniqush/log
  [18]: https://github.com/martini-contrib/render
  [19]: https://github.com/achun/template
  [20]: https://github.com/typepress/i18n