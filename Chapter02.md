永远的MVC
========

[MVC][0] 是对软件软件系统三个基础部分的描述, 就好像[冯·诺伊曼结构][1] 或者[哈佛结构][2]对计算机体系结构的定义. MVC 是客观存在的, 是事实.

- 无论你的代码中是否显示的使用了 MVC 的方法, 她都存在.
- 无论你的代码是否显示的遵循 MVC 的方法, 她都存在.
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

**MVC常见目录**

依据 MVC 那么比较具有可读性的目录看起来是这个样子

	├───conf
	├───controllers // 这里面因归属关系又嵌套了 N 层
	├───models
	└───views
	    ├───login
	    └───signup

这同样增强了代码可读性, 是一种明显的以名称显示表达的 MVC.

TypePress的方法
===============
## 平板化目录
基于 Martini 下, 由于 Injector 的设计已经降低了包之间依赖, 这使得他们之间的关系变平板化, 没有明显的级别关系. 对于这些包 TypePress 使用平板化目录.

## 自动注册路由
而对于具体应用, 比如博客, 业务层面的控制器, 多具有层级关系. 其中还涉及用户角色和 http Request Method.
可以设计下述方法:

 - 层级目录 比如 `/User/profile.go` 或 `/Admin/update.go`
 - 文件名以 "." 间隔 比如 `/User.profile.go` 或 `/Admin.update.go`

如果再加上 http Request Method:

 - `/User/GET/profile.go`, `User.POST.profile.go` 类似于
 - Router.Get(`/profile`,RoleBase('User'), myHandler)

看上去只是个代码风格问题, 但是如果能依据这些名称规则, 设计出自动构建/装配工具, 就有了新的意义. runtime.Caller 函数可以获取完整的包函数调用路径, 这为根据包函数路径进行自动路由注册提供了基础. **TypePress 将尝试这种方式是否有效率**.


[0]: http://zh.wikipedia.org/zh/MVC
[1]: http://zh.wikipedia.org/wiki/%E5%86%AF%C2%B7%E8%AF%BA%E4%BC%8A%E6%9B%BC%E7%BB%93%E6%9E%84
[2]: http://zh.wikipedia.org/wiki/%E5%93%88%E4%BD%9B%E7%BB%93%E6%9E%84