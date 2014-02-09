为什么是Martini
===============

在上一版 [Go语言博客实践][1] 中, 作者提到不使用框架来完成一个 Blog 系统. 现在选择 [Martini][2] 作为基础框架确实和 Martini 设计的独特性有关. Martini 的核心 [Injector][3] 实现了[依赖注入][4] ( 参见 [控制反转][5] ).

这里有两篇博客可供参考 [Martini的工作方式][6] 和 [Martini中的Handler][7].
简单的说 Injector 通过 reflect 削弱合作对象需要引用定义的依赖关系. 

上一版本因为找不到能"解耦"的框架而放弃使用框架. Martini 在 Injector 的支持下为"解耦"提供了可能. 这正是笔者希望的.

Package选择与修改
=================

 - Martini社区 [martini-contrib][8]
    这是一组社区提供的 Martini package, 可能会使用一些. 事实上如果您研究过 Martini 和这些 contrib package 后您会发现一个事实: 真的解耦了.
 - 角色控制 [access.flags][9]
    角色控制是应用中的常见需求, 笔者基于 Martini 实现了一个通过 interger 标记值控制允许访问的 package. 可以用于角色控制. 希望能被 martini-contrib 收录.
 - 配置文件操作 [tom-toml][10]
    笔者重新写了一个 TOML 解析器 tom-toml, 参见文章[有关tom-toml的一些事儿][11].
 - 数据库操作 [typepress/db][12]
    [upper.io/db][13] 是 [gosexy/db][14] 的重构版本. 代码质量有很大提高. 但是同样的包路径给社区开发造成了同样的问题. 为了以后方便笔者 fork 了一个 github 版本 [typepress/db][15].
 - 日志支持 [achun/log][16]
    achun/log fork 自 [uniqush/log ][17], 做了些改进.
 - template 模板
    可能会有几个备选版本 martini-contrib 中有[render][18], 笔者写有 [template][19].

依赖注入
========

Martini 的核心就是依赖注入, 解耦支持. 那如果基于依赖注入的想法, 上述的 package 被替换掉应该不是一件复杂的事情. 随时引入依赖注入也应该很容易. 也许吧. 实践中我会关注这个事情.

  [1]: https://github.com/achun/Go-Blog-In-Action/tree/master
  [2]: https://github.com/codegangsta/martini "Martini"
  [3]: https://github.com/codegangsta/inject "inject"
  [4]: http://en.wikipedia.org/wiki/Dependency_injection "Dependency injection"
  [5]: http://zh.wikipedia.org/wiki/控制反转 "控制反转"
  [6]: http://my.oschina.net/achun/blog/192912 "Martini的工作方式"
  [7]: http://my.oschina.net/achun/blog/197546 "Martini中的Handler"
  [8]: https://github.com/martini-contrib "martini-contrib"
  [9]: https://github.com/achun/access.flags
  [10]: https://github.com/achun/tom-toml
  [11]: http://my.oschina.net/achun/blog/196953 "有关tom-toml的一些事儿"
  [12]: https://github.com/typepress/db
  [13]: https://github.com/upper/db
  [14]: https://github.com/gosexy/db
  [15]: https://github.com/typepress/db
  [16]: https://github.com/achun/log
  [17]: https://github.com/uniqush/log
  [18]: https://github.com/martini-contrib/render
  [19]: https://github.com/achun/template