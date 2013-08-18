还你以自由
=========
如前章所述 TypePress 是一个应用, 如果您使用她, 不需要 import 使用, 下载下来当作代码模板直接修改使用她是正确的方法.
不要担心不能和 TypePress 更新同步之类的问题, 我已经想清楚了, TypePress 就是要实现这样的开发模式.

还你以自由.

我将尝试在 TypePress 中通过组织代码逻辑实现这个目标. (或许需要开发其他辅助工具, 这需要您的建议和贡献)

目录调整
=======

```
src              // 代码都移动到src目录下, 您使用的时候需要设定 GOPATH. 用 GoSublime 用户使用  GS_GOPATH 会非常安逸
├───controllers  // 控制器代码目录
├───global       // 全局使用的对象都在这里, global 是一个很实用的代码组织方案
├───meta         // 数据表的所有定义, 包括验证器在内
├───models       // 业务需求对 meta 的操作
└───typepress    // main.go 被移动到这里了, 正如还你自由那样, 如果您有自己的目录组织想法, 自己写一个好了
    ├───conf     // TOML 配置和安装用的sql文件
    ├───log      // 用于记录日志, 当然您可以在 conf 中更改她
    ├───root     // 网站对应的静态文档根目录, 路由匹配不到就会执行静态文件匹配
    │   ├───css
    │   ├───font
    │   ├───img
    │   └───js
    └───views                // 视图(模板)文件所在base目录
        └───typepress        // 选用的视图方案以子目录的形式存在, 这是缺省提供的方案
            ├───layout.html  // layout 支持, 控制器可设定
            └───install      // 安装控制器(路由名称)对应的视图文件目录, 控制器可设定.
                └───get.tmpl // 视图文件, 控制器可设定. 缺省依据 request.Method 计算.
```

Template
========

如前章所述, 作者对 [achun/template][0] 进行了重构. 实践中发现和闭包结合后简直是逆天了.

作者此刻甚至认为 [achun/template][0]  完全没有必要更新新的功能了.

闭包很容易使用, 您需要什么功能参照闭包的方法自己实现就好了.

这里简单的提取几句代码来说明这种方法的安逸
```go
tpl := template.New("") // 是的不需要名字, tpl 其实是个集合, 官方的也是集合. 自动匹配到第一个有效的模板很容易实现.
tpl.Bultin() // 先执行这一句那些内建模板函数才会生效, 如果您需要那些内建模板函数的话.
content := "get.tmpl" // 这里示意定义一个 content 模板文件的名字
// 用闭包的方法加入您自己的FuncMap
tpl.Funcs(map[string]interface{}{
	"request": func() *http.Request { //
		return r // 这个就是是客户端发起 *http.Request, 提供这个只是一个演示, 说明这种方法完全可用
	},
	"content": func() string { // 用函数的方法运行输出 get.tmpl
		err := tpl.ExecuteTemplate(w, content, dat) // w 就是http.ResponseWriter了
		if err != nil {
			return err.Error()
		}
		return ""
	},
})
// 事实上我们只需要这两个文件既可, 其余的文件可以在模板中用import加载
tpl.ParseFiles("layout.html", "install/get.tmpl")
// 万事俱备执行吧
tpl.Execute(w, "your data")
```
layout.html的样子
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <!-- 这样写只是表示 request 函数是可用的 -->
    <title>{{request.URL.Path}}</title>
    <link rel="stylesheet" href="/css/pure.css">
    <link rel="stylesheet" href="/css/default.css">
</head>

<body>
<!-- 哈, content 用函数的方法完成是不是更优雅呢 -->
{{content}} 
</body>
<script src="/js/jquery-2.0.3.min.js"></script>
<script src="/js/md5.js"></script>
<script src="/js/common.js"></script>

</html>
```
get.tmpl
```html
<h1>这里您随意吧<h1>
```

架构轮廓
=======
核心基础
-------
1. 从[目录调整](#目录调整)中您应该可以看出, 所有的模块都尽量的分目录存储
2. global 的方法是非常实用的
3. 对于WEB应用, 最主要的就是把所有的 Handler 都控制起来, TypePress 对所有涉及的流程都留下了扩展可能.(其实就是再包装一层)
4. [achun/template][0] 中闭包的使用方法让视图获得了自由, 控制器只需要设定`layout`和一个`content` view file 即可.
5. [gorilla/mux][1] 的强大让控制器获得了自由

执行流程
-------
1. 第一个 Handler 其实是 [HandlerMux][2], 当时不知道写什么, 现在知道至少可以写 `w.Header().Set("Server", "TypePress")`
2. 路由匹配失败会执行静态文件`StaticFile`方法
3. 路由匹配成功首先执行是否安装成功的检查, 如果没有安装重定向到`/install/`
4. 为控制器初始化`contextSet`对象, 此对象默认`template`是访问不到的. 支持控制器变换或者设定视图参数.
5. 传递 `*http.Request` 给`fireBeforeFilter` 实施控制器前置过滤
6. 执行控制器 handler, 也就是路由的 `HandlerFunc`, 期间可以设定`contextSet`对象
7. 传递 `*http.Request` 给`fireAfterFilter` 实施控制器后置过滤,(也可以叫做渲染前置过滤)
8. 计算视图相关文件位置, 并渲染视图
9. 传递 `*http.Request`和渲染视图中发生的error 给`fireEndRender` 实现控制器完成观察者


[0]: https://github.com/achun/template
[1]: https://github.com/gorilla/mux
[2]: https://github.com/achun/Go-Blog-In-Action/blob/master/Chapter08.md#mux