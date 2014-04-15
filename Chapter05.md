Go-Pages
========
熟悉 [GitHub Pages][1] 的读者, 看到 Go-Pages 已经想到 **静态博客** 这个词了. TypePress 从静态博客起步, 一点点迈进带数据库的博客系统. Github 的 Pages 功能已经提出了实用简洁的静态博客方案, [jekyllrb][2] 引擎为其提供强劲动力. Jekyll 给出了很好的文档规范, 可以直接借鉴其目录结构. Liquid 模板也有 Go 实现 [Liquid Template Engine for Go][3]. Go-Pages 尽可能兼容 Jekyll, 不能兼容的部分以后制作转换工具进行处理. 为此需要准备一些 package.

## RootPath
[rootpath][4] 为多域名服务器绑定目录的 package. 效果上有点像 URLRewrite 的一个子集. 仅对 `http.Request.Host` 进行分析, 匹配成功设定相应的静态文件目录, 内容目录, 模板目录. 匹配失败拒绝访问或者不做任何操作.
RootPath 让 Go-Pages 博客支持子域名(站群)或者 CNAME (绑定域名)支持.

## static
[static][5] 在设定好的静态文件目录下, 响应 `URL.Path` 请求的静态文件, 尝试发送对应的 Gzip 预压缩文件 `pathto/URL.Path.gz`. 如果没有找到 static 不产生 404 , 它什么都不做.
不产生 404 有很多好处. 基于 Martini 的 Handler 一旦产生输出就会结束响应过程,  不产生 404 就可以继续进行处理, 比如自定义 404 页面, 比如进行动态 Gzip 压缩, 然后再交给 static 进行输出, 又或者那根本就不是个静态页面, 交给后续的 Handler 处理, 如果最终无法匹配, Martini 会执行 `http.NotFound`.

## Liquid
[Liquid][6] 包提供了基本 Liquid 模板支持. Jekyll 对 liquid 其进行了一些扩展, 如果要完全兼容 Jekyll 是个庞大的工程. 但是, 有必要实现一些如 [Global Variables][7] 之类的. 用到的时候再分析.

特别的, Liquid 中的 `include` tag 需要使用者自己实现 `IncludeHandler`, 参见 `liquid.Configuration`的接口.

## MarkDown
轻量文本标记语言可以让书写者专注文章内容, 而不是为版式费神, 很适合书写博客. 有多种格式可选. Go-Pages 暂时支持最简单的 MarkDown 格式, 在前后端都要有所支持.

前端支持 MarkDown 的编辑器很多, [markdown-editor][8] 是比较简单的一个. [blackfriday][9] 是 Go 语言下的 MarkDown 解析器. 前端的博客文章编辑和提交这里不讨论了.

## JingYes
前端 CSS 框架更是有太多选择, 当前比较受欢迎的当属 BootStrap 和 PureCSS. Go-Pages 使用 [JingYes][10]. 这里不再列举可能用到的其他前端库.

JingYes只支持现代的浏览器, 不过 html 源代码非常简洁, 可以很方便的改写成其它 CSS 框架.

## TOML
配置文件采用 TOML 格式, 这里分析几个 `table`. 

**defalut**

    [defalut]
    # 安全密匙, 请妥善保管, 切勿外泄.
    # 受密匙的具体使用影响, 更改密匙可能会造成不可预计的破坏.
    secret = ""
    # 主站顶级域名
    domain = ""
    # 本地监听地址
    laddr = ":80"
    # 共享静态文件目录
    static = "pages/defalut"
    # 内容目录是独立的, 需要在 [[rootpath]] 中设置
    content =""
    # 共享模板文件目录
    template = "pages/defalut/_layouts"

作为多域名博客系统有些 css,js,image 资源文件是可以共享的, static 目录起到这个作用. 但是, 可以预计, 很可能会把 MarkDown 文章源文件预渲染成 html 文件, 它也是静态文件, 他们所属的 base 目录是不同的. `static` package 不产生 404 的方式很好的解决了这个问题. Go-Pages 可以这样做(m 是 Martini 对象):

```go
// 设定 defalut.static dir
m.Map(http.Dir(core.Conf["defalut.static"].String()))

m.Use(staticDefalutHandler) // defalut static 优先
m.Use(rootPathHandlerForDomain) // 这个一旦执行 root 就改变了
m.Use(staticHandlerForDomain) // 现在访问的静态文件就是站点的了
```

**rootpath**

    [[rootpath]]
	Flag    = 1 # 1 == FStatic, 每个域名都可以独立有静态文件
	Root    = "pages/domain"
	Pattern = "*"
	Domain  = "localhost"
	CategoryName = ["_site"] # Jekyll 的习惯用 _site 目录, 下述同理

    [[rootpath]]
	Flag    = 2 # 2 == FContent, 每个域名都有独立的 content
	Root    = "pages/domain"
	Pattern = "*"
	Domain  = "localhost"
	CategoryName = ["", "_posts"]

    [[rootpath]]
	Flag    = 4 # 4 == FTemplate, 尝试独立的 _layouts
	Root    = "pages/domain"
	Pattern = "?"
	Domain  = "localhost"
	CategoryName = ["", "", "_layouts"]

上述几个 package 给 Go-Pages 提供了最基础的动力. 流程也基本确定, coding...

  [1]: https://pages.github.com/
  [2]: http://jekyllrb.com/docs/structure/
  [3]: https://github.com/karlseguin/liquid
  [4]: https://github.com/typepress/rootpath
  [5]: https://github.com/typepress/static
  [6]: https://github.com/karlseguin/liquid
  [7]: http://jekyllrb.com/docs/variables/
  [8]: https://github.com/jbt/markdown-editor
  [9]: https://github.com/russross/blackfriday
  [10]: https://github.com/achun/JingYes