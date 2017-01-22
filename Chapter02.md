永远的MVC
========

[MVC][0] 是对软件软件系统三个基础部分的描述, 就好像[冯·诺伊曼结构][1] 或者[哈佛结构][2]对计算机体系结构的定义. MVC 是客观存在的, 是事实.

- 无论你的代码中是否显式的使用了 MVC 的方法, 她都存在.
- 无论你的代码是否显式的遵循 MVC 的方法, 她都存在.
- 无论你的代码是否违背了公认的 MVC 方法, 她都存在.

总之只要你写代码, 无论你怎么写, 她都存在.
对于一个运算表达式:
```go
a := b + c
```
又或者对于一个函数:
```go
func Foo(b,c interface{})(a interface{}){
	// 函数主体, 完成的就是控制了
}
```

- a 就是 view
- "+" 就是 controller
- b,c 就是model

**MVC在心中**

## 常见的方法

开发者通常会这么做:

- 类型声明, 包含 `Controller` 这个词
- 文件名, 包含 `Controller` 这个词
- 目录名, 包含 `Controller` 这个词

在名称上显示出来是好方法. 这增强了代码可读性, 一目了然.
当然如果目录名已经用了`Controller`了, 目录之下的文件或者类型声明是否有必要再加上 `Controller`, 语言不同, 习惯不同, 并没有定式. Go 语言一向提倡 **能省则省**.

依据 MVC 目录看起来是这个样子

	├───conf
	├───controllers // 这里面因归属关系又嵌套了 N 层
	│   └───something
	├───models
	└───views
	    ├───login
	    └───signup

典型的**树状结构**增强了代码可读性, 是常见 MVC 目录组织形式.

TypePress的方法
===============

能保有树状代码目录结构无疑有助于管理维护. 基于 Martini Injector 风格下, 对象间的依赖被降低, 对象依赖关系不必遵循树状结构, 目录结构也不必保持树状. 这在很多时候会更灵活, 同时这也是一种不常见的方法, TypePress 将尝试使用一些. 

## 扁平目录

意思是尽量降低目录曾经深度, 视觉上不显示依赖关系. 事实上这类似于组件独立化.
如果代码是辅助性的, 例如服务器端的 Handler, 那就表现为独立的 Rep. 如果可以分组, 例如浏览器前端组件, 那就在同一个 Rep 下做扁平目录.

## 自动注册路由

**实验性**想法, 目的是给应用生成工具提供基础支持.
对于具体应用, 比如博客, 业务层面的控制器, 多具有层级关系. 其中还涉及角色控制和 http Request Method. 用 martini.Router 写起来像这样

```go
router.Get("/profile", roleAllow("Admin"),youHandler)
```

如果用自动注册路由写起来像这样

```go
core.AutoRouter(youHandler)
```

前提是要把 path,method,role 写到文件路径里. 对应上面的例子, 完整的运行期路径写起来有这样几种

 - github.com/UserName/RepName/Admin/GET/profile.go
 - github.com/UserName/RepName/Admin/GET.profile.go
 - github.com/UserName/RepName/Admin.GET.profile.go

都是能被识别的写法, 依据大小写和"/","."作为分割符号实现自动注册路由是可能的.
由此设计出自动构建/装配工具就有了基础.**TypePress 将尝试这种方式**.

[0]: http://zh.wikipedia.org/zh/MVC
[1]: http://zh.wikipedia.org/wiki/%E5%86%AF%C2%B7%E8%AF%BA%E4%BC%8A%E6%9B%BC%E7%BB%93%E6%9E%84
[2]: http://zh.wikipedia.org/wiki/%E5%93%88%E4%BD%9B%E7%BB%93%E6%9E%84
